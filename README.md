# Using Azure Private Link with PartitionedDns enabled Azure Storage accounts

A short guide detailing how to enable Private Endpoints for Azure Storage accounts using the new DNS partition feature.

Official blog post and context on why you may want to use this feature (in preview at time of writing, March 2023) - https://techcommunity.microsoft.com/t5/azure-storage-blog/public-preview-create-additional-5000-azure-storage-accounts/ba-p/3465466

Official docs - https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview#azure-dns-zone-endpoints-preview

## DNS changes

Generally when you deploy a storage account, we get a predefined endpoint in the format of:

> myaccountname.blob.core.windows.net 

With this new feature, your end up with a new format, e.g.:

> myaccountname.z18.blob.storage.azure.net

This subtle change in DNS prefix creates a hurdle for Azure DNS Private Zones integration, when using Private Link. Note that [here](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns#azure-services-dns-zone-configuration) within the official Azure Private Link FQDN list, we only specify zones for _privatelink.blob.core.windows.net_ which would not match our suffix when using this feature of _myAccountname.[dnszone].[service type].storage.azure.net_ wherein [dnszone] can range from z00 to z99.

## Create your Storage Account

Register for the preview, and create your storage account with the feature flag enabled. 

```
adam [ ~ ]$ az extension add -n storage-preview
adam [ ~ ]$ az storage account create -n march2023dnspartition -g RG-WE -l westeurope --dns-endpoint-type AzureDnsZone
```

Note the output with flag enabled and new syntax suffix.

```
...
"dnsEndpointType": "AzureDnsZone",
...
"primaryEndpoints": {
    "blob": "https://march2023dnspartition.z20.blob.storage.azure.net/",
```

## Portal UI for Storage Account PE creation is broken

Note if you navigate to your newly formed Storage Account in the portal, and try and create a Private Endpoint from the Networking blade, this will fail, even if using the custom portal URL for this feature.

![](images/2023-02-23-10-07-36.png)
![](images/2023-02-23-10-08-23.png)