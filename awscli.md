# AWS CLI

- [AWS CLI](#aws-cli)
  - [Configuration](#configuration)
  - [User-related](#user-related)
  - [Account-related](#account-related)
  - [Lightsail](#lightsail)

## Configuration

```sh
configure
```

## User-related

```sh
iam list-users
iam get-user --user-name=$username
```

## Account-related

```sh
budgets describe-budgets --account-id=$account_id
```

## Lightsail

```sh
lightsail get-instances
```
