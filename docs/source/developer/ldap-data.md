# Manipulating LDAP data

Gluu LDAP directory is key in Gluu Casa [architecture](./architecture.md#backend). Plugins are very likely to require LDAP interaction so it's important to learn how this works in Casa. Formerly in the [introductory page](./intro-plugin.md#ldap-sdk-for-java) we highlighted the importance of UnboundID LDAP SDK Persistence Framework as a vehicle to map LDAP entries into Java objects. We also mentioned about `casa-shared` module bringing an abstraction to easily perform CRUD. In this page we will get into the details and provide some useful examples.

## Requisites

Before proceding ensure you have:

- [A basic grasp of LDAP](./intro-plugin.md#ldap-notions)
- Downloaded the [UnboundID LDAP SDK](https://github.com/pingidentity/ldapsdk/releases) (version 4.0.0 or newer is OK). 
- Skimmed through the persistence framework [doc page](https://docs.ldap.com/ldap-sdk/docs/persist/index.html) (also found in the SDK zip file). It will help you to start very quickly. Bookmarks in your browser pointing to that page and the API docs would be helpful.
- The means to connect to Gluu lightweight directory, as mentioned [here](./intro-plugin.md#find-a-graphical-ldap-client). Normally, you'll use the `o=gluu` base branch DN.

## Know your LDAP

It is important to get some acquaintance with the structure of the directory. One you connect you will see two branches, `ou=appliances` and `o=...`. The former contains mostly configuration of the server itself, the latter holds more interesting stuff for developers such as:

- Users and groups data
- OpenID clients and scopes
- Custom scripts

Traverse the tree. Use your LDAP client to inspect schema, that way you can familiarize with the most relevant object classes (e.g. `gluuPerson`) and attributes types they can hold (e.g. `nickname`).

### Adding attributes to directory schema

While Gluu schema exhibits lots of attribute types, there are cases where you need to add your own attributes to cope with particular needs, for instance, storing user's age.

This [page](https://gluu.org/docs/ce/admin-guide/attribute#custom-attributes) is a reference doc demonstrating how an attribute can be added to directory schema. In this case an attribute is added to `gluuCustomPerson` object class, but you can use any other or even create your own. Basically you need to:

1. Choose a name and a description for the attribute.
1. Assign a syntax (e.g. string, boolean, etc.). For more information check RFC 4517 or search "LDAP syntax definitions" in the web.
1. Assign the attribute to one or more object classes.
1. Save the editions in the (plain text) schema file of your LDAP server.
1. Restart the LDAP service. Normally this service will not start if the supplied changes don't have the correct format expected by the LDAP implementation bundled in your Gluu Server.

For plugin development there is no need to register the attribute in oxTrust unless you know you'll need to inspect the attribute in oxTrust itself.

### Branches DNs

Instances of `org.gluu.casa.service.ILdapService` interface provide some `get*` methods to easily obtain distinguished names for the most relevant branches in the directory. Accessing DNs allow plugin developers to specify proper locations for searches or updates.

### POJOs for reuse

`casa-shared` provides a few [persistence-framework compatible POJOs](./intro-plugin.md#ldap-ped-pojos) you can reuse or extend when developing. Plugin requirements are varied so we decided not to provide ready-to-use classes for the majority of existing LDAP object classes. This would result in classes with several dozens of fields (plus associated getters and setters) that are not as meaningful as having small tailored classes focused on modeling the problem at hand. Additionally, writing your own classes will allow you to understand how modeling works and do troubleshooting if problems arise.

## About the persistence framework

If you have ever used the Java Persistence API (JPA), the term "object/relational mapping" may result familiar to you. It consists of a series of rules and annotations to facilitate managing relational data in Java applications. The LDAP persistence framework is aimed at solving a similar problem in LDAP domain, however, this is not a standard itself but a facility available in PingIdentity's LDAP SDK for Java. 

The framework allows to express (via annotations) the logical rules to store a Java object into LDAP, and the conversion (retrieval) of an entry into a Java object.

!!! Note
    We highly encourage to visit the persistence framework [docs](https://docs.ldap.com/ldap-sdk/docs/persist/index.html) now. The framework is very small so it's a short reading. Ensure you read the examples shown.  
    
Now that you have the grasp of the framework, let's proceed with an example. Here, we will generate a POJO with the tool for this purpose, minimize it to fit a particular need, and put it to work on real data.

## A simplified client management tool

Suppose your boss asked you to write a plugin so that your Gluu administrator can edit some details of the existing OpenID Connect Clients found in the Gluu Server installation. Of particular interest is the possibility to assign ownership to clients. This ownership would consist of assigning one or more existing users to a client.

Admins are supposed to have access to a page of your plugin where clients are listed and when a item of the list is selected, client's details are shown as well as the possibility to assign people to it.

In the following, we will focus on what needs to be done just from data perspective. You'll have to figure out how to build the UI and presentation logic code.

### Key facts from schema

Task description involves at least the following is required to fulfil:

1. Query the existing clients. Assume we only need to show their names in the list.
1. Obtain the information of a particular client. In practice OpenID clients have a bunch of attributes, assume only identifier, name, and owners will be shown.
1. Modify the owners of a client. To keep this simple, only usernames will be displayed.

If you have some Gluu LDAP acquaintance, you already know clients are stored under `ou=clients,o=...,o=gluu` branch. Every entry listed there 	correspond to a single client which you can also visualize via oxTrust web console (navigate to `OpenID` > `clients`).

After some LDAP inspection the following facts are worth noting:

- There is an objectClass named `oxAuthClient` to which every client (entry) belongs to.
- Associated to `oxAuthClient` there are around 60 LDAP attribute types in the schema.
- There is an attribute type `associatedPerson` that can serve the purpose of storing the ownerships needed. This attribute has syntax `1.3.6.1.4.1.1466.115.121.1.12` also known as "distinguished name". In other words, `associatedPerson` can be used to reference DNs users (no their user names directly).
- Users are stored under `ou=people,o=...,o=gluu` branch, and subordinate entries belong to `gluuPerson` objectClass. There is an attribute `uid` where the user name is stored.

### Generating a persistence-aware POJO

Most of the complexity in this tasks revolves around retrieving and updating clients, so let's create a Java class that allow us to manipulate client data!. Firstable ensure you have set an environment variable `JAVA_HOME` pointing to your Java SDK root. Then `cd` to the directory where you extracted the LDAP SDK downloaded [earlier](#requisites) and issue a command of the following form:

(for Windows)

```
tools\generate-source-from-schema.bat --hostname <hostname> --port 1636 --bindDN "cn=directory manager" --bindPassword <password> \ 
  --structuralClass oxAuthClient --rdnAttribute inum --packageName <java-package> --className <java-class-name> --terse
```

(for UNIX-like systems)

```
tools/generate-source-from-schema --hostname <hostname> --port 1636 --bindDN "cn=directory manager" --bindPassword <password> \ 
  --structuralClass oxAuthClient --rdnAttribute inum --packageName <java-package> --className <java-class-name> --terse
```

Replace content between angle brackets properly:

- Use the same password you employ to connect in your graphical LDAP client
- For `java-package` use the package the class will belong to, for instance `com.mycompany.casa.plugins.clientmanager`
- For `java-class-name` choose a suitable name, for example `Client`
- If using OpenLDAP instead of OpenDJ, supply "cn=directory manager,o=gluu" for `bindDN`

A few comments before you open the generated file:

- Command line parameter `structuralClass` is the LDAP objectClass employed for representing OpenID clients.
- Parameter `rdnAttribute` is used to specify the RDN attribute of this objectClass. In this case, it is `inum`: note every client RDN has the form `inum=string` where `string` is the value of client's `inum` attribute. Check the `generate-source-from-schema` documentation to learn more about `rdnAttribute`.

There should be a new Java file created in the directory where you issued the command (no directories are created to reflect the package hierarchy). It's a big templatic file despite `terse` param was supplied.

#### Create a thin POJO

In your IDE or favorite editor you are using to code your plugin, create a java package with the same structure as the one supplied in the command line. Then create an empty Java class with the same name of the generated file. Proceed as follows:

- Preceding the class declaration paste the `LDAPObject` annotation found in generated code. Remove elements `postDecodeMethod` and `postEncodeMethod`.

- Paste `ldapEntry` field and its associated annotation in your Java class.

- Paste `inum` and its associated annotation in your Java class. Locate the getter associated paste that too. Use the getter named like `getFirstInum`.

- Repeat the above for `displayName` field.

- Repeat the above for `associatedPerson` field. Use the getter that returns an array.

- Locate the setter for `inum` and add it to the file.

That's all we will need in order to query clients. Save your file.

### Do some retrieval

Create a Java class intended for manipulating clients (for instance `ClientService`). Add to it:

- A private field `ldapService` of class `org.gluu.casa.service.ILdapService`
- Add a public parameterless constructor an initialize `ldapService`. You can do `ldapService = org.gluu.casa.misc.Utils.managedBean(ILdapService.class);`
- Create a method `getClients` that returns a `List<Client>` (use the name of the class you created in the previous section)
- Perform a retrieval of all clients and return it, it is this simple: `ldapService.find(Client.class, ldapService.getClientsDn(), null);`

Here `ldapService` is the object you use to interface with Gluu LDAP (there is no need to worry about LDAP connection details). In the command above, `find` will query all entries under `ou=clients,o=...,o=gluu` that are structurally conformant to `Client` class, that is, entries that belong to `oxAuthClient` objectClass. The actual query will only account for the attributes that are mapped as fields in `Client` class not the whole attributes found at each entry.

**Test it!**: Override `start` method of your plugin class and instantiate `ClientService` in it, then call `getClients`. You can use a logging statement like the following to print the names of all Gluu Server clients registered so far:

```
logger.info("{}", clientService.getClients().stream().map(Client::getDisplayName)
    .collect(Collectors.toList()).toString())
```

Compile, pack your plugin, and deploy it. You should be able to see the list printed in Casa log.

### Retrieve info of a particular client

So far point 1 of basic goals listed [here](#key-facts-from-schema) is satisfied. Now let's try to gather information of a single client, having `inum` as key to retrieve it:

- Add a method `getClient` to class `ClientService` receiving one `String` parameter, say `id`
- Copy the following:

```
Client client = new Client();
client.setInum(id);
return ldapService.find(client, Client.class, ldapService.getClientsDn())
        .stream().findFirst().orElse(null);
```

Pay special attention to line 3: it executes a search for `Client` instances that "look like" `client` variable. Search is performed in the subtree referenced by `ldapService.getClientsDn()` (ie. `ou=clients,o=...,o=gluu`). `client` has only set its `inum`, so the underlying LDAP filter generated for the search will account for:

- Entries with the right structural class (`oxAuthClient` of LDAP schema)
- Entries whose `inum` attribute matches the value of `id` variable

Since `find` method returns a `List`, in line 4 we try to retrieve just the first item: Due to this search being performed using the client's unique identifier, the list could have zero or one elements.

**Test it!**: Add to your plugin's `start` method a call to `getClient` passing, for instance a harcoded `inum` value. You can print the client contents this way:

```
logger.info("Client {} has owners {}", 
    client.getDisplayName(), client.associatedPerson());
```

`generate-source-from-schema` tool generates code that returns `null` in the case of empty arrays, so `null` is what you will see in the log if the chosen client has no "owners" assigned yet. you may like adding the following method to `Client` class:

```
public List<DN> getAssociatedPersonAsList() {
    return Utils.isEmpty(associatedPerson) ? 
              Collections.emptyList() : 
              Arrays.asList(associatedPerson);
}
```

Besides returning a more friendly data structure (`List`) if the underlying LDAP attribute is unassigned, you'll get an empty list - not a ` null` value -.

We are already done with point 2 of the [list](#key-facts-from-schema). Let's move onto the more interesting 3 (modifying).

### Setting the owners of a client

Selecting owners for a client is out of the scope of this tutorial. This task only involves retrievals so you can easily extrapolate your current knowledge to the field of users. This time use as base the branch `ldapService.getPeopleDn()` (ou=people,o=...,o=gluu) and `gluuPerson` as structural class.

Actually, for our toy example you can write a class that resembles `org.gluu.casa.core.ldap.BaseLdapPerson` (found in `casa-shared` maven dependency). Ensure there is a setter for `uid`, and also a method `getDn` that returns `ldapEntry.getParsedDN()`. 

Assuming `client` variable represents the instance you want to assing owners to, you can do the following:

```
//People to co-own this client
Stream<User> owners = Stream.of(larry, moe, curly... 
DN[] peopleDNs = owners.map(User::getDn).collect(Collectors.toList())
                       .toArray(new DN[0]);
client.setAssociatedPerson(peopleDNs);

ldapService.modify(client, Client.class);
```

`modify` takes charge of the underlying complexity of LDAP entry updates. Just ensure `client` was previously retrieved with a call to `find`. The return value here was ignored (a boolean value), in a real world example, you should add logic to process whether the update was successful or not.

This basically covers the need. There are more useful methods found in `org.gluu.casa.service.ILdapService` to manipulate LDAP data. We encourage you to check this interface java docs.
