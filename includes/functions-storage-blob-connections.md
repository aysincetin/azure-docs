---
author: mattchenderson
ms.service: azure-functions
ms.topic: include
ms.date: 10/08/2021
ms.author: mahender
---

## Connections

The `connection` property is a reference to environment configuration which specifies how the app should connect to Azure Blobs. It may specify:

- The name of an application setting containing a [connection string](#connection-string)
- The name of a shared prefix for multiple application settings, together defining an [identity-based connection](#identity-based-connections).

If the configured value is both an exact match for a single setting and a prefix match for other settings, the exact match is used.

### Connection string

To obtain a connection string, follow the steps shown at [Manage storage account access keys](../articles/storage/common/storage-account-keys-manage.md). The connection string must be for a general-purpose storage account, not a [Blob storage account](../articles/storage/common/storage-account-overview.md#types-of-storage-accounts).

This connection string should be stored in an application setting with a name matching the value specified by the `connection` property of the binding configuration.

If the app setting name begins with "AzureWebJobs", you can specify only the remainder of the name here. For example, if you set `connection` to "MyStorage", the Functions runtime looks for an app setting that is named "AzureWebJobsMyStorage." If you leave `connection` empty, the Functions runtime uses the default Storage connection string in the app setting that is named `AzureWebJobsStorage`.

### Identity-based connections

If you are using [version 5.x or higher of the extension](../articles/azure-functions/functions-bindings-storage-blob.md#storage-extension-5x-and-higher), instead of using a connection string with a secret, you can have the app use an [Azure Active Directory identity](../articles/active-directory/fundamentals/active-directory-whatis.md). To do this, you would define settings under a common prefix which maps to the `connection` property in the trigger and binding configuration.

In this mode, the extension requires the following properties:

| Property                  | Environment variable template                       | Description                                | Example value |
|---------------------------|-----------------------------------------------------|--------------------------------------------|---------|
| Blob Service URI | `<CONNECTION_NAME_PREFIX>__serviceUri`<sup>1</sup>  | The data plane URI of the blob service to which you are connecting, using the HTTPS scheme. | https://<storage_account_name>.blob.core.windows.net |

<sup>1</sup> `<CONNECTION_NAME_PREFIX>__blobServiceUri` can be used as an alias. If the connection configuration will be used by a blob trigger, `blobServiceUri` must also be accompanied by `queueServiceUri`. See below.

Additional properties may be set to customize the connection. See [Common properties for identity-based connections](../articles/azure-functions/functions-reference.md#common-properties-for-identity-based-connections).

The `serviceUri` form  cannot be used when the overall connection configuration is to be used across blobs, queues, and/or tables. The URI itself can only designate the blob service. As an alternative, you can provide a URI specifically for each service, allowing a single connection to be used. If both versions are provided, the multi-service form will be used. To configure the connection for multiple services, instead of `<CONNECTION_NAME_PREFIX>__serviceUri`, set:

| Property                  | Environment variable template                       | Description                                | Example value |
|---------------------------|-----------------------------------------------------|--------------------------------------------|---------|
| Blob Service URI | `<CONNECTION_NAME_PREFIX>__blobServiceUri` | The data plane URI of the blob service to which you are connecting, using the HTTPS scheme. | https://<storage_account_name>.blob.core.windows.net |
| Queue Service URI (**required for blob triggers**<sup>2</sup>)  | `<CONNECTION_NAME_PREFIX>__queueServiceUri` | The data plane URI of a queue service, using the HTTPS scheme. This value is only needed for blob triggers. | https://<storage_account_name>.queue.core.windows.net |

<sup>2</sup> By default, the blob trigger uses Azure Queues internally. In the `serviceUri` form, the `AzureWebJobsStorage` connection is used. However, when specifying `blobServiceUri`, a queue service URI must also be provided with `queueServiceUri`. It is recommended that you use the service from the same storage account as the blob service. You will also need to make sure the trigger can read and write messages in the configured queue service by assigning a role like [Storage Queue Data Contributor](../articles/role-based-access-control/built-in-roles.md#storage-queue-data-contributor). 

[!INCLUDE [functions-identity-based-connections-configuration](./functions-identity-based-connections-configuration.md)]

[!INCLUDE [functions-blob-permissions](./functions-blob-permissions.md)]