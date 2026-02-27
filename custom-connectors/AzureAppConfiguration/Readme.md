
### NOTE
> This is a *sample* connector.  The connector is provided here with the intent to illustrate certain features and functionality around connectors.

## Azure App Configuration [Sample] Connector
Azure App Configuration is an Azure service that helps you manage application settings and feature flags centrally. Using the App Configuration REST API, you can retrieve key-values (configurations) and feature flags stored in an App Configuration store.

## Prerequisites
You will need the following to proceed:
* A Microsoft Power Apps or Power Automate plan with custom connector feature
* An Azure subscription with an App Configuration store
* The Power platform CLI tools

## Building the connector
Since App Configuration APIs are secured by Azure Active Directory (AD), we first need to set up a few things in Azure AD so that our connector can securely access the App Configuration store. After that is completed, you can create and test the sample connector.

### Set up an Azure AD application for your custom connector
We first need to register our connector as an application in Azure AD. This will allow the connector to identify itself to Azure AD so that it can ask for permissions to access App Configuration data on behalf of the end user. You can read more about this [here](https://docs.microsoft.com/en-us/azure/active-directory/develop/authentication-scenarios) and follow the steps below:

1. Create an Azure AD application
This Azure AD application will be used to identify the connector to Azure App Configuration. This can be done using [Azure Portal](https://portal.azure.com), by following the steps [here](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app). Once created, note down the value of Application (Client) ID. You will need this later.

2. Configure (Update) your Azure AD application to access the Azure App Configuration API
This step will ensure that your application can successfully retrieve an access token to invoke Azure App Configuration on behalf of your users. To do this, follow the steps [here](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-configure-app-access-web-apis).
    - For redirect URI, use "https://global.consent.azure-apim.net/redirect"
    - For the credentials, use a client secret (and not certificates). Remember to note the secret down, you will need this later and it is shown only once.
    - For API permissions, make sure "Azure App Configuration" and "user_impersonation" are added.

At this point, we now have a valid Azure AD application that can be used to get permissions from end users and access Azure App Configuration.

### Deploying the sample
Run the following commands and follow the prompts:

```paconn
paconn create --api-def apiDefinition.swagger.json --api-prop apiProperties.json --secret <client_secret>
```

### Connector Security setup

When configuring the connector manually in the [Power Automate Portal](https://flow.microsoft.com), use the following values for the OAuth 2.0 security page:

* `Authentication type`: OAuth 2.0
* `Identity Provider`: Azure Active Directory
* `Client id`: the application (client) ID from the app registration
* `Client secret`: the secret from the app registration
* `Login URL`: https://login.windows.net
* `Tenant ID`: common
* `Resource URL`: https://azconfig.io
* `Scope`: https://azconfig.io/user_impersonation
* `Refresh URL`: https://login.windows.net/common/oauth2/token
* `Redirect URL`: https://global.consent.azure-apim.net/redirect

> **Note on Managed Identity**: Power Platform custom connectors do not support Azure Managed Identity directly. Authentication always requires a registered Azure AD application (service principal) with a client ID and client secret as described above.
>
> The connector uses OAuth 2.0 delegated permissions (user impersonation), meaning the end user's identity is used to access the App Configuration store.
>
> To restrict access, assign the appropriate role (e.g., *App Configuration Data Reader* or *App Configuration Data Owner*) to the Azure AD application in your App Configuration store via Azure RBAC.

## Supported Operations
The connector supports the following operations:
* `List key-values`: Gets a list of key-values with optional key and label filters (to list all feature flags, use key filter `.appconfig.featureflag/*`)
* `Get key-value`: Gets a single key-value by key name and optional label
* `Create or update key-value`: Creates a new key-value or updates an existing one
* `Delete key-value`: Deletes a key-value by key name and optional label
* `Get feature flag`: Gets a single feature flag by name and optional label

## Feature Flags
Feature flags are stored in App Configuration as key-values with the key prefix `.appconfig.featureflag/`. The `Get feature flag` operation handles this prefix automatically. To list all feature flags, use the `List key-values` operation with the key filter set to `.appconfig.featureflag/*`. The `value` field in the response contains the JSON representation of the feature flag definition, which includes properties such as `id`, `description`, `enabled`, and `conditions`.

## Additional Resources
* [Azure App Configuration REST API](https://learn.microsoft.com/en-us/azure/azure-app-configuration/rest-api)
* [Azure App Configuration feature management overview](https://learn.microsoft.com/en-us/azure/azure-app-configuration/concept-feature-management)
