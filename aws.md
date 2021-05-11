# AWS

- [AWS](#aws)
  - [S3](#s3)

## S3

Bucket public URL format: https://mybucket.s3.amazonaws.com

When versioning is enabled, uploaded files have `VersionId` set; it's null otherwise.

When the storage class of an object is changed, a new version is created (even if versioning is disabled), with updated `LastModified`, `IsLatest` = true (and according `VersionId`).
