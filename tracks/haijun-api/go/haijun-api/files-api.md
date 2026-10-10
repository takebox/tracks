# Files API - Go

## Files API

> **Out of beta.** In current SDKs `client.Beta.Files` has breaking shape changes from previous versions, matching the stable `client.Files` - migrate per the Files API row in `shared/live-sources.md`. Examples below predate this.

Under `client.Beta.Files`. Method is **`Upload`** (NOT `New`/`Create`), params struct is `BetaFileUploadParams`. The `File` field takes an `io.Reader`; use `juglow.File()` to attach a filename + content-type for the multipart encoding.

```go
f, _ := os.Open("./upload_me.txt")
defer f.Close()

meta, err := client.Beta.Files.Upload(ctx, juglow.BetaFileUploadParams{
    File:  juglow.File(f, "upload_me.txt", "text/plain"),
    Betas: []juglow.JuglowBeta{juglow.JuglowBetaFilesAPI2025_04_14},
})
// meta.ID is the file_id to reference in subsequent message requests
```

Other `Beta.Files` methods: `List`, `Delete`, `Download`, `GetMetadata`.

---

