Cofoundry has an object caching abstraction that it uses internally to store frequently accessed data. You can make use of this by requesting `IObjectCacheFactory` from the DI container.

`IObjectCacheFactory` will manage the cache state, so all we need to is request a cache store using a unique name:

```csharp
using Cofoundry.Core.Caching;

public class CacheSample()
{
    public CacheSample(IObjectCacheFactory cacheFactory)
    {
        var myCache = cacheFactory.Get("MyExampleCache");
    }
}
```

## Creating Modular Caches

Internally we modularize our caches by creating one for each entity type we're caching. This pattern is optional, but you might find it useful if you're using the cache in many places:

```csharp
public class ImageAssetCache : IImageAssetCache
{
    private const string IMAGE_ASSET_RENDER_DETAILS_CACHEKEY = "ImageAssetRenderDetails:";

    private readonly IObjectCache _cache;

    public ImageAssetCache(
        IObjectCacheFactory cacheFactory
        )
    {
        _cache = cacheFactory.Get("COF_ImageAssets");
    }

    public Task<ImageAssetRenderDetails> GetOrAddAsync(int imageAssetId, Func<Task<ImageAssetRenderDetails>> getter)
    {
        return _cache.GetOrAddAsync(IMAGE_ASSET_RENDER_DETAILS_CACHEKEY + imageAssetId, getter);
    }

    public ImageAssetRenderDetails GetOrAdd(int imageAssetId, Func<ImageAssetRenderDetails> getter)
    {
        return _cache.GetOrAdd(IMAGE_ASSET_RENDER_DETAILS_CACHEKEY + imageAssetId, getter);
    }

    public void Clear(int imageAssetId)
    {
        _cache.Clear(IMAGE_ASSET_RENDER_DETAILS_CACHEKEY + imageAssetId);
    }

    public void Clear()
    {
        _cache.Clear();
    }
}
```

## IObjectCacheFactory Implementations

The default implementation is `InMemoryObjectCacheFactory` which uses an in-memory cache that is not designed to be used in a multi-server (web-farm) deployment. 

*Currently there are no additional cache plugin packages, but an `IObjectCacheFactory` implementation should be fairly straightforward. We hope to add additional distributed caching options soon - see [Issue #46](https://github.com/cofoundry-cms/cofoundry/issues/46)*
