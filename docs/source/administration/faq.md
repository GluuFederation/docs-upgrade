# Frequently Asked Questions

<!-- This page summarizes situations that may arise and their suggested solutions. -->

## Common administrative tasks

### Where are the logs?

The application logs are located at `/opt/gluu/jetty/casa/logs`. By default, Casa uses the INFO level for messages. You can change the log level at will using the app's admin UI.

<!--
**Note: check the logs**

To work properly, Casa requires stable dependent components (filesystem, ldap, oxd, oxauth, servlet container, etc). Thus, it's important to determine if all expected services are working properly. 

At startup, the app gathers a good amount of information from its environment. If something is missing or went wrong, messages will be shown in the log. Some messages may warn you about parameters not supplied that were simply inferred for you. 

During normal use, the app will show feedback to users if operations were successful or failed. In the latter case, the log is also worth to look at to diagnose the anomalies.
-->


### How do I restart the application?

To restart Casa, just issue

```
$ service casa restart
```

from within the Gluu Server chroot container.

### How do I custom brand Casa?

We have a dedicated page covering the topic of custom branding [here](custom-branding.md).

### What ports are used by the application?

Casa uses port `8091`. One way to see if the app is up and running is by checking whether this port is open by issuing a command like `netstat -nltp`.

### How to reset a user's preferred authentication method?

If a user has been locked out for any reason (e.g. lost devices), you can reset his preferred method by accessing the admin console and choosing the menu item labelled "Reset users preference".  Type the username (or part of) in the text field and then press search. Once you locate the user in the result grid, click the corresponding row and then hit "Change to password". The row will become disabled, and you'll see a success message.

If you've followed the steps as described above, next time he attempts to log in, **password** will be his new preference and he won't be asked to present additional credentials to enter.

### How to adjust the issuer for OTP tokens

When people add OTP mobile apps, the enrollment appears in the device associated with an "issuer", so it is easy to recognize where the OTPs generated can be used. To keep track of which OTPs are valid for which IDPs, the issuer property can be adjusted in the Gluu Server OTP script. For example, you might want to set the `issuer` property to `ACME Dev` on your dev server, and `ACME, Inc.` on your production server. 

See the screenshot below to see where to edit this property in your Gluu Server IDP. 

