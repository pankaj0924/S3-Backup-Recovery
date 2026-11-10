# AWS S3 Backup and Recovery System

A hands-on AWS project demonstrating backup uploads to Amazon S3, restricted IAM access, S3 Versioning, and recovery of an older object version.

## Project objectives

- Store backup files in an Amazon S3 bucket.
- Enable S3 Bucket Versioning to retain earlier object versions.
- Use a dedicated IAM user rather than root credentials.
- Apply least-privilege permissions for uploads to a specific prefix.
- Use AWS CLI from Windows PowerShell to upload a backup.
- Download and inspect an older version of an object.

## Architecture

```text
Windows workstation
       |
       | Windows PowerShell + AWS CLI
       v
IAM user: s3-backup-test-user
       |
       | s3:PutObject (restricted to backups/*)
       v
S3 bucket: secure-cloud-backup-system24
       |
       v
backups/test-backup.txt
       |
       +--> Latest object version
       |
       +--> Previous object version
                    |
                    v
             Download old version
             and inspect recovered file
```

## AWS resources

| Setting | Value |
|---|---|
| AWS Region | Asia Pacific (Mumbai), `ap-south-1` |
| S3 bucket | `secure-cloud-backup-system24` |
| Backup prefix | `backups/` |
| IAM user | `s3-backup-test-user` |
| Test object | `test-backup.txt` |
| Storage class observed | S3 Standard |

## 1. Create and configure the S3 bucket

Create the general purpose bucket `secure-cloud-backup-system24` in `ap-south-1`.

In the bucket properties:

1. Enable **Bucket Versioning**.
2. Confirm the bucket's default server-side encryption setting.
3. Create a folder/prefix named `backups/`.

S3 folders are represented by object-key prefixes. The uploaded object's key in this project is:

```text
backups/test-backup.txt
```

## 2. Configure least-privilege IAM access

A dedicated IAM user named `s3-backup-test-user` was used for the AWS CLI upload test.

The intended upload-only identity policy is:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "UploadBackupsOnly",
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::secure-cloud-backup-system24/backups/*"
    }
  ]
}
```

This policy allows object uploads only under the `backups/` prefix. It does not grant `s3:ListBucket`, object download, object deletion, or bucket administration.

**Important:** The policy above is the intended least-privilege policy for this project. Before publishing, compare it with the policy currently attached to the IAM user in the AWS Console. Do not claim the policy was verified unless you have checked the live policy.

## 3. Configure AWS CLI

Install AWS CLI v2 on the Windows workstation, then configure the dedicated IAM user's access keys locally:

```powershell
aws configure
```

Enter the IAM user's access key and secret access key when prompted. Set:

```text
Default region name: ap-south-1
Default output format: json
```

Verify the identity:

```powershell
aws sts get-caller-identity
```

The command returned the identity for `s3-backup-test-user`, confirming that the CLI credentials authenticated successfully.

**Credential safety:** Never paste access keys or secret keys into GitHub, README files, screenshots, or chat. Do not include your local AWS credentials file in this repository.

## 4. Create and upload a test backup

Create the local test file:

```powershell
Set-Content -Path .\test-backup.txt -Value "test backup file"
```

Read it back:

```powershell
Get-Content .\test-backup.txt
```

Upload it to S3:

```powershell
aws s3 cp .\test-backup.txt s3://secure-cloud-backup-system24/backups/
```

The upload completed successfully.

The IAM identity intentionally did not have `s3:ListBucket`. Therefore, this command is expected to be denied unless a separate listing permission is deliberately added:

```powershell
aws s3 ls s3://secure-cloud-backup-system24/backups/
```

The `AccessDenied` response for listing does not mean the earlier upload failed. The object was checked in the S3 Console instead.

## 5. Test S3 Versioning

Change the local file contents:

```powershell
Set-Content .\test-backup.txt "This is my second backup version"
```

Upload the same object key again:

```powershell
aws s3 cp .\test-backup.txt s3://secure-cloud-backup-system24/backups/
```

With bucket versioning enabled, uploading to the same key creates a new object version instead of replacing the older version permanently.

The S3 Console showed two versions of `backups/test-backup.txt`:

| Version shown | Size | Interpretation |
|---|---:|---|
| Latest | 34 bytes | Second uploaded version |
| Previous | 38 bytes | Original uploaded version |

The sizes are the values observed during this project run; they can differ if the file contents change.

## 6. Test recovery of an older version

In the S3 Console:

1. Open bucket `secure-cloud-backup-system24`.
2. Open the `backups/` prefix.
3. Turn on **Show versions**.
4. Select the older `test-backup.txt` version (38 bytes in the recorded test).
5. Download that version.
6. Open the downloaded file and inspect its contents.

The downloaded older version contained the original text, `test backup file`.

This demonstrates recovery by downloading a previous object version. It does **not** by itself make that older version the current/latest version. A separate copy operation would be needed if the goal were to restore the old content as the current object.

## 7. Evidence and screenshots

The `screenshots/` folder contains the S3 Console screenshot showing:

- The `backups/` prefix.
- **Show versions** enabled.
- Both versions of `test-backup.txt`.
- The observed version sizes.

Add further screenshots only after reviewing and redacting them:

- S3 bucket properties and versioning status.
- IAM user and attached policy (hide account-specific details where practical).
- Successful AWS CLI upload output.
- S3 object listing in the Console.
- Older-version download/recovery evidence.

## 8. Troubleshooting notes

### `NoSuchBucket`

The bucket name in the first upload command was mistyped. The correct bucket name is:

```text
secure-cloud-backup-system24
```

### `AccessDenied` for `s3:PutObject`

The IAM user initially lacked upload permission. The upload succeeded after the required permission was made available. Verify the live IAM policy before publishing this project.

### `AccessDenied` for `s3:ListBucket`

The upload-only policy does not grant bucket listing. This is expected least-privilege behavior and is separate from `s3:PutObject`.

## 9. Key learnings

- Configuring AWS CLI with a dedicated IAM identity.
- Understanding the difference between `s3:PutObject` and `s3:ListBucket`.
- Restricting object upload permissions to a bucket prefix.
- Uploading files with `aws s3 cp`.
- Understanding S3 object versioning.
- Downloading and inspecting a previous object version.
- Distinguishing an upload failure from a listing-permission failure.

## 10. Possible future improvements

- Automate recurring backups with a scheduled script or service.
- Configure S3 Lifecycle rules for noncurrent versions.
- Add monitoring and notifications for backup failures.
- Explore cross-region replication and recovery objectives.
- Add automated tests and a safe restore workflow.

## Conclusion

This project implemented and tested a basic S3 backup workflow using AWS CLI and a dedicated IAM identity. A test file was uploaded to the `backups/` prefix, a second upload created another version, and the older version was downloaded and inspected successfully.

The project is a learning lab, not a production backup design. Production use would require additional controls such as monitoring, retention policies, restore testing, and carefully reviewed IAM permissions.
