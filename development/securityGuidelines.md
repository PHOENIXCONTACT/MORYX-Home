# Security Guidlines for Application Developers

This article provides a shallow overview of the different security related aspects when creating a MORYX application.
It lists a selection of topics you will frequently come across when setting up a new application.
To not bore you with tedious explanations, there are only links to more in depth references for further reading.

For a general overview you might also refer to the [secure coding guidelines](https://learn.microsoft.com/en-us/dotnet/standard/security/secure-coding-guidelines).

## Make sure you have an Identity and Access Management

.Net provides a solution for managing identities and access to resources in the form of [Asp.Net Core Identity](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity?view=aspnetcore-8.0&tabs=visual-studio).
Make sure that you use this or any comparable solution to protect your application from unauthorized access.
The MORYX team has wrapped the Asp.Net Core Identity functionality in an easy to use module, the MORYX-AccessManagement.
This module completely relies on the secure implementations and architecture of .Net.
In addition to that, it is build to integrate well with any MORYX application and it's user interfaces.

## Enable CORS only if really needed

If you don't know what this means, don't worry it's turned off by default.
For situations were you need to enable it, take a look at [this article](https://docs.microsoft.com/en-us/aspnet/core/security/cors?view=aspnetcore-6.0) from the Microsoft docs.

## Check how static files are made available

`UseDirectoryBrowser` and `UseStaticFiles` can leak secrets.
While by default static files are served from the `wwwroot` you could change this behviour and include other directories to be served.
Please consider [these remarks from Microsoft](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/static-files?view=aspnetcore-8.0#security-considerations-for-static-files), if you decide to do so.

## Configure your Application to be Https only

This is quite a simple configuration which protects all the data coming from clients against interception.
Take a look at the [documentation from Microsoft](https://learn.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?view=aspnetcore-8.0&tabs=visual-studio%2Clinux-ubuntu) on how to enforce HTTPS.
It holds detailed information to ensure the safty of transmitted data from clients.
If you host your MORYX application in docker, [Microsoft also has your back](https://learn.microsoft.com/en-us/aspnet/core/security/docker-https?view=aspnetcore-8.0)

## Consider Database encryption

While Https ensures encryption of your data in transit you should also consider to encrypt your Data-at-Rest.
This can be done on [multiple levels](https://en.wikipedia.org/wiki/Data_at_rest#Encryption) and which solution fits you best depends on your system architecture.
For the commonly used Postgresql database different options can be used when you decide to enrypt the database itself.
You can read up on them in the [postgresql documentation](https://www.postgresql.org/docs/current/encryption-options.html) and select the most fitting solution for your application.

## Make sure the modules you use come from a trustworthy vendor

By design installed modules as well as their user interfaces can communicate freely within the application.
While this feature allows major usability benefits, it puts the selection of trustworthy modules in your responsibility!
Modules provided and licensed by the MORYX-Team directly are accounted for of course.
However, all MORYX components you might get from sources on the web should be checked for their compliance to security standards.

## Beware of security responsibilities in your own modules

If you use your own modules, especially when they have their own UIs, you need to pay attention to vulnerabilities they might have.
Please inform yourself about what these might be depending on the functionality of your module and make it as hard for attackers as you can reasonably do.

## Take a look at the module specific guidlines

- [MORYX Media](https://github.com/PHOENIXCONTACT/MORYX-Framework/blob/future/docs/articles/module-media/SecurityGuidelines.md)
