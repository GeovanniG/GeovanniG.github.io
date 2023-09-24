---
title: "Setting Up Google SSO for .NET Core Web Application"
date: 2023-09-24T00:00:00-07:00
draft: false
---

A simple way to improve the user experience of your website is to add third-party SSO options. In this article, we will describe how to add Google SSO to your .NET Core web apps.

## Creating a Google Cloud Account

Before we can add Google SSO, we will need two pieces of information to communicate with Google's OAuth servers: Client Id and Client Secret. This information is unique to our app and will allow Google to authenticate users on our behalf.

To obtain this information, we will need a Google Cloud account. If you do not have a Google Cloud account, you will need to [create one](https://cloud.google.com/?hl=en). Once your account has been made, you will need to [create a project in Google Cloud](https://developers.google.com/workspace/guides/create-project). With a project set up, we are ready to configure a unique Client Id and Client Secret for our application:

1. Go to [Google API & Services](https://console.cloud.google.com/apis).
2. In the OAuth consent screen of the Dashboard:
    * Select User Type - External and CREATE.
    * In the App information dialog, Provide an app name for the app, user support email, and developer contact information.
    * Step through the Scopes step.
    * Add a Test user and continue on to the next step.
    * Review the OAuth consent screen and go back to the app Dashboard.
3. In the Credentials tab of the application Dashboard, select CREATE CREDENTIALS > OAuth client ID.
4. Select Application type > Web application, and choose a name.
5. In the Authorized redirect URIs section, select ADD URI to set the redirect URI. Example redirect URI: https://localhost:{PORT}/signin-google, where the {PORT} placeholder is the app's port.
6. Select the CREATE button.
7. Save the Client ID and Client Secret for use in the app's configuration.
> When deploying the site, either:
> update the app's redirect URI in the Google Console to the app's deployed redirect URI, or
> create a new Google API registration in the Google Console with its production redirect URI for the production app.

For the next section, we will assume our configurations are stored in the following format:

```bash
...
"GoogleSso": {
  "ClientId": "",
  "ClientSecret": ""
}
...
```

In a non-production environment, these configurations should be stored as user secrets. In production, they should be store in Azure App Key Vault or Azure Application Configuration. 

## Registering Google SSO During Startup

With the credentials added to our app's configuration, we are ready to add Google SSO to our project. To begin, we will need to install the [Microsoft.AspNetCore.Authentication.Google](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Google) Nuget package. We can install it with the following command:

```bash
dotnet add package Microsoft.AspNetCore.Authentication.Google
```

With the package added, we can configure our Google SSO options class:

```c#
public class GoogleSsoOptions
{
    public string ClientId { get; set; }
    public string ClientSecret { get; set; }
}
```

Finally, we register Google as an Authentication provider:

```c#
    public static IServiceCollection AddAuthServices(this IServiceCollection services,
        [NotNull] IConfiguration configuration)
    {
        _ = services.Configure<GoogleSsoOptions>(configuration.GetSection("GoogleSso"));

        var googleSsoOptions = configuration.GetSection("GoogleSso").Get<GoogleSsoOptions>()
                                ?? throw new InvalidOperationException(nameof(GoogleSsoOptions));

        _ = services.AddAuthentication().AddGoogle(googleOptions =>
        {
            googleOptions.ClientId = googleSsoOptions.ClientId;
            googleOptions.ClientSecret = googleSsoOptions.ClientSecret;
        });

        return services;
    }
```

With these options set, we will see a Google button appear when we navigate to:
* /Identity/Account/Login or
* /Identity/Account/Register

{{< image
src="/google-sso/google-sso-button.png"
width="90%"
height="90%"
alt="Screenshot of google sso button" >}}

Lastly, when registering or logging in using Google SSO, please do so using a browser where you are currently logged in. Otherwise, you will see the following message and be unable to register/log in.

{{< image
src="/google-sso/unable-to-sign-in.png"
width="90%"
height="90%"
alt="Screenshot of being unable to sign in using google sso" >}}

This occurs because Google blocks SSO logins from browsers that "Are being controlled through software automation rather than a human".

Thanks for reading!