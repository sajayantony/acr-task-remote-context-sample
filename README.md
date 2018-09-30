# ACR Task from a Remote context

ACR Tasks can be created or run from a remote context. 
The context can be a zipped tar archive. 
The task runtime downloads the context and will execute the task file or the build.

```bash
az acr run -f mytask.yaml https://example.com/context.tar.gz
```

## Creating and uploading the context

1. Create a tar and upload the repo to an azure blob
1. Generate a SAS URL for the blob
1. Schedule a run with the SAS URL as the remote context

### Upload the context

The script below uploads the current folder as a tar.gz. 

1. Tar the contents of this directory
1. Create a Azure storage container and obtain the connections string for a storage account
1. Uploads the blob

```bash
STORAGE_ACCOUNT=
CONTAINER_NAME=context
CONTENT_FILE=content.tar.gz
tar  --exclude='./.git' --exclude "./.DS_Store" -czvf  ./content.tar.gz .
STORAGE_CONNECTION_STRING=$(az storage account show-connection-string -n $STORAGE_ACCOUNT -o tsv)
az storage container create -n context  --connection-string $STORAGE_CONNECTION_STRING
az storage blob upload -c context -f $CONTENT_FILE -n $CONTENT_FILE --connection-string $STORAGE_CONNECTION_STRING
```

### Generate SAS URL from the context

The below helper script is to generate a SAS URL for for the uploaded context. The SAS URL has an expiration of 1 day. This SAS URL is used to run the [task](acr-task.yaml) on the configured Azure Container Registry.

```bash
STORAGE_ACCOUNT=
CONTAINER_NAME=context
CONTENT_FILE=content.tar.gz
URL=$(az storage account show -n $STORAGE_ACCOUNT --query "primaryEndpoints.blob" -o tsv)$CONTAINER_NAME/$CONTENT_FILE?
STORAGE_CONNECTION_STRING=$(az storage account show-connection-string -n $STORAGE_ACCOUNT -o tsv)
SAS_TTL=$(date -v "+1d" -u +%FT%TZ)
SAS_TOKEN=$(az storage blob generate-sas -c $CONTAINER_NAME -n $CONTENT_FILE --permissions r --expiry $SAS_TTL --connection-string $STORAGE_CONNECTION_STRING -o tsv)
SAS_URL="$URL$SAS_TOKEN"
#curl -o ./download-context.tar.gz "$SAS_URL"
```

### RUN ACR Task from a remote context

> Ensure you have run `az configure --defaults acr=<REGISTRY>`

```bash
az acr run -f acr-task.yaml "$SAS_URL"
```
