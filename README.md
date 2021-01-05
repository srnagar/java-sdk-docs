# Java SDK Docs staging repo

Planned structure is as follows:

* [Get Started](https://docs.microsoft.com/azure/developer/java/sdk/java-sdk-azure-get-started)
* Concepts
  * [Overview](overview.md)
  * Common concepts
    * [HTTP clients & pipeline](http_client_pipeline.md)
    * [Asynchronous programming](asynchronous_programming.md)
    * [Pagination & iteration](pagination.md)
    * [Long-Running operations](long_running_operations.md)
    * [Proxying](proxying.md)
    * [Tracing](tracing.md)
  * Identity & Authentication (Newly written content borrowing from existing content [here](identity_overview.md) and [here](identity_examples.md))
    * [Authenticate with Azure using Azure Identity](identity.md)
    * [Authenticating with Azure in Developer Environment](identity_env_auth.md)
    * [Authenticating Azure Hosted Applications](identity_azure_hosted_auth.md)
    * [Authenticating with Service Principal](identity_service_principal_auth.md)
    * [Authenticating with User Credentials](identity_user_auth.md)
  * Logging
    * [Overview](logging.md)
    * [java.util.logging](java-util-logging.md)
    * [Logback](logback.md)
    * [Log4J](log4j.md)
* Integrations
  * [Spring](https://docs.microsoft.com/azure/developer/java/spring-framework/spring-boot-starters-for-azure)
  * [MicroProfile](https://docs.microsoft.com/azure/developer/java/eclipse-microprofile/)
* Troubleshooting
  * [Network issues](troubleshooting_network.md)
* Quick Starts
  * (Links to Java SDK track two quick starts for each library)
* Code Samples
  * [Samples for Management Libraries](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/resourcemanager/docs/SAMPLE.md)
  * (Updated list of code samples that are for track two Java SDKs)
* [Reference](https://docs.microsoft.com/java/api/overview/azure/?view=azure-java-stable)

## Future Topics

* Common concepts
  * [Context](context.md)
  * [Configuration](configuration.md)
  * [Serialization](serialization.md)  
* Identity & Authentication
  * Authenticating with Azure Stack
  * Troubleshooting authentication issues
