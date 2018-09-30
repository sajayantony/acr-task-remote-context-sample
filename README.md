# acr-task-remote-context-sample

Create a build or run a task from a remote context. 

## Creating and uploading the context

1. The script below will tar and upload the repo to an azure storage blob.
1. The task will be executed and will reference the blob using a SAS URL.

### Upload the context

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

```bash
az acr run -f acr-task.yaml "$SAS_URL"
```
