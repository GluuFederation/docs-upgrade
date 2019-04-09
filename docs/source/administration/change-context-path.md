# Change Application Context Path

To publish the application at a location other than `/casa`, do the following:

1. Log into the Gluu Server chroot (e.g. `service gluu-server-3.1.6 login`)       
      
1. Rename Casa's war file with the new path you want to use. For example, if you chose `/creds`, you would do the following:    
   
    ```  
      cd /opt/gluu/jetty/casa/webapps  
      mv casa.war creds.war      
    ```  
1. Adjust Apache's .conf file:    

    - Locate the `https_gluu.conf` file. The exact location will vary depending on your distribution. In Ubuntu, for example, you can find it at `/etc/apache2/sites-available`
   
    - Find the section starting with `<Location /casa>` and replace the 2 occurrences of `casa` with the path of your choice. Do not use trailing slashes   

    - Add the following directive: `Redirect /casa /<new-context-path>` before all `<Location>` and `<Proxy>` sections

1. Adjust custom script settings:    

    - Log in to oxTrust and go to `Configuration` > `Manage custom scripts`
   
    - On the "Person Authentication" tab, collapse the script labeled `casa` and adjust the "supergluu_app_id" property accordingly. Skip this step if your Casa installation has already been used for enrolling or authenticating via Super Gluu.
   
    - At the bottom, press the "Update" button

1. Wait for around 1 minute (so the server picks the script changes), then restart Casa and Apache services. Use these commands as a guide:    

    ```  
      service casa restart    
      service apache2 restart      
    ```  

1. The application should be accessible now at the new URL.
