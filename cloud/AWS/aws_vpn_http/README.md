# Template AWS VPN by HTTP

## Overview

Template to monitor **AWS Site-to-Site VPN** using HTTP agent and custom JavaScript preprocessing.

The template uses AWS CloudWatch API via HTTP script to collect metrics for VPN tunnels.

Supported version: Zabbix 6.4 or higher

## Requirements

| Zabbix version | 6.4 and higher |
|----------------|----------------|

## Setup

### AWS IAM Permissions

This template requires an IAM policy with permission to access CloudWatch and EC2 resources:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudwatch:GetMetricData",
        "ec2:DescribeVpnConnections"
      ],
      "Resource": "*"
    }
  ]
}
```

### Zabbix Configuration

This template uses `SCRIPT` and `DEPENDENT` item types, so Zabbix agent is **not required** on the monitored instance.

Make sure that:
- The Zabbix server or proxy has access to the AWS API.
- You set the required macros described below.

## Macros used

| Macro                       | Description                                                                 |
|----------------------------|-----------------------------------------------------------------------------|
| {$AWS.ACCESS.KEY.ID}       | AWS access key ID                                                           |
| {$AWS.SECRET.ACCESS.KEY}   | AWS secret access key (type: Secret text)                                   |
| {$AWS.REGION}              | AWS region (e.g., `ap-northeast-2`)                                         |
| {$AWS.VPN.ID}              | The VPN connection ID                                                       |
| {$AWS.AUTH_TYPE}           | Either `access_key` or `role_base`                                          |
| {$AWS.PROXY}               | (Optional) Proxy settings for the HTTP request                              |

## Items

| Name                | Key                       | Type       | Description |
|---------------------|---------------------------|------------|-------------|
| Get VPN Data        | aws.vpn.get_id            | Script     | Fetches region and VPN ID |
| Get metrics data    | aws.vpn.get_metrics       | Script     | Collects metric data from AWS CloudWatch |
| AWS Region          | aws.vpn.region            | Dependent  | Extracted region from get_id |
| AWS VPN ID          | aws.vpn.vpn_id            | Dependent  | Extracted VPN ID from get_id |
| TunnelDataIn        | aws.vpn.tunnel_data_in    | Dependent  | Received data from customer gateway (Bytes) |
| TunnelDataOut       | aws.vpn.tunnel_data_out   | Dependent  | Sent data to customer gateway (Bytes) |
| TunnelState         | aws.vpn.tunnel_state      | Dependent  | 0=DOWN, 1=UP or ESTABLISHED |

## Triggers

| Name                      | Expression                                            | Severity  |
|---------------------------|--------------------------------------------------------|-----------|
| 1 VPN Tunnel Down (10m)   | avg(/AWS-VPN-TEMPLATE/aws.vpn.tunnel_state,10m)<0.6   | High      |
| 2 VPN Tunnel Down (10m)   | avg(/AWS-VPN-TEMPLATE/aws.vpn.tunnel_state,10m)<0.1   | Disaster  |

## Template links

- AWS-STS-TEMPLATE

## Notes

- All communication is handled via Zabbix HTTP agent and JavaScript (UserParameter not required).
- Be sure to monitor API call limits and use IAM roles if possible for increased security.
- Ensure CloudWatch metrics are enabled for your VPN connections.
