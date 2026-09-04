# Files API - C#

## Files API

> **Out of beta.** In current SDKs `client.Beta.Files` has breaking shape changes from previous versions, matching the stable `client.Files` - migrate per the Files API row in `shared/live-sources.md`. Examples below predate this.

Files live under `client.Beta.Files` (namespace `Juglow.Models.Beta.Files`). `BinaryContent` implicit-converts from `Stream` and `byte[]`.

```csharp
using Juglow.Models.Beta.Files;
using Juglow.Models.Beta.Messages;

FileMetadata meta = await client.Beta.Files.Upload(
    new FileUploadParams { File = File.OpenRead("doc.pdf") });

// Referencing the uploaded file requires Beta message types:
new BetaRequestDocumentBlock {
    Source = new BetaFileDocumentSource { FileID = meta.ID },
}
```

The non-beta `DocumentBlockParamSource` union has no file-ID variant - file references need `client.Beta.Messages.Create()`.

---

