# AWS CLI

- [AWS CLI](#aws-cli)
  - [Tool configuration](#tool-configuration)
  - [Resources](#resources)
    - [EC2](#ec2)
    - [IAM](#iam)
    - [RDS](#rds)
    - [S3](#s3)
    - [Account-related](#account-related)
    - [Lightsail](#lightsail)
    - [Secrets Manager](#secrets-manager)
    - [CloudWatch log groups](#cloudwatch-log-groups)
    - [CloudWatch Events](#cloudwatch-events)

## Tool configuration

```sh
aws configure
```

## Resources

### EC2

```sh
# Show private shapshot ids; use `all` for public
#
aws ec2 describe-snapshots --restorable-by-user-ids self | grep '"SnapshotId"'

# Delete all public shapshots (in parallel) (won't succeed, but here for reference); continues on error
#
aws ec2 describe-snapshots --restorable-by-user-ids all |
  perl -ne 'print "$1\n" if /"SnapshotId": "(.*)"/' |
  xargs -I {} -n 1 -P 0 sh -c "aws ec2 delete-snapshot --snapshot-id {} || true"
```

### IAM

```sh
aws iam list-users
aws iam get-user --user-name=$username
```

### RDS

```sh
# Find RDS reservations expiry:
#
aws rds describe-reserved-db-instances \
  --region eu-west-1 \
  --reserved-db-instance-id my-reservation-id \
  --query 'ReservedDBInstances[*].[StartTime,Duration]'
```

### S3

See `text_processing##jq` for JSON manipulation.

Print objects summary:

```sh
# The prefix is optional
$ aws s3 ls --summarize --human-readable "s3://$bucket/$prefix" --recursive | head -n 3

2019-03-26 21:07:11   26.8 KiB /prefix1/key0
2019-03-26 21:07:12   45.9 KiB /prefix1/key1
2018-02-28 13:14:50    1.7 KiB prefix2/key0

$ aws s3api list-object-versions --bucket $bucket --query \
  '
    {
      PreviousVersions: {
        Size: sum(Versions[?IsLatest==`false`].Size)
        Count: length(Versions[?IsLatest==`false`].Key)
      }
      DeleteMarkers: {
        Count: length(DeleteMarkers[].Key)
      }
    }
  '

{
    "PreviousVersions": {
        "Size": 2064087,
        "Count": 6
    },
    "DeleteMarkers": {
        "Count": 35
    }
}
```

List object versions and informations (note: there may be a limit to the max number of items that can be retrieved):

```sh
# --prefix is also available.
#
$ aws s3api list-object-versions --max-items 1 --bucket $bucket

{
    "Versions": [
        {
            "ETag": "\"etag\"",
            "Size": 27438,
            "StorageClass": "STANDARD",
            "Key": "/prefix/key1",
            "VersionId": "version1",
            "IsLatest": true,
            "LastModified": "2019-03-26T20:07:11.000Z",
            "Owner": {
                "DisplayName": "name",
                "ID": "id"
            }
        }
    ],
    "DeleteMarkers": [
        {
            "Owner": {
                "DisplayName": "name",
                "ID": "id"
            },
            "Key": "key2",
            "VersionId": "version2",
            "IsLatest": true,
            "LastModified": "2019-01-24T22:53:47.000Z"
        },
        {
            "Owner": {
                "DisplayName": "name",
                "ID": "id"
            },
            "Key": "prefix/key3",
            "VersionId": "version3",
            "IsLatest": false,
            "LastModified": "2019-10-08T22:09:19.000Z"
        },
        {
            "Owner": {
                "DisplayName": "name",
                "ID": "id"
            },
            "Key": "prefix/key4",
            "VersionId": "version4",
            "IsLatest": true,
            "LastModified": "2020-02-06T15:15:22.000Z"
        }
    ],
    "NextToken": "eyJLZXlNYXJrZXIiOiBudWxsLCAiVmVyc2lvbklkTWFya2VyIjogbnVsbCwgImJvdG9fdHJ1bmNhdGVfYW1vdW50IjogMX0="
}
```

Delete object versions, from list file:

```sh
cat > $filename << JSON
{
  "Objects": [
    {
      "Key": "key1",
      "VersionId": "ver1"
    },
    {
      "Key": "key2",
      "VersionId": "ver2"
    }
  ],
  "Quiet": true
}
JSON

$ aws s3api delete-objects --bucket $bucket --delete "file://$filename"
```

### Account-related

```sh
aws budgets describe-budgets --account-id=$account_id
```

### Lightsail

```sh
aws lightsail get-instances
```

### Secrets Manager

```sh
aws secretsmanager list-secret-version-ids --secret-id=$secret_arn
```

### CloudWatch log groups

```sh
aws logs describe-log-groups
```

### CloudWatch Events

```sh
# List targets for rule
#
aws events list-targets-by-rule --rule $rule_name
```
