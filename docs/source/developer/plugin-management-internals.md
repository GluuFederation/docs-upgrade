# Plugin management internals

This page does not cover aspects needed for plugin development. It surveys how Casa handles some aspects of plugin processing under the hood. This information can be useful for developers of Casa or to understand certain behaviors of plugins.

## Configuration file

Casa stores most of its configurations in the file `/etc/gluu/conf/casa.json`. This is a plain text file that is updated every time changes are applied via the [administration console](../administration/admin-console.md). If for some reason you need to apply configuration changes manually, carefully edit this file and restart Casa (e.g. `service casa restart`).

Specifically, there is a "plugins" section that lists which plugins are currently known, whether they are stopped or started. Upon starting the application, this list is processed in order to load and start plugins accordingly.

## Plugins directory

Every time a plugin is uploaded, its jar file is copied to `/opt/gluu/jetty/casa/plugins`. This action also updates the configuration file accordingly. When a plugin is deleted, its jar file is removed from this directory, as well.

Upon starting the application, all plugins with jar files not present in this directory are removed from the configuration file.

You should not copy files to this directory unless you have a specific reason to do so.

## Resource extraction and registration

When a plugin is started, the following occurs internally:
- All files in the `assets` directory are extracted into the application's exploded directory inside `pl/<plugin-id>` 
- A new custom ZK Label Locator is registered. This allows labels defined in jar file `labels` directory to be recognized by the ZK framework 
- The jar is thoroughly scanned to search for JAX-RS resources annotated with `javax.ws.rs.Path`. These classes are attached to the RestEasy registry and associated with URLs prefixed by `https://<host>/casa/rest/pl/<plugin-id>/`
- A new log4j2 appender is added dynamically, based on the plugin's `Logger-Name` metadata property

When a plugin is stopped, the analog steps are performed to de-register all elements added dynamically. Also, plugin assets are removed.

## Default (system) extensions

Casa uses the PF4J concept of [default/system extensions](https://pf4j.org/doc/defaultsystem-extension.html) to implement the authentication mechanisms supported out of the box. Basically these are extensions that are not tied to a plugin. This approach was employed for developers to have examples to follow when adding authentication mechanisms in their plugins.

Currently, Casa has the following system extensions:
- `org.gluu.credmanager.plugins.authnmethod.OTPSmsExtension` for OTP backed by SMS (using Twilio service)
- `org.gluu.credmanager.plugins.authnmethod.SecurityKeyExtension` for U2F security keys
- `org.gluu.credmanager.plugins.authnmethod.OTPExtension` for OTP tokens (hard and soft)
- `org.gluu.credmanager.plugins.authnmethod.SuperGluuExtension` for Super Gluu devices

