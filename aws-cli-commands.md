# AWS CLI Commands — S3 Backup and Recovery

Run these commands in Windows PowerShell. Confirm the bucket name and AWS region before running them.

## Configure and verify credentials

```powershell
aws configure
aws sts get-caller-identity
```

Use the dedicated IAM user's credentials. Never put access keys in this file or commit them to GitHub.

## Create and inspect the local test file

```powershell
Set-Content -Path .\test-backup.txt -Value "test backup file"
Get-Content .\test-backup.txt
```

## Upload

```powershell
aws s3 cp .\test-backup.txt s3://secure-cloud-backup-system24/backups/
```

## Change the file and upload a second version

```powershell
Set-Content .\test-backup.txt "This is my second backup version"
aws s3 cp .\test-backup.txt s3://secure-cloud-backup-system24/backups/
```

## Optional: list objects

```powershell
aws s3 ls s3://secure-cloud-backup-system24/backups/
```

The upload-only IAM policy used in this lab does not grant `s3:ListBucket`, so this listing command can return `AccessDenied`. That does not invalidate a successful upload. Use the S3 Console to inspect versions unless listing permission is intentionally added.

## Optional: inspect object versions with a separately authorized identity

```powershell
aws s3api list-object-versions --bucket secure-cloud-backup-system24 --prefix backups/test-backup.txt
```

This requires `s3:ListBucketVersions`, which is not included in the upload-only policy. Do not broaden the upload user's permissions merely to run this optional command.
