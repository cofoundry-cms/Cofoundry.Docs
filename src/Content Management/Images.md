## Managing Image Assets

The admin panel has a whole section devoted to managing images where users can add, update and delete images assets from a central place.

## Supporting Image Assets

Images assets are only supported by installing a plugin, this is because there isn't a good option for working with images in .NET without using a 3rd party library.

Currently the only image plugin available is [Cofoundry.Plugins.Imaging.ImageSharp](https://github.com/cofoundry-cms/Cofoundry.Plugins.Imaging.ImageSharp), which you can add via [NuGet](https://www.nuget.org/packages/Cofoundry.Plugins.Imaging.ImageSharp/).

If you're interested in plugins for other imaging libraries or other 3rd party services do get in touch and let us know.

## Generating Image Urls

### From a View or Template

The [Cofoundry View Helper](Cofoundry-View-Helper) is the best way to access this:

```html
@using Cofoundry.Web

@model MyContentDisplayModel
@inject ICofoundryHelper<MyContentDisplayModel> Cofoundry

/* From a model */
<img src="@Cofoundry.Routing.ImageAsset(Model.ThumbnailImageAsset)">

/* From an id (see warning below) */
<img src="@await Cofoundry.Routing.ImageAssetAsync(3)">

/* Simple Resizing */
<img src="@Cofoundry.Routing.ImageAsset(Model.ThumbnailImageAsset, 200, 200)">

/* Resizing by passing in a constant of type IImageResizeSettings */
<img src="@Cofoundry.Routing.ImageAsset(Model.ThumbnailImageAsset, MyImageSizes.Thumbnail)">

```

Warning: Generating a url from just an image asset id involves getting more information about the image from the database. Although caching is used to speed this up, it is recommended that you try to include the full `ImageAssetRenderDetails` object in your view model to take advantage of batch requests and async methods.

### From Code

You can request `IImageAssetRouteLibrary` from the DI container and use this to generate urls. It is the same api used by the Cofoundry View Helper above.

```csharp
using Cofoundry.Domain;

public class ImageExample
{
    private IImageAssetRouteLibrary _imageAssetRouteLibrary;

    public ImageExample(IImageAssetRouteLibrary imageAssetRouteLibrary)
    {
        _imageAssetRouteLibrary = imageAssetRouteLibrary;
    }

    public string GetExampleUrl(IImageAssetRenderable image)
    {
        var url = _imageAssetRouteLibrary.ImageAsset(image);

        return url;
    }
}

```

## Getting Image Data

The simplest way to get image data is by resolving an instance of `IImageAssetRepository` from the DI container.

```csharp
using Cofoundry.Domain;

public class ImageExample
{
    private IImageAssetRouteLibrary _imageAssetRouteLibrary;
    private IImageAssetRepository _imageAssetRepository;

    public ImageExample(
        IImageAssetRouteLibrary imageAssetRouteLibrary,
        IImageAssetRepository imageAssetRepository
        )
    {
        _imageAssetRouteLibrary = imageAssetRouteLibrary;
        _imageAssetRepository = imageAssetRepository;
    }

    public Task<string> GetExampleUrl(int imageId)
    {
        var image = await _imageAssetRepository.GetImageAssetRenderDetailsByIdAsync(imageId);
        var url = _imageAssetRouteLibrary.ImageAsset(image);

        return url;
    }
}
```

Alternatively you can resolve an instance of `CofoundryDbContext` from the DI container and use Entity Framework to completely customize your query.