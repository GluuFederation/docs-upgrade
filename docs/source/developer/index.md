# Developer Guide

This page introduces two relevant topics for developers: [writing plugins](#plugins) and writing applications for [credentials enrollment](#credentials-enrollment-in-your-applications).

## Plugins

### What are plugins?

A plugin is an artifact packaged in a Java ARchive (<i>jar</i> file) that augments the functionalities available in your default Gluu Casa installation. Under certain circumstances, plugins can override existing application functionalities.

Plugins are mostly written in the Java programming language, but Kotlin can also be used.

#### What can you do with plugins?

Plugins are very versatile, the following is by no means an extensive list of things you can do via plugins:

- Add menu items in user's menu, top-right dropdown menu, or admin's dashboard menu
- Add UI pages with arbitrary content (and backend-functionality!), this also applies for the admin dashboard
- Add static files (e.g. Javascript, images, stylesheets, etc.)
- Add RESTEasy services
- Add or override authentication mechanisms to be supported by the application

In addition to the above:

- Any plugin can have easy access to the underlying LDAP of the Gluu Server where casa was installed
- Plugins can onboard their own libraries (jar files) and classes

#### What can't you do?

- Remove, deactivate or alter existing menu items found in your default installation or added by other plugins
- Alter the way in which certain features look or work such as: password reset, default admin dashboard functionalities, logout, etc.
- Plug custom logic to intercept calls or listen events when they occur (e.g. trigger a notification when a user has enrolled a specific type of credential)
- Alter the authentication flow. While this is not feasible via plugins, you can customize the flow by editing the Casa [interception script](https://gluu.org/docs/ce/admin-guide/custom-script) and its associated custom pages bundled with your Gluu Server installation 3.1.6 or higher.
- Use Dependency Injection
- Use Enterprise Beans (EJB)

### Plugin basics

- [Architectural details](./architecture.md)
- [Introduction to plugin development](./intro-plugin.md)
- [Writing your first plugin](./writing-first.md)

### Common tasks

- [Working with menus](./menus.md)
- [Adding UI pages](./ui-pages.md)
- [Manipulating LDAP data](./ldap-data.md)
- [Adding REST services](./rest-services.md)

### Other topics

- [Adding/overriding authentication mechanisms](./authn-methods/index.md)
- [Tips for plugin development](./tips-development.md)
- [Internals of plugin management in Casa](./plugin-management-internals.md)
- [FAQ](./faq.md)

## APIs for credential enrollment

Despite Casa having enrollment capabilities built-in, a use case may arise where credential enrollment needs to happen elswhere in your app ecosystem. A typical scenario is in a user registration application, where users are asked to enroll strong authentication credentials during account creation.

For this, developers have access to a REST API which faciliates the credential enrollment process. Currently, the following types of credentials can be enrolled using the API:

- OTP by SMS
- TOTP or HOTP mobile applications (like Google Authenticator) 
- Super Gluu push authentication

!!! Note
    FIDO credentials can **not** be enrolled via APIs because the security model requires a direct connection to the server. 

The [Swagger](https://swagger.io/docs/specification/2-0/) definition document is located at `https://host/casa/enrollment-api.yaml`. You can leverage [swagger-codegen](https://github.com/swagger-api/swagger-codegen) to bootstrap the process of creating a client application in order to consume the service in a variety of programming languages. You can achieve similar effects by using [Swagger Hub](https://app.swaggerhub.com).

Additionally, the Casa Github project contains a small [client-side application](https://github.com/GluuFederation/casa/tree/master/extras/enrollment-client/) that mimicks the process of enrolling credentials in Casa using the REST API. 

As the Swagger yaml document states, the API is protected by a bearer token. That is, developers have to pass a suitable value in the authorization header for requests. This means an OpenID Connect client **must be** previously registered in the underlying Gluu Server in order to interact with the server's token endpoint.

For more information about crendential enrollment via APIs, visit the developer [FAQs](./faq.md#enrollment-apis).
