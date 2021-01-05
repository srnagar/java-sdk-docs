# Authenticate with Azure using Azure Identity

The Azure Identity library provides Azure Active Directory token authentication support across the Azure SDK. It provides a set of TokenCredential implementations which can be used to construct Azure SDK clients which support AAD token authentication.

This library currently supports:
- [Service principal authentication](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals)
- [Managed identity authentication](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview)
- [Device code authentication](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-device-code)
- Interactive browser authentication, based on [OAuth2 authentication code](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow)
- [Username + password authentication](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth-ropc)
- IntelliJ authentication, with the login information saved in Azure Toolkit for IntelliJ
- Visual Studio Code authentication, with the login information saved in Azure plugin for Visual Studio Code
- Azure CLI authentication, with the login information saved in Azure CLI
- Shared Token Cache credential, which shares login information with Visual Studio, Azure CLI, and more

[Source code][source] | [API reference documentation][javadoc] | [Azure Active Directory documentation][aad_doc]

## Getting started
### Include the package

Maven dependency for Azure Secret Client library. Add it to your project's pom file.

```xml
<dependency>
<groupId>com.azure</groupId>
<artifactId>azure-identity</artifactId>
<version>1.2.1</version>
</dependency>
```

### Prerequisites
* A [Java Development Kit (JDK)][jdk_link], version 8 or later.
* An [Azure subscription][azure_sub].
* The Azure CLI can also be useful for authenticating in a development environment, creating accounts, and managing account roles.

### Authenticate the client

When debugging and executing code locally it is typical for a developer to use their own account for authenticating calls to Azure services. There are several developer tools which can be used to perform this authentication in your development environment. You can find instructions here to [Authenticate with Azure in Developer Environment](./identity_env_auth.md)

## Key concepts
### Credentials

A credential is a class which contains or can obtain the data needed for a service client to authenticate requests. Service clients across Azure SDK accept credentials when they are constructed, and service clients use those credentials to authenticate requests to the service. 

The Azure Identity library focuses on OAuth authentication with Azure Active directory, and it offers a variety of credential classes capable of acquiring an AAD token to authenticate service requests. All of the credential classes in this library are implementations of the `TokenCredential` abstract class in [azure-core][azure_core_library], and any of them can be used by to construct service clients capable of authenticating with a `TokenCredential`. 

See [Credential Classes](#credential-classes) for a complete list of available credential classes.

### DefaultAzureCredential
The `DefaultAzureCredential` is appropriate for most scenarios where the application is intended to ultimately be run in the Azure Cloud. This is because the `DefaultAzureCredential` combines credentials commonly used to authenticate when deployed, with credentials used to authenticate in a development environment. The `DefaultAzureCredential` will attempt to authenticate via the following mechanisms in order.

![DefaultAzureCredential authentication flow](https://github.com/Azure/azure-sdk-for-java/raw/master/sdk/identity/azure-identity/images/defaultazurecredential.png)

- Environment - The `DefaultAzureCredential` will read account information specified via [environment variables](#environment-variables) and use it to authenticate.
- Managed Identity - If the application is deployed to an Azure host with Managed Identity enabled, the `DefaultAzureCredential` will authenticate with that account.
- IntelliJ - If the developer has authenticated via Azure Toolkit for IntelliJ, the `DefaultAzureCredential` will authenticate with that account.
- Visual Studio Code - If the developer has authenticated via the Visual Studio Code Azure Account plugin, the `DefaultAzureCredential` will authenticate with that account.
- Azure CLI - If the developer has authenticated an account via the Azure CLI `az login` command, the `DefaultAzureCredential` will authenticate with that account.

## Examples
You can find more examples of authenticating with different Token Credentials below:

* [Authenticate with Azure in Developer Environment](./identity_env_auth.md)
* [Authenticate with Service Principal](./identity_service_principal_auth.md)
* [Authenticate applications hosted in Azure](./identity_azure_hosted_auth.md)
* [Authenticate with User Credentials](./identity_user_auth.md)


### Authenticating with `DefaultAzureCredential`
This example demonstrates authenticating the `SecretClient` from the [azure-security-keyvault-secrets][secrets_client_library] client library using the `DefaultAzureCredential`. There's also [a compilable sample](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-secrets/src/samples/java/com/azure/security/keyvault/secrets/IdentityReadmeSamples.java) to create a Key Vault secret client you can copy-paste.

```java
/**
* The default credential first checks environment variables for configuration.
* If environment configuration is incomplete, it will try managed identity.
*/
public void createDefaultAzureCredential() {
DefaultAzureCredential defaultCredential = new DefaultAzureCredentialBuilder().build();

// Azure SDK client builders accept the credential as a parameter
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://{YOUR_VAULT_NAME}.vault.azure.net")
  .credential(defaultCredential)
  .buildClient();
}
```

See more how to configure the `DefaultAzureCredential` on your workstation or Azure in [Configure DefaultAzureCredential](./identity_azure_hosted_auth.md#configure-defaultazurecredential).

### Authenticating a user assigned managed identity with `DefaultAzureCredential`
This example demonstrates authenticating the `SecretClient` from the [azure-security-keyvault-secrets][secrets_client_library] client library using the `DefaultAzureCredential`, deployed to an Azure resource with a user assigned managed identity configured.

See more about how to configure a user assigned managed identity for an Azure resource in [Enable managed identity for Azure resources](./identity_azure_hosted_auth.md#configure-managedidentitycredential).

```java
/**
* The default credential will use the user assigned managed identity with the specified client ID.
*/
public void createDefaultAzureCredentialForUserAssignedManagedIdentity() {
DefaultAzureCredential defaultCredential = new DefaultAzureCredentialBuilder()
  .managedIdentityClientId("<MANAGED_IDENTITY_CLIENT_ID>")
  .build();

// Azure SDK client builders accept the credential as a parameter
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://{YOUR_VAULT_NAME}.vault.azure.net")
  .credential(defaultCredential)
  .buildClient();
}
```

### Authenticating a user in Azure Toolkit for IntelliJ with `DefaultAzureCredential`
This example demonstrates authenticating the `SecretClient` from the [azure-security-keyvault-secrets][secrets_client_library] client library using the `DefaultAzureCredential`, on a workstation with IntelliJ IDEA installed, and the user has signed in with an Azure account to the Azure Toolkit for IntelliJ.

See more about how to configure your IntelliJ IDEA in [Sign in Azure Toolkit for IntelliJ for IntelliJCredential](./identity_env_auth.md#sign-in-azure-toolkit-for-intellij-for-intellijcredential).

```java
/**
* The default credential will use the KeePass database path to find the user account in IntelliJ on Windows.
*/
public void createDefaultAzureCredentialForIntelliJ() {
DefaultAzureCredential defaultCredential = new DefaultAzureCredentialBuilder()
// KeePass configuration required only for Windows. No configuration needed for Linux / Mac
  .intelliJKeePassDatabasePath("C:\\Users\\user\\AppData\\Roaming\\JetBrains\\IdeaIC2020.1\\c.kdbx")
  .build();

// Azure SDK client builders accept the credential as a parameter
SecretClient client = new SecretClientBuilder()
  .vaultUrl("https://{YOUR_VAULT_NAME}.vault.azure.net")
  .credential(defaultCredential)
  .buildClient();
}
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

