# Using real Azure Blob Storage

Local development and `docker compose` use the **Azurite** emulator by default, so you can try uploads with no Azure account. For marking you should also test against real Azure.

## 1. Create the storage account
1. Azure portal > **Storage accounts** > **Create**.
2. Pick a subscription/resource group, a unique name (lower-case letters/numbers), a nearby region (e.g. South Africa North) and **Standard / LRS** redundancy.
3. On the **Advanced** tab make sure **Allow enabling anonymous access on individual containers** is ticked (the site shows images with a plain `<img>` tag, so blobs must be publicly readable). If you create the account with this off, enable it later under *Configuration*.
4. Create the account.

You do not need to create the container yourself - the API creates `raceday-images` (public access level *Blob*) the first time an image is uploaded.

## 2. Get the connection string
Storage account > **Security + networking** > **Access keys** > copy a **Connection string**.

> Never commit this string. Anyone who has it can read and write your storage.

## 3. Give it to the API

**Running locally from Visual Studio / `dotnet run`** (stored outside the repo with user-secrets):
```bash
cd src/RaceDay.Api
dotnet user-secrets set "BlobStorage:ConnectionString" "DefaultEndpointsProtocol=https;AccountName=<account>;AccountKey=<key>;EndpointSuffix=core.windows.net"
```

**Running with Docker Compose** - create a `.env` file next to `docker-compose.yml` (it is git-ignored):
```
BLOB_CONNECTION_STRING=DefaultEndpointsProtocol=https;AccountName=<account>;AccountKey=<key>;EndpointSuffix=core.windows.net
BLOB_PUBLIC_BASE_URL=https://<account>.blob.core.windows.net
```
Then `docker compose up --build`.

## 4. Check it worked
1. Sign in as an Organiser, create an event and choose a banner image.
2. Open the event: the banner should appear.
3. In the Azure portal open the storage account > **Containers** > `raceday-images` > `events/<id>/...` - you should see the blob.
4. Take a screenshot for your video.

## Troubleshooting
| Symptom | Likely cause |
|---|---|
| "Image storage is not configured" (503) | `BlobStorage:ConnectionString` is empty |
| "Image storage rejected the upload" (502) | wrong key, or anonymous access is disabled on the account (error `PublicAccessNotPermitted`) |
| Upload works but the image is broken | container is not public, or `BLOB_PUBLIC_BASE_URL` points at the wrong host |
