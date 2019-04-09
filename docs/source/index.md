# Gluu Casa 3.1.6 Documentation

## Overview

Gluu Casa ("Casa") is a self-service web portal for end-users to manage security preferences for their account in a [Gluu Server](https://gluu.org/docs/ce). 

For example, as people interact with an organization's digital services, they may need to:

- Add and remove two-factor authentication (2FA) credentials
- Turn 2FA on and off
- View which external apps have access to their personal data
- Manage trusted devices   

Casa provides a platform for people to perform these account security functions and more. 

## Two-factor authentication

The core use case for Casa is self-service 2FA. If people need to call the helpdesk every time they get a new phone or security key, supporting strong authentication becomes prohibitively expensive. 

Out-of-the-box, Casa can be used to enroll and manage the following authenticators:    

- U2F security keys like [Yubikeys](https://www.yubico.com/products/yubikey-hardware/)       
- Gluu's U2F push-notification mobile app, [Super Gluu](https://super.gluu.org)    
- OTP hardware cards like [these](https://www.ftsafe.com/products/OTP/Display_Card) or dongles like [these](https://www.ftsafe.com/Products/OTP/Single_Button_OTP)      
- OTP mobile apps like Google Authenticator, FreeOTP, etc.       
- Mobile phone numbers able to receive OTPs via SMS   
- Passwords (if stored in the corresponding Gluu Server's local LDAP, i.e. not a backend LDAP like AD)      

## 2FA enrollment APIs

To facilitate 2FA enrollment during account registration or elsewhere in an application ecosystem, Casa exposes APIs for enrolling the following types of authenticators:   

- Phone numbers for SMS OTP   
- OTP apps, cards or dongles        
- Super Gluu Android and iOS devices        

Learn more in the [developer guide](./developer/index.md#apis-for-credential-enrollment).  

## Plugins

Casa is a plugin-oriented, Java web application. Existing functionality can be extended and new functionality and APIs can be introduced through plugins. 

Gluu has written multiple plugins to extend Casa to solve use-cases such as:

- [Account linking](./plugins/account-linking.md)  
- [Consent management](./plugins/consent-management.md) 
- [2FA settings](./plugins/2fa-settings.md)  
- [Custom branding](./plugins/custom-branding.md)  
- Developer portal (*coming soon!*)     

To extend Casa to meet your own custom requirements, learn more about writing plugins in the [developer guide](./developer/index.md).

## Get started

Use the following links to get started with Casa:  

### Admin Guide

  - [Installation](./administration/installation.md)
  - [Admin console](./administration/admin-console.md)
  - [Credentials storage](./administration/credentials-stored.md)        
  - [Custom branding](./administration/custom-branding.md)        
  - [FAQs](./administration/faq.md)            

### User Guide

- [Home](./user-guide.md)

### Developer Guide

- [Home](./developer/index.md)

## License
Gluu Casa is made available under the [Gluu License Agreement for Gluu Casa](https://github.com/GluuFederation/casa/blob/master/LICENSE.md). To obtain a license grant, [contact us](https://gluu.org/contact). 

