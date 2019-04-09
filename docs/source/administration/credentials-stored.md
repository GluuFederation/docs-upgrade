# Storage of User Credentials

The following provides a summary of where user credentials can be found in LDAP. 

## U2F devices
Information about U2F security keys and Super Gluu devices are stored under a `fido` branch for every user entry. The entries belong to the `oxDeviceRegistration` object class and contain information about each enrolled device. One way to differentiate between a U2F Security Key and a Super Gluu device is to inspect the `oxDeviceData` attribute. Super Gluu devices will include information here, while U2F Security Keys will not.

!!! Note  
    We strongly recommend **not** modifying these attributes directly unless you have a good reason to do so.  
    
## TOTP / HOTP devices
TOTP/HOTP device information is stored in the `oxExternalUid` attribute as well as in `oxOTPDevices`.

## Phone Numbers
Verified mobile phone numbers are stored in the `mobile` attribute of the user entry. Associated information (date added, nickname of device, etc.) is stored in JSON format in the `oxMobileDevices` attribute.

## Preferred 2FA Method
The user's preferred method of authentication is stored in the `oxPreferredMethod` attribute as a plain ACR value string. If no value is present, it means the user has not turned on 2FA. 

## 2FA enforcement policy

When administrators allow users to set their own strong authentication policy, that is, users being able to decide if 2FA authentication always takes place or only when device/location used is not recognized, the LDAP attributes involved are `oxStrongAuthPolicy` and `oxTrustedDevicesInfo`. The former contains the user preference, and the latter the information of his trusted devices and locations. For privacy reasons, data stored in `oxTrustedDevicesInfo` is encoded so the only applicable operation upon this data is flushing the list; this can be achieved by removing the attribute entirely.

