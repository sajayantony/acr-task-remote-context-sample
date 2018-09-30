# acr-task-remote-context-sample

Create a build or run a task from a remote context. 

## Creating and uploading the context

1. The script below will tar and upload the repo to an azure storage blob.
1. The task will be executed and will reference the blob using a SAS URL.

```bash
CONTAINER_NAME=context
CONTENT_FILE=content.tgz
tar  --exclude='./.git' -cvf  ./content.tgz .
STORAGE_CONNECTION_STRING=$(az storage account show-connection-string -n $STORAGE_ACCOUNT -o tsv)
az storage container create -n context  --connection-string $STORAGE_CONNECTION_STRING
az storage blob upload -c context -f $CONTENT_FILE -n $CONTENT_FILE --connection-string $STORAGE_CONNECTION_STRING
```

```bash
SAS_TTL=$(date -v "1d" -u +%FT%TZ)
SAS_URL=$(az storage blob generate-sas -c $CONTAINER_NAME -n $CONTENT_FILE --permissions r  --expiry $SAS_TTL --connection-string $STORAGE_CONNECTION_STRING)
```
