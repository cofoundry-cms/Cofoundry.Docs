There are a few places in Cofoundry that auto-generate data entry forms from annotated POCO data models, such as [Page Block Types](/content-management/page-block-types) and [Custom Entities](/content-management/custom-entities).  

## Primitive Types

Primitive types such as `string` and `int` will be given a basic editor that corresponds with their type, which can be overridden by adding a data annotation, for example a `string` property automatically generates a single line text input, but you can use the `[MultiLineText]` attribute to turn this into a textarea.

## .Net DataAnnotations

Most of the standard .Net DataAnnotations data annotations like `[Required]`, `[MaxLength(10)]` and `[Display(Name="Title")]` should work as expected. 

Models are always validated on the server, and most of these annotations will have client-side validation equivalents 

## Built-in Cofoundry Attributes

Cofoundry has a range of built in data annotations that either enhance existing types or compliment Cofoundry entities like images and documents. 

#### UI/Validation Attributes

- **CheckboxList:** Use this to decorate a collection property to indicate it should be rendered as a list of checkbox inputs in the admin UI. The collection type  should use the same type as the associated option values. The option source could be an `Enum` type, or a class that inherits from either `IListOptionSource` (a static set of options) or `IListOptionApiSource` (options generated from an API request).
- **Color:** Use this to decorate a string field and provide a UI hint to the admin interface to display an html editor field. Toolbar options can be specified in the constructor and the CustomToolbar property can be used to show a completely custom toolbar.
- **CustomEntity:** Use this to decorate an integer and indicate that it should be the id for a custom entity of a specific type.
- **CustomEntityCollection:** Allows a user to pick multiple custom entities of a specific type, which can optionally have 'drag and drop' ordering. Designed to be applied to an `ICollection<int>` property.
- **CustomEntityMultiTypeCollection:** Use this to decorate a collection of `CustomEntityIdentity` types and indicate that it should be a collection of custom entities for a mix of custom entity types. Optional parameters indicate whether the collection is sortable.
- **Date:** Use this to decorate a DateTime field and provide a UI hint to the admin interface to display a date picker field
- **Document:** Use with an (optionally nullable) integer to indicate this is for the id of a DocumentAsset. A non-null integer indicates this is a required field. Optional parameters allow you to restricts the file extensions permitted to be selected.
- **DocumentCollection:** Use this to decorate an integer collection of DocumentAssetIds and indicate that it should be a collection of document assets. The editor allows for sorting of linked document assets and you can set filters for restricting file types.
- **[Html:](Html-Data-Model-Annotation)** Use this to decorate a string field and provide a UI hint to the admin interface to display an html editor field. Toolbar options can be specified in the constructor and the CustomToolbar property can be used to show a completely custom toolbar.
- **Image:** Use with an (optionally nullable) integer to indicate this is for the id of an ImageAsset. A non-null integer indicates this is a required field. Optional parameters allow the search filter to be restricted e.g. width/height etc
- **ImageCollection:** Use this to decorate an integer array of ImageAssetIds and indicate that it should be a collection of image assets. The editor allows for sorting of linked assets and you can set filters for restricting image sizes.
- **MultiLineText:** Use this to decorate a string field and provide a UI hint to the admin interface to display a text area field
- **NestedDataModelCollection:** Use this to decorate a collection of INestedDataModel objects, allowing them to be edited in the admin UI. Optional parameters indicate whether the collection is sortable.
- **Number:** Use this to decorate a numeric field and provide a UI hint to the admin interface to display an html5 number field. The step property can be used to specify the precision of the number e.g. 2 decimal places
- **Placeholder:** Use this to provide a UI hint to the admin interface to add a placeholder attribute to an html input field.
- **PreviewTitle:** Indicates the property of a model that can be used as a title, name or short textual identifier. Typically this is used in a grid of items to identify the row.
- **PreviewImage:** Indicates the property of a model that can be used as the main image when displaying the model. Typically this is used in a grid of items to show an image representation of the row.
- **PreviewDescription:** Indicates the property of a model that can be used as a description field. Typically this is used in a grid of items to describe the item.
- **RadioListList:** Use this to decorate a collection property to indicate it should be rendered as a radio input list in the admin UI. The collection type should use the same type as the associated option values. The option source could be an `Enum` type, or a class that inherits from either `IListOptionSource` (a static set of options) or `IListOptionApiSource` (options generated from an API request).
- **SelectListList:** Use this to decorate a collection property to indicate it should be rendered as a select list (drop down list) in the admin UI. The collection type  should use the same type as the associated option values. The option source could be an `Enum` type, or a class that inherits from either `IListOptionSource` (a static set of options) or `IListOptionApiSource` (options generated from an API request).

#### Special Behaviour Attributes

- **EntityDependency:** This can be used to decorate an integer id property that links to another entity. The entity must have a definition that implements `IDependableEntityDefinition`. Defining relations allow the system to detect and prevent entities used in required fields from being removed.
- **EntityDependencyCollection:** This can be used to decorate an integer id array property that links to a set of entities. The entity must have a definition that implements `IDependableEntityDefinition`. Defining relations allow the system to detect and prevent entities used in required fields from being removed.
- **CustomEntityRouteData:** Use this to mark up a property in a custom entity data model, this property will be extracted and added to the cached `CustomEntityRoute` object and therefore make the property available for routing operations without having to re-query the db. E.g. for a blog post custom entity you could mark up a category Id and then use this in an `ICustomEntityRoutingRule` to create a /category/blog-post URL route

## Nested Data Models

Cofoundry allows you to nest data models, which is a simple and flexible way to create more complex data with an auto-generating admin interface. Nested data models should implement the `INestedDataModel` interface, and can then be included as a child property of your data model. 

The `NestedDataModelCollection` data attribute is used to markup the property and tell the admin UI to render an editor that let's you update the collection.

A good example of this is creating a page block type data model for a carousel, which contains a collection of items for each slide:

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
    public ICollection<CarouselItemDataModel> Items { get; set; }
}

public class CarouselItemDataModel : INestedDataModel
{
    [PreviewImage]
    [Image]
    public int ImageId { get; set; }

    [PreviewTitle]
    [Required]
    [MaxLength(100)]
    public string Title { get; set; }

    [PreviewDescription]
    [Required]
    [MultiLineText]
    [MaxLength(200)]
    public string Summary { get; set; }
}
```

Note that you can optionally specify whether the nested collection is sortable, and specify a minimum and maximum number of items in the collection.

You can also use the *preview* set of attributes to control which properties display as columns in the selection grid in the admin UI e.g. `PreviewTitle`, `PreviewDescription` or `PreviewImage`. If no preview attributes are found, it will look for a property named `Title` or fallback to displaying "Item 1", "Item 2" etc..

For more information we have a detailed [blog post on nested data models](https://www.cofoundry.org/blog/14/introducing-nested-data-models) with more examples.

## Creating Your Own Attributes

You can create your own attributes, but to get UI integration you'll need to to be familiar with Angular.js. We're not quite ready to document the Angular UI process just yet, but if you're interested you can see some examples of UI components on [github](https://github.com/cofoundry-cms/cofoundry/tree/master/src/Cofoundry.Web.Admin/Admin/Modules/Shared/Js/UIComponents)

TODO: Document admin UI development

When creating your annotations you should implement `IMetadataAttribute` and add any additional data you want to send to the angular client to the `AdditionalValues` collection. These will be rendered into your control with the 'cms-' prefix. 

The `TemplateHint` property can be used to customize the tag name that is output.

You can also add additional metadata to an existing attribute by creating an attribute class that implements `IModelMetaDataDecorator` 
