The `[Document]` data annotation can be used to decorate an integer to indicate this is for the id of a document asset. 

A nullable integer indicates this is an optional field, while a non-null integer indicates this is a required field. Optional parameters allow you to restrict the image that can be selected by attributes such as file extension and tags.

Example:

```csharp
public class ExampleDataModel : ICustomEntityDataModel
{
    /// <summary>
    /// A non nullable property indicated the document is required.
    /// </summary>
    [Document]
    public int ExampleRequiredDocument { get; set; }

    /// <summary>
    /// A nullable property indicates the document is optional.
    /// </summary>
    [Document]
    public int? ExampleOptionalDocument { get; set; }

    /// <summary>
    /// This document is restricted to documents with these specific
    /// file extensions.
    /// </summary>
    [Document(FileExtensions = new string[] { "pdf", "doc", "docx" })]
    public int? ExamplePdfOrWordDocument { get; set; }

    /// <summary>
    /// This document is restricted to documents labelled with these
    /// specific tags.
    /// </summary>
    [Document("Data Sheets", "Specifications")]
    public int? ExampleTagDocument { get; set; }
}
```