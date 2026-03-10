
### NOTE
> This is a *sample* connector.  The connector is provided here with the intent to illustrate certain features and functionality around connectors.

## Azure App Configuration [Sample] Connector
Azure App Configuration is an Azure service that helps you manage application settings and feature flags centrally. Using the App Configuration REST API, you can retrieve key-values (configurations) and feature flags stored in an App Configuration store.

## Prerequisites
You will need the following to proceed:
* A Microsoft Power Apps or Power Automate plan with custom connector feature
* An Azure subscription with an App Configuration store
* The Power platform CLI tools

## Authentication types
The connector supports three authentication types, which can be selected when creating a connection:

### Default Microsoft Entra ID application for OAuth
Sign in with a Microsoft Entra ID (Azure AD) user account. This is the delegated authentication flow – the connection acts on behalf of the signed-in user.

**Setup steps:**
1. Register an Azure AD application in the [Azure Portal](https://portal.azure.com) (see [quickstart](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app)) and note the Application (Client) ID.
2. Set the redirect URI to `https://global.consent.azure-apim.net/redirect`.
3. Add a client secret and note it down – you will need it when deploying the connector.
4. Grant the **Azure App Configuration** / **user_impersonation** API permission.
5. Replace `<<Enter your client ID>>` in `apiProperties.json` with your Application (Client) ID.

**Deploying:**
```paconn
paconn create --api-def apiDefinition.swagger.json --api-prop apiProperties.json --secret <client_secret>
```

### Service Principal Authentication
Authenticate using an Azure AD service principal (application identity). No interactive user sign-in is required.

**Setup steps:**
1. Register an Azure AD application (service principal) in the [Azure Portal](https://portal.azure.com) and note the Application (Client) ID and Tenant ID.
2. Add a client secret and note it down.
3. Assign the service principal the **App Configuration Data Reader** role (or a more permissive role) on your App Configuration store.
4. Replace the following placeholders in `apiProperties.json`:
   - `<<Enter your client ID>>` with your Application (Client) ID
   - `<<Enter your tenant ID>>` with your Tenant (Directory) ID

**Deploying:**
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
