# About Two-Factor Authentication (2FA)

## Availability

By default, strong (two-factor) authentication will be available to users of Casa once they have added at least two credentials. By requiring users to register two strong credentials, the number of account lockouts that require admin intervention will be greatly reduced. If one credential is lost, there will be at least one fallback mechanism available. Avoiding user lockout is important because it prevents a serious burden for IT administrators.

You can increase or even reduce this value at your discretion via the [admin console](admin-console.md#2fa-settings).

There is no limit to the number of credentials a user can enroll, and credentials do not need to be of the same type: any combination is valid. 

## Supported types of 2FA

Users will only be able to add credentials with a type matching one of the already enabled authentication methods in the admin console. See the [Admin console page](./admin-console.md/#enabled-methods) to learn more. Out of the box, all the following authentication methods are supported:

- U2F security keys ([browser restrictions](./faq.md#u2f-restrictions) may apply)
- Super Gluu
- HOTP/TOTP
- OTP sent by SMS with the Twilio service

## Resetting a user's preferred method

In the event a user loses access to his account, you can revert the preferred authentication method to "password only" by following the steps shown in the [troubleshooting guide](faq.md)

## Forcing users to enroll a specific credential before 2FA is available

To further reduce the likelihood of lockouts, you can force users to initially enroll, for instance, one OTP credential before any other. OTP credentials are generally more accessible than their counterparts (like U2F) since they normally don't demand special conditions from the device used to access, like having a USB port.

To do so, just add a new configuration property named `2fa_requisite` to the custom interception script corresponding to the authentication method, and assign `true` as its value. It may take more than one minute for Casa to pick up the changes. To add the property, open oxTrust web console and navigate to `Configuration` > `Manage custom scripts`, collapse the method you want to set as requisite for 2FA, and click on `Add new property`.

You can flag more than one method as requisite. In this case users will be encouraged to enroll one credential associated to any of the flagged methods.

If you are using an authentication method you added your own, just ensure the corresponding plugin implements the `mayBe2faActivationRequisite` method.

If a user attempts to delete their only available credential matching the requisite method, a prompt will appear warning that doing so will disable 2FA, that is, resorting to password authentication.