![adjust_issuer](https://user-images.githubusercontent.com/5271048/46755827-abd23280-cc8b-11e8-851b-8ffb09b835ae.png)

## Errors shown in the UI

### A page with a "Service Temporarily Unavailable" message appears when accessing the application
This is the 503 HTTP error. There is an Apache server in front of the application (as with oxAuth/oxTrust) and this means the reverse proxy couldn't establish a communication internally with the app. This usually happens when Casa hasn't started completely, so it's usually a matter of waiting a few seconds.

For instance, when you restart the `gluu-server` service, it may take some time for Casa and oxd to become available even when the prompt is back in the console terminal. See also this [question](#what-ports-are-used-by-the-application).

### An "incorrect email or password" error is shown when pressing the login button in the SSO form

This reveals a problem in execution of *casa* custom script. Check if `oxauth_script.log` is showing an error related to the authentication method in question.

If you cannot diagnose the issue, please use the [support forum](https://support.gluu.org) to ask for help. 

### An "Unauthorized access" error is shown when accessing the application

This is caused by an unauthorized access attempt (e.g. users requesting URLs without ever logging in or after session has expired).

### "An error occurred during authorization step" is shown when accessing the application

This is shown when it is not possible to initiate a "conversation" with the authorization server. Check that the oxd server is up and running and that the oxd settings are properly configured. Find them at the administration console or in the configuration file.

### "An error occurred: Casa did not start properly" is shown when accessing the application

This occurs whenever the application failed to start successfully and may be caused by a syntax problem in the configuration file or an inconsistent configuration supplied. Check the log to diagnose the problem. Try to find a message like "WEBAPP INITIALIZATION FAILED" and see the traces above it. Often, error messages are self-explanatory.

Once fixed, please restart the application. You will have to see a "WEBAPP INITIALIZED SUCCESSFULLY" message to know that it's working.

## oxd

### In case of lockout

If for any reason an update to oxd settings results in lockout, or if you provided wrong data during installation, do the following:

* Navigate to `/etc/gluu/conf`
* Edit the config file of the application `casa.json`. Provide the oxd details in under the "oxd_config" section. If you are not sure of what to provide for the "client" section, you can remove it entirely
* Save the file and [restart Casa](faq.md#How-to-restart-the-application)
* [Check the logs](#check-the-logs) and there should be a successful message regarding oxd registration
* Log in to the application

### Casa log shows "Setting oxd-server configs failed"

If you configured Casa to use oxd-https-extension and your log also shows "javax.net.ssl.SSLPeerUnverifiedException: peer not authenticated", this is due to self-signed certificate usage (as is the case in a default oxd installation). You can configure oxd to use a production cert or  add the self-signed cert to the trusted Java certificates file. For the latter take the `.keystore` file referenced in the `oxd-https.yml` config file, and export the certificate (this file contains the password too). Then run a command like the following in the Gluu chroot:

`keytool -import -trustcacerts -keystore /opt/jre/jre/lib/security/cacerts -storepass changeit -noprompt -alias mycert -file PATH_TO_CERT_FILE` 

Finally, restart casa.

## Miscellanenous

### Troubleshooting interception scripts

In order to ensure authentication scripts are properly configured in Gluu, it is advisable to:

* Check the contents of the `oxauth_script.log` file (found at `/opt/gluu/jetty/oxauth/logs`). There should be no errors; only successful initialization messages. Every time a new script is enabled, give the server a minute to pick up the changes.

* Scripts included in the default Gluu Server distribution will provide basic working functionality. To see a script in action, enable a script in `Configuration` > `Manage custom scripts`, then set it as the default oxTrust acr in `Manage authentication` > `Default authentication method` > `oxTrust acr`. Open an incognito browser and login to your oxTrust dashboard. By default, your 2FA credential will be enrolled during the first authentication attempt. 

!!! Note  
    Learn more about how the Gluu Server handles authentication in the [user authentication intro](https://www.gluu.org/docs/ce/authn-guide/intro). In addition, to learn more about how custom interception scripts work, review the [custom script tutorial](https://www.gluu.org/docs/ce/admin-guide/custom-script).  
 

### A previously enabled method is not available anymore

If for some reason you disabled the custom script of an authentication method used by Casa, such method will not be available for enrollment or authentication until you enable both the custom script (in oxTrust) and the method (in Casa admin dashboard).

### What kind of TOTP/HOTP devices are supported?

Both soft (mobile apps) or hard tokens (keyfobs, cards, etc.) are supported. Supported algorithms are `HmacSHA1`, `HmacSHA256`, and `HmacSHA512`.

### Authentication fails when using TOTP or HOTP with no apparent reason

For Time-based OTP, ensure the time of your server is correctly synchronized (use NTP for instance). The time lag of the authentication device used (for instance, a mobile phone) with respect to server time should not be representative. 

Big time differences can cause unsuccessful attempts to enroll TOTP credentials in Casa.

For Event-based OTP (HOTP), ensure you are using a suitable value for `look ahead window` (we suggest at least 10). Check contents of file `/etc/certs/otp_configuration.json`. If you apply editions, it is recommended to press the "Update" button of the "Manage Custom Scripts" form in oxTrust and wait a couple of minutes before retrying..

### U2F restrictions

U2F keys (for enrollment or authentication) are supported in the following desktop browsers only:

- Chrome or Opera (versions greater than 40)
- Firefox (version greater than 57), requires prior u2f activation. 
- Firefox versions older than 57 need the [u2f add-on](https://addons.mozilla.org/en-US/firefox/addon/u2f-support-add-on/) installed.

In all cases, the app interface will display appropriate messages about u2f support, and instructions in case action is needed to use the feature. Currently Casa does not support adding U2F devices from mobile browsers.

### The user interface is not showing any means to enroll credentials

Ensure the following are met:

* You have enabled custom scripts as needed. For instance, if you want to offer users the ability to authenticate using Google Authenticator, you have to enable the script "HOTP/TOPT authentication module". Whenever you enable or disable scripts, please wait a couple of minutes for oxAuth to pick the changes.

* In the administration console, you can see which methods are already enabled in the server, and modify those you want to offer.

### The preferred method for authentication is set to password and user cannot change it

To choose a strong method for authentication, the user has to have enrolled at least a certain number of credentials through the app. Only after this is met, he will be able to set his preferred method. 

In the administration console you can specify the minimum number of enrolled credentials needed to enable second factor authentication for users.

### The log shows an error related to *Obtaining "acr_values_supported" from server*

Upon startup, the application needs to query the OpenID metadata URL of oxAuth to build the list of available authentication mechanisms.  This warning is shown when the URL is not yet accessible or the list hasn't loaded fully. However, the application does several (around 15) evenly timed attempts in order to gather the required info. These messages are not a sign of a malfunction.

## My problem is not listed here

Feel free to open a [support](https://support.gluu.org) ticket.
