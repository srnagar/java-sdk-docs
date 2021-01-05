# Authenticate with Azure using Azure Identity

The Azure Identity library provides Azure Active Directory token authentication support across the Azure SDK. It provides a set of TokenCredential implementations which can be used to construct Azure SDK clients which support AAD token authentication.

This library currently supports:
- [Authenticate with Azure in Developer Environment](./identity_env_auth.md)
    * IntelliJ authentication, with the login information saved in Azure Toolkit for IntelliJ
    * Visual Studio Code authentication, with the login information saved in Azure plugin for Visual Studio Code
    * Azure CLI authentication, with the login information saved in Azure CLI
- [Authenticate with Service Principal](./identity_service_principal_auth.md)
    * Client Secret Authentiation
    * Client Certificate Authentication
- [Authenticate Spplications hosted in Azure](./identity_azure_hosted_auth.md)
    * Default Azure Credential Authentication
    * Managed Identity Authentication
- [Authenticate with User Credentials](./identity_user_auth.md)
    * Interactive browser authentication
    * Device code authentication
    * Username password authentication

## Getting started
### Include the package

The Maven dependency for Azure Identity Client library can be found [here](https://search.maven.org/artifact/com.azure/azure-identity).

## Key concepts
### Credentials

A credential is a class which contains or can obtain the data needed for a service client to authenticate requests. Service clients across Azure SDK accept credentials when they are constructed, and service clients use those credentials to authenticate requests to the service. 

The Azure Identity library focuses on OAuth authentication with Azure Active directory, and it offers a variety of credential classes capable of acquiring an AAD token to authenticate service requests. All of the credential classes in this library are implementations of the `TokenCredential` abstract class in [azure-core][azure_core_library], and any of them can be used by to construct service clients capable of authenticating with a `TokenCredential`. 

See [Credential Classes](#credential-classes) for a complete list of available credential classes.

### DefaultAzureCredential
The `DefaultAzureCredential` is appropriate for most scenarios where the application is intended to ultimately be run in the Azure Cloud. This is because the `DefaultAzureCredential` combines credentials commonly used to authenticate when deployed, with credentials used to authenticate in a development environment. Further details and examples of using `DefaultAzureCredential` can be found [here](identity_azure_hosted_auth.md#default-azure-credential).

## Authenticating Azure Client Libraries

Azure Java client libraries support all `TokenCredential` implementations provided by Azure Identity library.

### Examples
You can find examples of authenticating Azure client libraries with different Token Credential implementations below:

* [Authenticate with Azure in Developer Environment](./identity_env_auth.md)
* [Authenticate with Service Principal](./identity_service_principal_auth.md)
* [Authenticate Applications hosted in Azure](./identity_azure_hosted_auth.md)
* [Authenticate with User Credentials](./identity_user_auth.md)


## Authenticating Azure Management Libraries

Azure Java management libraries support all `TokenCredential` implementations provided by Azure Identity library.

### Set up your environment for authentication on management libraries

In addition to the `TokenCredential`, the subscription ID of your [Azure subscription](https://docs.microsoft.com/learn/modules/create-an-azure-account/4-multiple-subscriptions) is required by the management libraries for managing the Azure resources on that subscription.

The subscription IDs can be find on the [Subscriptions page in the Azure portal](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).

Alternatively, use the [Azure CLI][azure_cli] snippet below to get subscription IDs.

```bash
az account list --output table
```

The subscription ID can be set to environment variable `AZURE_SUBSCRIPTION_ID`.
It will be picked up by `AzureProfile` as the default subscription ID, during the creation of `Manager` service API similar to the following code: 

### Authenticate Management Libraries
The `DefaultAzureCredential` is used in the example below to authenticate `AzureResourceManager` in Azure Management library. Other Token Credential implementations offered in Identity library can be used here as well in place of `DefaultAzureCredential`.

```java
AzureResourceManager azureResourceManager = AzureResourceManager
    .authenticate(
        new DefaultAzureCredentialBuilder().build(),
        new AzureProfile(AzureEnvironment.AZURE))
    .withDefaultSubscription();
```

## Environment Variables
`DefaultAzureCredential` and `EnvironmentCredential` can be configured with environment variables. Each type of authentication requires values for specific variables:

#### Service principal with secret
<table border="1" width="100%">
<thead>
<tr>
<th>variable name</th>
<th>value</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>AZURE_CLIENT_ID</code></td>
<td>id of an Azure Active Directory application</td>
</tr>
<tr>
<td><code>AZURE_TENANT_ID</code></td>
<td>id of the application's Azure Active Directory tenant</td>
</tr>
<tr>
<td><code>AZURE_CLIENT_SECRET</code></td>
<td>one of the application's client secrets</td>
</tr>
</tbody>
</table>

#### Service principal with certificate
<table border="1" width="100%">
<thead>
<tr>
<th>variable name</th>
<th>value</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>AZURE_CLIENT_ID</code></td>
<td>id of an Azure Active Directory application</td>
</tr>
<tr>
<td><code>AZURE_TENANT_ID</code></td>
<td>id of the application's Azure Active Directory tenant</td>
</tr>
<tr>
<td><code>AZURE_CLIENT_CERTIFICATE_PATH</code></td>
<td>path to a PEM-encoded certificate file including private key (without password protection)</td>
</tr>
</tbody>
</table>

#### Username and password
<table border="1" width="100%">
<thead>
<tr>
<th>variable name</th>
<th>value</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>AZURE_CLIENT_ID</code></td>
<td>id of an Azure Active Directory application</td>
</tr>
<tr>
<td><code>AZURE_USERNAME</code></td>
<td>a username (usually an email address)</td>
</tr>
<tr>
<td><code>AZURE_PASSWORD</code></td>
<td>that user's password</td>
</tr>
</tbody>
</table>

Configuration is attempted in the above order. For example, if values for a client secret and certificate are both present, the client secret will be used.

## Troubleshooting
Credentials raise exceptions either when they fail to authenticate or cannot execute authentication.
When credentials fail to authenticate, the`ClientAuthenticationException` is raised and it has a `message` attribute which
describes why authentication failed. When this exception is raised by `ChainedTokenCredential`, the chained execution of underlying list of credentials is stopped.

When credentials cannot execute authentication due to one of the underlying resources required by the credential being unavailable on the machine, the`CredentialUnavailableException` is raised and it has a `message` attribute which
describes why the credential is unavailable for authentication execution . When this exception is raised by `ChainedTokenCredential`, the message collects error messages from each credential in the chain.


<!-- LINKS -->
[azure_cli]: https://docs.microsoft.com/cli/azure
[azure_sub]: https://azure.microsoft.com/free/
[source]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity
[aad_doc]: https://docs.microsoft.com/azure/active-directory/
[code_of_conduct]: https://opensource.microsoft.com/codeofconduct/
[keys_client_library]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-keys
[logging]: https://github.com/Azure/azure-sdk-for-java/wiki/Logging-with-Azure-SDK
[secrets_client_library]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-secrets
[eventhubs_client_library]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs
[azure_core_library]: https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/core
[javadoc]: https://azure.github.io/azure-sdk-for-java
[jdk_link]: https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable

