---
page_type: sample
languages:
- java
products:
- azure-event-hubs
description: "Azure Spring Cloud Stream Binder Sample project for Event Hub client library"
urlFragment: "azure-spring-cloud-sample-eventhubs-binder"
---

# Azure Spring Cloud Stream Binder for Event Hub Code Sample shared library for Java

## Key concepts

This code sample demonstrates how to use the Spring Cloud Stream Binder for Azure Event Hub.The sample app has two operating modes.
One way is to expose a Restful API to receive string message, another way is to automatically provide string messages.
These messages are published to an event hub. The sample will also consume messages from the same event hub.

## Getting started

Running this sample will be charged by Azure. You can check the usage and bill at 
[this link][azure-account].



### Create Azure resources

We have several ways to config the Spring Cloud Stream Binder for Azure
Event Hub. You can choose anyone of them.

>[!Important]
>
>  When using the Restful API to send messages, the **Active profiles** must contain `manual`.


#### Method 1: Connection string based usage

1.  Create [Azure Event Hubs][create-event-hubs].
    Please note `Basic` tier is unsupported. After creating the Azure Event Hub, you 
    can create your own Consumer Group or use the default "$Default" Consumer Group.

1.  Create [Azure Storage][create-azure-storage] for checkpoint use.

1.  Update [application.yaml][application.yaml].

    ```yaml
    spring:
      cloud:
        azure:
          eventhub:
            # Fill event hub namespace connection string copied from portal
            connection-string: [eventhub-namespace-connection-string] 
            # Fill checkpoint storage account name, access key and container 
            checkpoint-storage-account: [checkpoint-storage-account]
            checkpoint-access-key: [checkpoint-access-key]
            checkpoint-container: [checkpoint-container]
        stream:
          function:
            definition: consume;supply
          bindings:
            consume-in-0:
              destination: [eventhub-name]
              group: [consumer-group]
            supply-out-0:
              destination: [the-same-eventhub-name-as-above]
    ```

#### Method 2: Service principal based usage

1.  Create a service principal for use in by your app. Please follow 
    [create service principal from Azure CLI][create-sp-using-azure-cli].

1.  Create [Azure Event Hubs][create-event-hubs].
    Please note `Basic` tier is unsupported. After creating the Azure Event Hub, you
    can create your own Consumer Group or use the default "$Default" Consumer Group.

1.  Create [Azure Storage][create-azure-storage] for checkpoint use.

1.  Add Role Assignment for Event Hub, Storage Account and Resource group. See
    [Service principal for Azure resources with Event Hubs][role-assignment]
    to add role assignment for Event Hub, Storage Account, Resource group is similar.

    - Resource group: assign `Contributor` role for service principal.
    - Event Hub: assign `Contributor` role for service principal.
    - Storage Account: assign `Storage Account Key Operator Service Role` 
      role for service principal.
    
1.  Update [application-sp.yaml][application-sp.yaml].
    ```yaml
    spring:
      cloud:
        azure:
          client-id: [service-principal-id]
          client-secret: [service-principal-secret]
          tenant-id: [tenant-id]
          resource-group: [resource-group]
          eventhub:
            namespace: [eventhub-namespace]
            checkpoint-storage-account: [checkpoint-storage-account]
            checkpoint-container: [checkpoint-container]
        stream:
          function:
            definition: consume;supply
          bindings:
            consume-in-0:
              destination: [eventhub-name]
              group: [consumer-group]
            supply-out-0:
              destination: [the-same-eventhub-name-as-above]
    ```
    > We should specify `spring.profiles.active=sp` to run the Spring Boot application. 
        
#### Method 3: MSI credential based usage


##### Set up managed identity

Please follow [create managed identity][create-managed-identity] to set up managed identity.

##### Create other Azure resources

1.  Create [Azure Event Hubs][create-event-hubs].
    Please note `Basic` tier is unsupported. After creating the Azure Event Hub, you
    can create your own Consumer Group or use the default "$Default" Consumer Group.

1.  Create [Azure Storage][create-azure-storage] for checkpoint use.

1.  Add Role Assignment for Event Hub and Storage Account. See
    [Managed identities for Azure resources with Event Hubs][role-assignment]
    to add role assignment for Event Hub, Storage Account is similar.

    - Event Hub: assign `Contributor` role for managed identity.
    - Storage Account: assign `Storage Account Key Operator Service Role` 
      role for managed identity.

##### Update MSI related properties

