# Installing the Migration Tool

## Create backup storage

You can use vStorage as a backup storage.

* `region=hcm03`
* `s3Url=https://hcm03.vstorage.vngcloud.vn`
* `bucket=annd2-01`

Create a file `credentials-velero` with content:

```yaml
[default]
aws_access_key_id=<AWS_ACCESS_KEY_ID>
aws_secret_access_key=<AWS_SECRET_ACCESS_KEY>
```

## Installing Velero CLI

```bash
curl -OL https://github.com/vmware-tanzu/velero/releases/download/v1.13.2/velero-v1.13.2-linux-amd64.tar.gz
tar -xvf velero-v1.13.2-linux-amd64.tar.gz
cp velero-v1.13.2-linux-amd64/velero /usr/local/bin
```

## Install velero server in cluster

You have to install in source and target cluster

```bash
velero install \
    --provider aws \
    --plugins velero/velero-plugin-for-aws:v1.9.0 \
    --use-node-agent \
    --use-volume-snapshots=false \
    --secret-file ./credentials-velero \
    --bucket annd2-01 \
    --backup-location-config region=hcm03,s3ForcePathStyle="true",s3Url=https://hcm03.vstorage.vngcloud.vn
```
