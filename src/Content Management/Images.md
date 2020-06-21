## Managing Image Assets

The admin panel has a whole section devoted to managing images where users can add, update and delete images assets from a central place.

## Supporting Image Assets

Images assets are only supported by installing a plugin, this is because there isn't a good option for working with image files in .NET without using a 3rd party library.

Currently there are two plugin options:

- [Cofoundry.Plugins.Imaging.SkiaSharp](https://github.com/cofoundry-cms/Cofoundry.Plugins.Imaging.SkiaSharp): Uses the [SkiaSharp](https://github.com/mono/SkiaSharp)/[Skia](https://skia.org/) libraries which are MIT/BSD licenced. Does not support animated GIF resizing and has limited options for configuration. It is supported on a wide range of platforms but may not be supported in some linux configurations without a custom build of the native libraries.
- [Cofoundry.Plugins.Imaging.ImageSharp](https://github.com/cofoundry-cms/Cofoundry.Plugins.Imaging.ImageSharp): Uses the [ImageSharp](https://github.com/SixLabors/ImageSharp) library which is dual licenced under Apache 2.0 and a reasonably priced commercial support licence. It's currently in beta but it is fully cross-platform, supports a wide range of formats including animated gifs and has a comprehensive range of configuration options.

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