1.  Update [application-mi.yaml][application-mi.yaml]
    ```yaml
    spring:
      cloud:
        azure:
          msi-enabled: true
          client-id: [the-id-of-managed-identity]
          resource-group: [resource-group]
          # Fill subscription ID copied from portal
          subscription-id: [subscription-id]
          eventhub:
            namespace: [eventhub-namespace]
            checkpoint-storage-account: [checkpoint-storage-account]
            checkpoint-container: [checkpoint-container]
        stream:
          function:
            definition: consume;supply
          bindings:
            consume-in-0:
              destination: [eventhub-name]
              group: [consumer-group]
            supply-out-0:
              destination: [the-same-eventhub-name-as-above]
    ```
    > We should specify `spring.profiles.active=mi` to run the Spring Boot application. 
      For App Service, please add a configuration entry for this.

##### Redeploy Application

If you update the `spring.cloud.azure.managed-identity.client-id`
property after deploying the app, or update the role assignment for
services, please try to redeploy the app again.

> You can follow 
> [Deploy a Spring Boot JAR file to Azure App Service][deploy-spring-boot-application-to-app-service] 
> to deploy this application to App Service

#### Enable auto create

If you want to auto create the Azure Event Hub and Azure Storage account instances,
make sure you add such properties 
(only support the service principal and managed identity cases):

```yaml
spring:
  cloud:
    azure:
      subscription-id: [subscription-id]
      auto-create-resources: true
      environment: Azure
      region: [region]
```

#### Enable sync message
To enable message sending in a synchronized way with Spring Cloud Stream 3.x, 
azure-spring-cloud-stream-binder-eventhubs supports the sync producer mode to get responses for sent messages. 
By enabling following configuration, you could use [StreamBridge][StreamBridge] for the synchronized message producing.

```yaml
spring:
  cloud:
    stream:
      eventhub:
        bindings:
          supply-out-0:
            producer:
              sync: true
```

## Examples

1.  Run the `mvn spring-boot:run` in the root of the code sample to get the app running.

1.  Send a POST request
    
        $ ### Send messages through imperative.  
        $ curl -X POST http://localhost:8080/messages/imperative/staticalDestination?message=hello
        $ curl -X POST http://localhost:8080/messages/imperative/dynamicDestination?message=hello
    
        $ ### Send messages through reactive.
        $ curl -X POST http://localhost:8080/messages/reactive?message=hello
    
    or when the app runs on App Service or VM

        $ ### Send messages through imperative.
        $ curl -d -X POST https://[your-app-URL]/messages/imperative/staticalDestination?message=hello
        $ curl -d -X POST https://[your-app-URL]/messages/imperative/dynamicDestination?message=hello
    
        $ ### Send messages through reactive.
        $ curl -d -X POST https://[your-app-URL]/messages/reactive?message=hello

1.  Verify in your app’s logs that a similar message was posted:

        New message received: 'hello', partition key: 2002572479, sequence number: 4, offset: 768, enqueued time: 2021-06-03T01:47:36.859Z
        Message 'hello' successfully checkpointed

1.  Delete the resources on [Azure Portal][azure-portal] to avoid unexpected charges.

## Troubleshooting

## Next steps

## Contributing


<!-- LINKS -->

[azure-account]: https://azure.microsoft.com/account/
[azure-portal]: https://ms.portal.azure.com/
[create-event-hubs]: https://docs.microsoft.com/azure/event-hubs/ 
[create-azure-storage]: https://docs.microsoft.com/azure/storage/ 
[create-sp-using-azure-cli]: https://github.com/Azure-Samples/azure-spring-boot-samples/blob/main/create-sp-using-azure-cli.md
[create-managed-identity]: https://github.com/Azure-Samples/azure-spring-boot-samples/blob/main/create-managed-identity.md
[deploy-spring-boot-application-to-app-service]: https://docs.microsoft.com/java/azure/spring-framework/deploy-spring-boot-java-app-with-maven-plugin?toc=%2Fazure%2Fapp-service%2Fcontainers%2Ftoc.json&view=azure-java-stable

[role-assignment]: https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal
[application-mi.yaml]: https://github.com/Azure-Samples/azure-spring-boot-samples/blob/main/eventhubs/azure-spring-cloud-stream-binder-eventhubs/eventhubs-binder/src/main/resources/application-mi.yaml
[application.yaml]: https://github.com/Azure-Samples/azure-spring-boot-samples/blob/main/eventhubs/azure-spring-cloud-stream-binder-eventhubs/eventhubs-binder/src/main/resources/application.yaml
[application-sp.yaml]: https://github.com/Azure-Samples/azure-spring-boot-samples/blob/main/eventhubs/azure-spring-cloud-stream-binder-eventhubs/eventhubs-binder/src/main/resources/application-sp.yaml
[StreamBridge]: https://docs.spring.io/spring-cloud-stream/docs/3.1.3/reference/html/spring-cloud-stream.html#_sending_arbitrary_data_to_an_output_e_g_foreign_event_driven_sources
