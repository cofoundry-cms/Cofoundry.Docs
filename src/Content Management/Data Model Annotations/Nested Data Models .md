Nested data models allow you to define a model that can be used as child property of another data model. 

This is useful for creating rich data structures for your block types and custom entities, and enables the creation of more complex components such as multi-layered menus, advanced carousels and other nested content scenarios.

Currently the only data model attribute that supports nested data models is the [`[NestedDataModelCollection]`](#nesteddatamodelcollection) attribute.

## Defining a Nested Data Model

A nested data model is simply a class that inherits from `INestedDataModel`. You can add any properties you like and annotate them in the same way you would with any other data model. You can even recursively reference the same nested data model inside itself, which can be useful for building hierarchical data structures such as menus.

The following example represents one slide in a carousel:

```csharp
using System.ComponentModel.DataAnnotations;
using Cofoundry.Domain;

public class CarouselSlideDataModel : INestedDataModel
{
    [PreviewImage]
    [Display(Description = "Image to display as the background tot he slide.")]
    [Required]
    [Image]
    public int ImageId { get; set; }

    [PreviewTitle]
    [Required]
    [Display(Description ="Title to display in the slide.")]
    [MaxLength(100)]
    public string Title { get; set; }

    [Display(Description ="Formatted text to display in the slide.")]
    [Required]
    [Html(HtmlToolbarPreset.BasicFormatting)]
    public string Text { get; set; }
}
```

Note that the [display preview](Display-Preview) set of attributes can be used to control which properties display as columns in a selection grid.

The next section shows you how you can use this data model to create the data model for a carousel. 

## [NestedDataModelCollection]

The `[NestedDataModelCollection]` attribute is used to markup a property that contains a collection of nested data model types.

#### Optional Properties

- **MinItems:** The minimum number of items that need to be included in the collection. 0 indicates no minimum.
- **MaxItems:** The maximum number of items that can be included in the collection. 0 indicates no maximum.
- **IsOrderable:** Set to true to allow the collection ordering to be set by an editor using a drag and drop interface. Defaults to false.

#### Example

This example uses the `CarouselSlideDataModel` defined in the last section.

```csharp
using System.ComponentModel.DataAnnotations;
using Cofoundry.Domain;

public class CarouselDataModel : IPageBlockTypeDataModel, IPageBlockTypeDisplayModel
{
    [MaxLength(100)]
    [Required]
    public string Title { get; set; }

    [Required]
    [NestedDataModelCollection(IsOrderable = true, MinItems = 2, MaxItems = 6)]
    public ICollection<CarouselSlideDataModel> Items { get; set; }
}
```

#### Further Examples

You can find a full walk through of nested data models on [our blog](https://www.cofoundry.org/blog/14/introducing-nested-data-models), and more examples in our [Menus](https://github.com/cofoundry-cms/Cofoundry.Samples.Menus) and [Page Block Types](https://github.com/cofoundry-cms/Cofoundry.Samples.PageBlockTypes) samples projects.