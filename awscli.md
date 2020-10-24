# AWS CLI

- [AWS CLI](#aws-cli)
  - [Tool configuration](#tool-configuration)
  - [Resources](#resources)
    - [EC2](#ec2)
    - [IAM](#iam)
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
