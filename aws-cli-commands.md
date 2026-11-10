
# AWS CLI Commands — S3 Backup and Recovery

This document contains the AWS CLI commands used in my S3 Backup and Recovery project.

Run these commands in Windows PowerShell. Confirm the bucket name and AWS region before running them.

## Project Configuration

| Setting | Value |
|---|---|
| AWS Service | Amazon S3 |
| Bucket Name | `secure-cloud-backup-system24` |
| AWS Region | `ap-south-1` (Mumbai) |
| S3 Prefix | `backups/` |
| IAM User | `s3-backup-test-user` |
| Test File | `test-backup.txt` |

## 1. Configure and Verify AWS Credentials

Configure AWS CLI using the dedicated IAM user's credentials.

```powershell
aws configure
```

Verify the active AWS identity:

```powershell
aws sts get-caller-identity
```

**Security:** Never store AWS access keys or secret access keys in this file or commit them to GitHub.

## 2. Create and Inspect the Local Test File

Create the test file:

```powershell
Set-Content -Path .\test-backup.txt -Value "test backup file"
```

View its contents:

```powershell
Get-Content .\test-backup.txt
```

## 3. Upload the File to Amazon S3

Upload the test file to the `backups/` prefix:

```powershell
aws s3 cp .\test-backup.txt s3://secure-cloud-backup-system24/backups/
```

## 4. Upload a Second Version

Modify the local file:

```powershell
Set-Content -Path .\test-backup.txt -Value "This is my second backup version"
```

Upload it again using the same S3 object key:

```powershell
aws s3 cp .\test-backup.txt s3://secure-cloud-backup-system24/backups/
```

Since bucket versioning is enabled, uploading the updated file creates another version of the same object.

## 5. Optional: List Objects in the S3 Prefix

```powershell
aws s3 ls s3://secure-cloud-backup-system24/backups/
```

**Permission note:** The upload-only IAM policy used in this project does not grant `s3:ListBucket`. This command can return `AccessDenied`. That does not mean the upload failed.

Use the Amazon S3 Console to inspect uploaded objects.

## 6. Optional: List Object Versions

```powershell
aws s3api list-object-versions --bucket secure-cloud-backup-system24 --prefix backups/test-backup.txt
```

**Permission note:** This command requires `s3:ListBucketVersions`, which is not included in the upload-only IAM policy.

Use a separately authorized IAM identity for this operation. Do not broaden the upload user's permissions just to run this optional command.

## 7. Verify Versioning and Recovery

Use the Amazon S3 Console:

1. Open the `secure-cloud-backup-system24` bucket.
2. Navigate to the `backups/` folder.
3. Locate `test-backup.txt`.
4. Open the object versions view.
5. Confirm that multiple versions are available.
6. Download an older version to verify recovery.

## Project Learnings

- Configured AWS CLI credentials using an IAM user.
- Verified the active AWS identity.
- Uploaded files to an Amazon S3 bucket.
- Practiced S3 bucket versioning.
- Uploaded multiple versions of the same object.
- Understood how IAM permissions restrict S3 operations.
- Practiced recovering an older object version.

---

**Project:** S3 Backup and Recovery  
**AWS Service:** Amazon S3  
**Region:** ap-south-1 (Mumbai)
