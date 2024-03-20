# Project Configuration
## Overview
This guide provides instructions on how to configure the SALMON project to suit your monitoring and alerting needs.

## Configuration Files

The project utilizes the following configuration files:
| File Name                | Description                                                                 |
|--------------------------|-----------------------------------------------------------------------------|
| `general.json`           | Contains general settings for the tooling environment, monitored environments, and delivery methods. |
| `monitoring_groups.json` | Defines monitoring groups with associated Glue jobs and SLA settings.       |
| `recipients.json`        | Specifies recipients for alerts and digests, along with their subscriptions to monitoring groups. |
| `replacements.json`      | *(Optional)* Contains a replacements list for placeholders in other setting JSON files. |


## Configuration Steps

Follow these steps to configure the project according to your requirements:

#### 1. Provide General settings
The  `general.json` configuration file sets up the tooling environment, monitored environments, and delivery methods.
```json
{
    "tooling_environment": {
        "name": "Tooling Account [<<env>>]",
        "account_id": "<<tooling_account_id>>",
        "region": "eu-central-1",
        "metrics_collection_interval_min": 5,
        "digest_report_period_hours" : 24, 
        "digest_cron_expression": "cron(0 8 * * ? *)",
        "grafana_instance": {
            "grafana_vpc_id": "<<grafana_vpc_id>>",
            "grafana_security_group_id": "<<grafana_security_group_id>>"
        }
    },
    "monitored_environments": [
        {
            "name": "Dept1 Account [<<env>>]",
            "account_id": "123456789",
            "region": "eu-central-1",
            "metrics_extractor_role_arn": "arn:aws:iam::123456789:role/role-salmon-cross-account-extract-metrics-dev"
        }
    ],
    "delivery_methods": [
        {
            "name": "aws_ses",
            "delivery_method_type": "AWS_SES",
            "sender_email" : "<<sender_email>>"
        }
    ]
}
```     
**Tooling Environment Configuration**:
- in the `name` parameter, enter the name of your Tolling environment where SALMON monitoring and alerting infrastructure will be located.
- in the `account_id` parameter, enter the ID of your AWS account.
- in the `region` parameter, enter the region name of your AWS account.
- in the `metrics_collection_interval_min`, enter an interval (in minutes) for extracting metrics from monitored environments.
- in the `digest_report_period_hours`, indicate how many recent hours should be covered in the daily digest report. Defaults to `24` hours.
- in the `digest_cron_expression`, enter the cron schedule to trigger the daily digest report. Defaults to `cron(0 8 * * ? *)`, every day at 8am UTC. \
\
    **Grafana Configuration** (optional): \
    Only if the `grafana_instance` section exists, the Grafana stack will be deployed. If the Grafana deployment should be skipped, remove this `grafana_instance` nested configuration from the general settings.\
    If the Grafana stack should be deployed:
    - in the `grafana_vpc_id`, specify the ID of the Amazon VPC where the Grafana instance will be deployed. At least 1 public subnet required.
    - in the `grafana_security_group_id`, specify the ID of the security group that will be associated with the Grafana instance. Inbound access to Grafana’s default HTTP port: 3000 required. \
    Additionally, several optional configurations are available to customize the Grafana deployment: 
    - `grafana_key_pair_name`: add this parameter and specify the name of the key pair to be associated with the Grafana instance. If not provided, a new key pair will be created during the stack deployment.
    - `grafana_bitnami_image`: add this parameter and specify the Bitnami Grafana image from AWS Marketplace. Defaults to `bitnami-grafana-10.2.2-1-r02-linux-debian-11-x86_64-hvm-ebs-nami`.
    - `grafana_instance_type`: add this parameter and specify the EC2 instance type for the Grafana instance. Defaults to `t3.micro`.

**Monitored Environments Configuration**:
- in the `name` parameter, enter the name of your Monitored environment. Refered in items in monitoring_groups.json.
- in the `account_id`, `region` parameters, specify AWS region and account ID of your Monitored account. Refers to AWS Account and Region to be monitored.
- in the `metrics_extractor_role_arn` (optional), specify IAM Role Arn to get access for metrics extraction. If not specified - a default one is used (arn:aws:iam::{account_id}:role/role-salmon-cross-account-extract-metrics-dev). \
Several Monitored accounts can be specified here. Add new dictionary with the same parameters for each  Monitored account.

**Delivery Methods Configuration**:
- in the `name` parameter, enter the name of your delivery method.
- in the `delivery_method_type` parameter, enter a delivery method type (AWS_SES, SMTP).
- in the `sender_email` parameter, enter the sender email for notifications and digests.
Several Delivery methods can be specified here. Add new dictionary with the same parameters for each delivery method type.

#### 2. Configure Monitoring Groups
The `monitoring_groups.json` configuration file defines groups of Glue jobs along with their SLA settings.
```json
{
    "monitoring_groups": [
        {
            "group_name": "pipeline_source1",
            "glue_jobs": [
                {
                    "name": "ds-source1-historical-data-load",
                    "monitored_environment_name": "Dept1 Account [<<env>>]",
                    "sla_seconds": 1200,
                    "minimum_number_of_runs": 0
                },
                {
                    "name": "ds-source1-source-to-raw-full-load",
                    "monitored_environment_name": "Data Lake Account [<<env>>]",
                    "sla_seconds": 1300,
                    "minimum_number_of_runs": 2
                }
            ]
        }
    ]
}
```
**Monitoring Groups Configuration**:
- in the `group_name` parameter, enter the ID of your management account.
- in the `glue_jobs` parameter, enter the ID of your management account.
- in the `name` parameter, enter the ID of your management account.
- in the `monitored_environment_name` parameter, enter the ID of your management account.
- in the `sla_seconds`, enter an interval (in minutes) for extracting metrics from monitored environments.
- in the `minimum_number_of_runs`, enter an interval (in minutes) for extracting metrics from monitored environments. \

#### 3. Specify Recipients and Subscriptions 
The `recipients.json` file specifies recipients for alerts and digests, along with their subscriptions to monitoring groups.
```json
{
    "recipients": [
        {
            "recipient": "vasilii_pupkin@salmon.com",
            "delivery_method": "local_smtp",            
            "subscriptions": [
                {
                    "monitoring_group": "pipeline_source1",
                    "alerts": true,
                    "digest": true
                }
            ]
        }
    ]
}
```
**Recipients Configuration**:
- in the `recipient` parameter, enter the ID of your management account.
- in the `delivery_method` parameter, enter the ID of your management account.
- in the `monitoring_group` parameter, enter the ID of your management account.
- in the `alerts`, enter an interval (in minutes) for extracting metrics from monitored environments.
- in the `digest`, enter an interval (in minutes) for extracting metrics from monitored environments. \

#### 4. Modify `replacements.json` configuration file.
The general configuration file sets up the tooling environment, monitored environments, and delivery methods.
```json
{
    "<<env>>": "dev",
    "<<tooling_account_id>>": "323432554",
    "<<sender_email>>": "salmon-no-reply@soname.de",
    "<<grafana_vpc_id>>": "vpc_id", 
    "<<grafana_security_group_id>>": "security_group_id"
}
```
