The `[Image]` data annotation can be used to decorate an integer to indicate this is for the id of an image asset. 

A nullable integer indicates this is an optional field, while a non-null integer indicates this is a required field. Optional parameters allow you to restrict the image that can be selected by attributes such as width, height and tags.

Example:

```csharp
public class ExampleDataModel : ICustomEntityDataModel
{
    /// <summary>
    /// A non nullable property indicated the image is required.
    /// </summary>
    [Image]
    public int ExampleRequiredImageId { get; set; }

    /// <summary>
    /// A nullable property indicates the image is optional.
    /// </summary>
    [Image]
    public int? ExampleOptionalImageId { get; set; }

    /// <summary>
    /// This image is required to be exactly 600 pixels wide, but
    /// can be any height.
    /// </summary>
    [Image(Width = 600)]
    public int? Example600WidthImageId { get; set; }

    /// <summary>
    /// This image is required to be exactly 400 x 400.
    /// </summary>
    [Image(Width = 400, Height = 400)]
    public int? Example400SquareImageId { get; set; }

    /// <summary>
    /// This image must be at least 800 x 600, but can
    /// be larger.
    /// </summary>
    [Image(MinWidth = 800, MinHeight = 600)]
    public int? ExampleMinDimensionsImageId { get; set; }

    /// <summary>
    /// Here we set the dimensions that the image is displayed
    /// at in the admin panel. This is useful if you want to 
    /// preview the image in a specific crop ratio, without
    /// restricting the size of images uploaded.
    /// </summary>
    [Image(PreviewWidth = 1024, PreviewHeight = 480)]
    public int? ExampleWithPreviewRatioImageId { get; set; }

    /// <summary>
    /// This image is restricted to images labelled with these
    /// specific tags.
    /// </summary>
    [Image("cats", "dogs")]
    public int? ExampleTagImageId { get; set; }
}
```