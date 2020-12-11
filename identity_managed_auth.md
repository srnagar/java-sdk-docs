# Authenticating Azure Hosted Applications

The Azure Identity library provides Azure Active Directory token authentication support for applicaions hosted on Azure through a a set of TokenCredential implementations.


## DefaultAzureCredential
The `DefaultAzureCredential` is appropriate for most scenarios where the application is intended to ultimately be run in the Azure Cloud. This is because the `DefaultAzureCredential` combines credentials commonly used to authenticate when deployed, with credentials used to authenticate in a development environment. The `DefaultAzureCredential` will attempt to authenticate via the following mechanisms in order.

![DefaultAzureCredential authentication flow](https://github.com/Azure/azure-sdk-for-java/raw/master/sdk/identity/azure-identity/images/defaultazurecredential.png)

- Environment - The `DefaultAzureCredential` will read account information specified via [environment variables](#environment-variables) and use it to authenticate.
- Managed Identity - If the application is deployed to an Azure host with Managed Identity enabled, the `DefaultAzureCredential` will authenticate with that account.
- IntelliJ - If the developer has authenticated via Azure Toolkit for IntelliJ, the `DefaultAzureCredential` will authenticate with that account.
- Visual Studio Code - If the developer has authenticated via the Visual Studio Code Azure Account plugin, the `DefaultAzureCredential` will authenticate with that account.
- Azure CLI - If the developer has authenticated an account via the Azure CLI `az login` command, the `DefaultAzureCredential` will authenticate with that account.


### Configure `DefaultAzureCredential`

`DefaultAzureCredential` supports a set of configurations through setters on the `DefaultAzureCredentialBuilder` or environment variables.

- Setting environment variables `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, and `AZURE_TENANT_ID` as defined in [Environment Variables](#environment-variables) configures the `DefaultAzureCredential` to authenticate as the service principal specified by the values.
- Setting `.managedIdentityClientId(String)` on the builder or environment variable `AZURE_CLIENT_ID` configures the `DefaultAzureCredential` to authenticate as a user defined managed identity, verse leaving them empty configures it to authenticate as a system assigned managed identity.
- Setting `.tenantId(String)` on the builder or environment variable `AZURE_TENANT_ID` configures the `DefaultAzureCredential` to authenticate to a specific tenant for shared token cache, Visual Studio Code and IntelliJ IDEA.
- Setting environment variable `AZURE_USERNAME` configures the `DefaultAzureCredential` to pick the corresponding cached token from the shared token cache.
- Setting `.intelliJKeePassDatabasePath(String)` on the builder configures the `DefaultAzureCredential` to read a specific KeePass file when authenticating with IntelliJ credentials.

### Authenticating with `DefaultAzureCredential`
This example demonstrates authenticating the `SecretClient` from the [azure-security-keyvault-secrets][secrets_client_library] client library using the `DefaultAzureCredential`.

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

## Managed Identity Credential
The Managed Identity authenticates the managed identity (system or user assigned) of an Azure resource. So, if the application is running inside an Azure resource that supports Managed Identity through `IDENTITY/MSI` and/or `IMDS` endpoints, then this credential will get your application authenticated and offers a great secretless authentication experience.

### Configure `ManagedIdentityCredential`

#### Cloud shell
A system assigned managed identity is enabled by default in [Azure Cloud Shell](https://shell.azure.com).

#### Virtual machines, App Services, Function Apps
Go to [Azure Portal](https://portal.azure.com) and navigate to your resource. You should see an "Identity" tab:

You will be able to configure either system assigned or user assigned identities. For user assigned identities, the client ID of the managed identity must be used to create the `ManagedIdentityCredential` or `DefaultAzureCredential`.

#### Kubernetes Services (AKS)

Only user assigned identities are currently supported in AKS with the [AAD Pod Identity](https://github.com/Azure/aad-pod-identity) plugin. Please follow the instructions in the repo as it may change between versions.


see more about how to configure your Azure resource for managed identity in [Enable managed identity for Azure resources](https://github.com/Azure/azure-sdk-for-java/wiki/Set-up-Your-Environment-for-Authentication#enable-managed-identity-for-azure-resources)


### Authenticating in Azure with managed identity
This examples demonstrates authenticating the `SecretClient` from the [azure-security-keyvault-secrets][secrets_client_library] client library using the `ManagedIdentityCredential` in a virtual machine, app service, function app, cloud shell, service fabric, arc, or AKS environment on Azure, with system assigned, or user assigned managed identity enabled.

```java
/**
* Authenticate with a managed identity.
*/
public void createManagedIdentityCredential() {
ManagedIdentityCredential managedIdentityCredential = new ManagedIdentityCredentialBuilder()
.clientId("<USER ASSIGNED MANAGED IDENTITY CLIENT ID>") // only required for user assigned
.build();

// Azure SDK client builders accept the credential as a parameter
SecretClient client = new SecretClientBuilder()
.vaultUrl("https://{YOUR_VAULT_NAME}.vault.azure.net")
.credential(managedIdentityCredential)
.buildClient();
}
```
