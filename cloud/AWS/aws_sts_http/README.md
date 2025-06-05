# Template AWS STS by HTTP

## Overview

Template to monitor **AWS STS (Security Token Service)** using HTTP agent and custom JavaScript.

The template queries the `GetCallerIdentity` API to retrieve information about the IAM identity used to make the request.

Supported version: Zabbix 7.0 or higher

## Requirements

| Zabbix version | 7.0 and higher |
|----------------|------------------|

## Setup

### AWS IAM Permissions

This template requires an IAM policy with permission to access the STS API:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "sts:GetCallerIdentity"
      ],
      "Resource": "*"
    }
  ]
}
```

### Zabbix Configuration

This template uses `SCRIPT` and `DEPENDENT` item types. Zabbix agent is **not required** on the monitored instance.

Ensure that:
- Zabbix server or proxy can reach AWS STS endpoint.
- Required macros are set properly.

## Macros used

| Macro                       | Description                                                                 |
|----------------------------|-----------------------------------------------------------------------------|
| {$AWS.ACCESS.KEY.ID}       | AWS access key ID                                                           |
| {$AWS.SECRET.ACCESS.KEY}   | AWS secret access key (type: Secret text)                                   |
| {$AWS.REGION}              | AWS region (default: `us-east-1`)                                           |
| {$AWS.AUTH_TYPE}           | Authentication type: `access_key` or `role_base`                            |
| {$AWS.PROXY}               | (Optional) Proxy configuration                                              |

## Items

| Name                  | Key                 | Type       | Description                          |
|-----------------------|----------------------|------------|--------------------------------------|
| AWS Get Caller Identity | aws.sts.identity     | Script     | Retrieves identity information from AWS STS |
| AWS Account ID         | aws.sts.accounid     | Dependent  | Extracted account ID from identity info |

## Template links

_None_

## Notes

- Supports both access key-based and IAM role-based authentication.
- The script parses raw XML from the STS response to extract account ID, ARN, and user ID.
- Useful for auditing and identifying AWS API credentials used in Zabbix.
