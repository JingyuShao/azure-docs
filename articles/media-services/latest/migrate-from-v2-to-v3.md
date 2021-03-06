---
title: Migrate from Azure Media Services v2 to v3 | Microsoft Docs
description: This article describes changes that were introduced in Azure Media Services v3 and shows differences between two versions.
services: media-services
documentationcenter: na
author: Juliako
manager: femila
editor: ''
tags: ''
keywords: azure media services, stream, broadcast, live, offline

ms.service: media-services
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 11/04/2018
ms.author: juliako
---

# Migration guidance for moving from Media Services v2 to v3

This article describes changes that were introduced in Azure Media Services (AMS) v3, shows differences between two versions, and provides the migration guidance.

If you have a video service developed today on top of the [legacy Media Services v2 APIs](../previous/media-services-overview.md), you should review the following guidelines and considerations prior to migrating to the v3 APIs. There are many benefits and new features in the v3 API that improve the developer experience and capabilities of Azure Media Services. However, as called out in the  [Known Issues](#known-issues) section of this article, there are also some limitations due to changes between the API versions. This page will be maintained as the Media Services team makes continued improvements to the v3 APIs and addresses the gaps between the versions. 

## Benefits of Media Services v3

### API is more approachable

*  v3 is based on a unified API surface, which exposes both management and operations functionality built on Azure Resource Manager. Azure Resource Manager templates can be used to create and deploy Transforms, Streaming Endpoints, LiveEvents, and more.
* [Open API (aka Swagger) Specification](https://aka.ms/ams-v3-rest-sdk) document.
    Exposes the schema for all service components, including file-based encoding.
* SDKs available for [.NET](https://aka.ms/ams-v3-dotnet-ref), .NET Core, [Node.js](https://aka.ms/ams-v3-nodejs-ref), [Python](https://aka.ms/ams-v3-python-ref), [Java](https://aka.ms/ams-v3-java-ref), [Go](https://aka.ms/ams-v3-go-ref), and Ruby.
* [Azure CLI](https://aka.ms/ams-v3-cli-ref) integration for simple scripting support.

### New features

* For file-based Job processing, you can use a HTTP(S) URL as the input.
    You do not need to have content already stored in Azure, nor do you need to create Assets.
* Introduces the concept of [Transforms](transforms-jobs-concept.md) for file-based Job processing. A Transform can be used to build reusable configurations, to create Azure Resource Manager Templates, and isolate processing settings between multiple customers or tenants.
* An Asset can have [multiple StreamingLocators](streaming-locators-concept.md) each with different Dynamic Packaging and Dynamic Encryption settings.
* [Content protection](content-key-policy-concept.md) supports multi-key features.
* You can stream live events that are up to 24 hours long.
* New Low Latency live streaming support on LiveEvents.
* LiveEvent Preview supports Dynamic Packaging and Dynamic Encryption. This enables content protection on Preview as well as DASH and HLS packaging.
* LiveOutput is simpler to use than the Program entity in the v2 APIs. 
* You have role-based access control (RBAC) over your entities. 

## Changes from v2

* For Assets created with v3, Media Services supports only the [Azure Storage server-side storage encryption](https://docs.microsoft.com/azure/storage/common/storage-service-encryption).
    * You can use v3 APIs with Assets created with v2 APIs that had [storage encryption](../previous/media-services-rest-storage-encryption.md) (AES 256) provided by Media Services.
    * You cannot create new Assets with the legacy AES 256 [storage encryption](../previous/media-services-rest-storage-encryption.md) using v3 APIs.
* The v3 SDKs are now decoupled from the Storage SDK, which gives you more control over the version of Storage SDK you want to use and avoids versioning issues. 
* In the v3 APIs, all of the encoding bit rates are in bits per second. This is different than the v2 Media Encoder Standard presets. For example, the bitrate in v2 would be specified as 128 (kbps), but in v3 it would be 128000 (bits/second). 
* Entities AssetFiles, AccessPolicies, and IngestManifests do not exist in v3.
* ContentKeys is no longer an entity, it is now a property of the StreamingLocator.
* Event Grid support replaces NotificationEndpoints.
* The following entities were renamed
    * JobOutput replaces Task, and is now part of a Job.
    * LiveEvent replaces Channel.
    * LiveOutput replaces Program.
    * StreamingLocator replaces Locator.
* LiveOutputs do not need to be started explicitly, they start on creation and stop when deleted. Programs worked differently in the v2 APIs, they had to be started after creation.

## Feature gaps with respect to v2 APIs

The v3 API has the following feature gaps with respect to the v2 API. Closing the gaps is work in progress.

* The [Premium Encoder](../previous/media-services-premium-workflow-encoder-formats.md) and the legacy [media analytics processors](../previous/media-services-analytics-overview.md) (Azure Media Services Indexer 2 Preview, Face Redactor, etc.) are not accessible via v3.

    Customers who wish to migrate from the Media Indexer 1 or 2 preview can immediately use the AudioAnalyzer preset in the v3 API.  This new preset contains more features than the older Media Indexer 1 or 2. 

* Many of the advanced features of the Media Encoder Standard in v2 APIs are currently not available in v3, such as:
    * Clipping (for on-demand and live scenarios)
    * Stitching of Assets
    * Overlays
    * Cropping
    * Thumbnail Sprites
* LiveEvents with transcoding currently do not support Slate insertion mid-stream, custom presets, or ad marker insertion via API call. 

> [!NOTE]
> Please bookmark this article and keep checking for updates.

## Code differences

The following table shows the code differences between v2 and v3 for common scenarios.

|Scenario|V2 API|V3 API|
|---|---|---|
|Create an asset and upload a file |[v2 .NET example](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-aes/blob/master/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs#L113)|[v3 .NET example](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#L169)|
|Submit a job|[v2 .NET example](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-aes/blob/master/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs#L146)|[v3 .NET example](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#L298)<br/><br/>Shows how to first create a Transform and then submit a Job.|
|Publish an asset with AES encryption |1. Create ContentKeyAuthorizationPolicyOption<br/>2. Create ContentKeyAuthorizationPolicy<br/>3. Create AssetDeliveryPolicy<br/>4. Create Asset and upload content OR Submit job and use output asset<br/>5. Associate AssetDeliveryPolicy with Asset<br/>6. Create ContentKey<br/>7. Attach ContentKey to Asset<br/>8. Create AccessPolicy<br/>9. Create Locator<br/><br/>[v2 .NET example](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-aes/blob/master/DynamicEncryptionWithAES/DynamicEncryptionWithAES/Program.cs#L64)|1. Create Content Key Policy<br/>2. Create Asset<br/>3. Upload content or use Asset as JobOutput<br/>4. Create StreamingLocator<br/><br/>[v3 .NET example](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES/Program.cs#L105)|

## Known issues

* Currently, you cannot use the Azure portal to manage v3 resources. Use the [REST API](https://aka.ms/ams-v3-rest-sdk), CLI, or one of the supported SDKs.
* Today, Media Reserved Units can only be managed using the Media Services v2 API. For more information, see [Scaling media processing](../previous/media-services-scale-media-processing-overview.md).
* Media Services entities created with the v3 API cannot be managed by the v2 API.  
* It is not recommended to manage entities that were created with v2 APIs via the v3 APIs. Following are examples of the differences that make the entities in two versions incompatible:   
    * Jobs and Tasks created in v2 do not show up in v3 as they are not associated with a Transform. The recommendation is to switch to v3 Transforms and Jobs. There will be a relatively short time period of needing to monitor the inflight v2 Jobs during the switchover.
    * Channels and Programs created with v2 (which are mapped to LiveEvents and LiveOutputs in v3) cannot continue being managed with v3. The recommendation is to switch to v3 LiveEvents and LiveOutputs at a convenient Channel Stop.
    
        Presently, you cannot migrate continuously running Channels.  
> [!NOTE]
> Please bookmark this article and keep checking for updates.

## Next steps

To see how easy it is to start encoding and streaming video files, check out [Stream files](stream-files-dotnet-quickstart.md). 

