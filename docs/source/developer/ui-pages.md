# Adding UI pages

!!! Note
    It is highly recommended you follow the [Writing your first plugin](writing-first.md) guide before reading this contents.
    
## Overview

Developers can bundle plugins with any number of UI pages. As mentioned in [UI aspects](./architecture.md#ui-aspects) of architecture page, they are expected to be built using any (or a combination) of the dialects supported by the [ZK framework](https://www.zkoss.org) (xul, xhtml, and native).

Pages must be placed under the `resources` directory of your plugin's maven project. Nesting directories is also allowed. As an example, assuming your project looks like:

```
jar
|
+-- assets 
|   +-- mypage.zul
|   +-- subdir
|       +-- master-detail.zul
|-- ...
|-- ...
``` 

page `mypage.zul` will be accessible in the browser at `https://<host>/casa/pl/<plugin-id>/mypage.zul`, and `master-detail` respectively at `https://<host>/casa/pl/<plugin-id>/subdir/mypage.zul`.

## Template usage

Ideally, plugin pages should leverage the existing look and feel of the application. For this purposes, they can leverage the template used by all pages found in Casa by default. This template, called **general template** (found in file `general.zul` of Casa [github project](./writing-first.md#download-project)) ensures all pages using it as skeleton will:

- Include all javascript and stylesheets needed
- Show the navigation menu on the left (whether regular user or admin menus)
- Display a suitable header and footer

In addition, due to class `org.gluu.casa.core.navigation.PageInitiator` used by this template, an error page will be shown in case there is no user currently logged in or the session has expired.

!!! Note
    If you want some plugin page to be accessed anonymously, you will have to create your own skeleton template.  

Particularly, the template allows page coders to define three named content fragments:

- **title**: Normally, a `title` HTML tag where developers can supply its page title

- **maincontent**: Consists of all markup to be rendered in the main content area of your page (under the header and alongside the left menu)

- **extra**: Markup to be added before the `body` HTML tag closes. This is useful to load javascript files or code once most of page markup is ready.

All three fragments are optional, you can skip any or even all of them in your pages.


## Writing your first pages

Writing pages from scratch can be hard at first. We recommend identifying an existing page that somewhat resembles your needs, and check its code. For this you will have to clone [Casa project](./writing-first.md#download-project) or at least, check the ["Hello World"](./writing-first.md#anatomy) plugin code.

Reuse small pieces of markup until you get acquaintance with the tags, namespaces, and expressions used in zul code. Having some [ZK docs](./intro-plugin.md#reference-docs) at hand is useful as well.

The [UI pages](./writing-first.md#ui-pages) section of "Writing your first plugin" offers an illustrative dissection of a real page with useful explanations.

Also, investing some time to get the grasp of involved CSS frameworks pays a lot in the overall development effort. [Here](./architecture.md#css-frameworks) you can find useful pointers.

## Targetting pages to users

Casa has two types of users, administrators and regular users. Every page added by a plugin is intended for a specific audience. By default, all pages leveraging the general template will be rendered with the regular user menu on the left. If you want your page to be shown as if it were an admin-targetted page, create a directory named `admin` just under the `assets` directory and place your zul file there.

Note **it is your responsability** to ensure an admin page can only be accessed by an administrator since crafting a URL by hand following the pattern `/casa/pl/<plugin-id>/admin/...` will just give access to anyone with a session. For this purpose, keep reading.

## Securing pages

While the general template guarantees no anonymous access to a page is allowed, proper checks need to be added in case you want to restrict access to regular users.

To do so, ensure the ZK tag you are using to pass the `maincontent` has an `if` attribute whose expression evaluates `true` when access should be granted. Using this technique, no `maincontent` will be defined and thus no rendering in the central area of page will take place. A typical example of this is:

```
<z:div if="${sessionContext.user.admin}" self="@define(maincontent)">
...
</z:div>
```

The template also ensures that despite your page has a URL like `/casa/pl/<plugin-id>/admin/...`, the admin menu items on the left will not be displayed. 

## Displaying feedback to end-users

One important aspect of good user experience is providing feedback to users after an operation is performed. Hinting when a task completed successfully or failed is key. For this purpose you can use several techniques:

### ZK Message boxes

These are modal dialogs where users are forced to press a button in order to continue interacting with the application. This is useful when you want to communicate important messages to your users. Messages boxes are triggered from Java code using method `show` of class `org.zkoss.zul.MessageBox`.

Message boxes are powerful because they also allow prompting users for their consent, that is using a proceed/cancel or yes/no approach to execute some action.

### Bootstrap alerts

When the message you want to communicate is easy to get (e.g. operation successful), a simple [alert](https://getbootstrap.com/docs/4.0/components/alerts/) should suffice. For this, you can use `showMessageUI` of class `org.gluu.casa.ui.UIUtils`. Account you have to previously create the proper `div` element to hold the alert in your markup and ensure javascript variable `alertRef` points to the DOM element of the `div`. 

You can find examples of how to do so everywhere in Casa code. A good minimalistic example is in `/admin/logging.zul` page of Casa project; this is the page where administrators can set the logging level of the app.