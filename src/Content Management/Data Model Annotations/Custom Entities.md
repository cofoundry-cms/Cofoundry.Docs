There are three different types of custom entity data model annotations:

- [`[CustomEntity]`](#customentity)
- [`[CustomEntityCollection]`](#customentitycollection)
- [`[CustomEntityMultiTypeCollection]`](#customentitymultitypecollection)

Each of these are explained below:

## [CustomEntity]

The `[CustomEntity]` data annotation can be used to decorate an integer to indicate it is the id of a custom entity. The annotation is for a specific custom entity type and the definition code must be supplied in the attribute constructor.

A nullable integer indicates this is an optional field, while a non-null integer indicates this is a required field. 

#### Example

```csharp
public class ExampleDataModel : ICustomEntityDataModel
{
    [CustomEntity(CategoryCustomEntityDefinition.DefinitionCode)]
    public int RequiredCategoryId { get; set; }

    [CustomEntity(CategoryCustomEntityDefinition.DefinitionCode)]
    public int? OptionalCategoryId { get; set; }
}
```

## [CustomEntityCollection]

The `[CustomEntityCollection]` data annotation can be used to decorate a collection of integers, indicating the property represents a set of custom entity ids. The annotation is for a specific custom entity type and the definition code must be supplied in the attribute constructor.

#### Optional parameters:

- **IsOrderable:** Set to true to allow the collection ordering to be set by an editor using a drag and drop interface. Defaults to false.

#### Example:

```csharp
public class ExampleDataModel : ICustomEntityDataModel
{
    [CustomEntityCollection(CategoryCustomEntityDefinition.DefinitionCode)]
    public ICollection<int> CategoryIds { get; set; }

    [CustomEntityCollection(CategoryCustomEntityDefinition.DefinitionCode, IsOrderable = true)]
    public ICollection<int> OrderableCategoryIds { get; set; }
}
```

## [CustomEntityMultiTypeCollection]

The `[CustomEntityMultiTypeCollection]` data annotation can be used to decorate a collection of `CustomEntityIdentity` objects, indicating the property represents a set of custom entities of mixed types. The entity types must be defined in the attribute constructor by passing in custom entity definition codes.

#### Optional parameters:

- **IsOrderable:** Set to true to allow the collection ordering to be set by an editor using a drag and drop interface. Defaults to false.

#### Example:

```csharp
public class ExampleDataModel : ICustomEntityDataModel
{
    [CustomEntityMultiTypeCollection(
            BlogPostCustomEntityDefinition.DefinitionCode, 
            CaseStudyCustomEntityDefinition.DefinitionCode)]
    public ICollection<CustomEntityIdentity> CustomEntityIds { get; set; }

    [CustomEntityMultiTypeCollection(
            BlogPostCustomEntityDefinition.DefinitionCode, 
            CaseStudyCustomEntityDefinition.DefinitionCode, 
            IsOrderable = true)]
    public ICollection<CustomEntityIdentity> OrderableCustomEntityIds { get; set; }
}
```