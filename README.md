# SALMON Configuration

## Table of Contents
* [Overview](#overview)
* [Configuration Files](#configuration-files)
* [Configuration Steps](#configuration-steps)
    1. [Copy Configuration Samples](#copy-configuration-samples)
    2. [Provide General Settings](#provide-general-settings)
    3. [Configure Monitoring Groups](#configure-monitoring-groups)
    4. [Specify Recipients and Subscriptions ](#specify-recipients-and-subscriptions)
    5. [[Optional] Provide Replacements for Rlaceholders](#provide-replacements-for-placeholders)


## Overview
This guide provides instructions on how to configure the SALMON project to suit your monitoring and alerting needs. \
The configuration files are structured as follows:
```
project_root/
│
└── config/
│
├── sample_settings/
│ ├── general.json
│ ├── monitoring_groups.json
│ ├── recipients.json
│ └── replacements.json
│
└── settings/
├── general.json
├── monitoring_groups.json
├── recipients.json
└── replacements.json
```
In the `sample_settings` directory, you will find sample configurations that you can use as templates. After copying them to the `settings` directory, fill in the necessary values according to your requirements (please refer to [Configuration Steps](#configuration-steps)). \
During the CDK deployment process, these configuration files from the `/config/settings` directory will be uploaded to the S3 bucket `s3-salmon-settings-<<stage-name>>` automatically. If any modifications are made to the configuration files locally, you would need to redeploy the stacks (`cdk deploy --all --context stage-name=<<stage-name>>`) in order to apply the changes. 


## Configuration Files

The project utilizes the following configuration files:
| File Name                | Description                                                                 |
|--------------------------|-----------------------------------------------------------------------------|
| `general.json`           | Contains general settings for the tooling environment, monitored environments, and delivery methods. |
| `monitoring_groups.json` | Defines monitoring groups with associated Glue jobs and SLA settings.       |
| `recipients.json`        | Specifies recipients for alerts and digests, along with their subscriptions to monitoring groups. |
| `replacements.json`      | [Optional] Contains a replacements list for placeholders in other setting JSON files. |


## Configuration Steps

Follow these steps to configure the project according to your requirements:

### 1. Copy Configuration Samples <a name="copy-configuration-samples"></a>
Navigate to the `/config/sample_settings` directory and copy the sample configuration files (`general.json`, `monitoring_groups.json`, `recipients.json`, and `replacements.json` if needed) to the `/config/settings` directory. \
**Note:** Always ensure that the settings you utilize are up-to-date.

### 2. Provide General Settings  <a name="provide-general-settings"></a>
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
- in the `name` parameter, enter the name of your Tolling environment where SALMON monitoring and alerting infrastructure will be located. \
**Note**: Here, `<<env>>` acts as a placeholder that represents the environment name. This allows you to specify a generic name for the tooling account while keeping the option to customize it based on the environment. To define the actual values for placeholders, you can use the `replacements.json` file (please refer to [Provide Replacements for Rlaceholders](#provide-replacements-for-placeholders)). This file serves as a mapping between placeholders and their corresponding values.
- in the `account_id`, `region` parameters, enter AWS region and account ID for this Tolling environment.
- in the `metrics_collection_interval_min`, enter an interval (in minutes) for extracting metrics from monitored environments.
- in the `digest_report_period_hours`, indicate how many recent hours should be covered in the daily digest report. Defaults to `24` hours.
- in the `digest_cron_expression`, enter the cron schedule to trigger the daily digest report. Defaults to `cron(0 8 * * ? *)`, every day at 8am UTC. \
\
    **Grafana Configuration** [Optional]: \
    \
    Only if the `grafana_instance` section exists, the Grafana stack will be deployed. If the Grafana deployment should be skipped, remove this `grafana_instance` nested configuration from the general settings.\
    If the Grafana stack should be deployed:
    - in the `grafana_vpc_id`, specify the ID of the Amazon VPC where the Grafana instance will be deployed. At least 1 public subnet required.
    - in the `grafana_security_group_id`, specify the ID of the security group that will be associated with the Grafana instance. Inbound access to Grafana’s default HTTP port: 3000 required. \
Additionally, several optional configurations are available to customize the Grafana deployment: 
    - `grafana_key_pair_name`: add this parameter and specify the name of the key pair to be associated with the Grafana instance. If not provided, a new key pair will be created during the stack deployment.
    - `grafana_bitnami_image`: add this parameter and specify the Bitnami Grafana image from AWS Marketplace. Defaults to `bitnami-grafana-10.2.2-1-r02-linux-debian-11-x86_64-hvm-ebs-nami`.
    - `grafana_instance_type`: add this parameter and specify the EC2 instance type for the Grafana instance. Defaults to `t3.micro`.

**Monitored Environments Configuration**:
- in the `name` parameter, enter the name of your Monitored environment. Refered in `monitoring_groups.json`.
- in the `account_id`, `region` parameters, specify AWS region and account ID of the account to be monitored.
- in the `metrics_extractor_role_arn` [Optional], specify IAM Role ARN to extract metrics for the resources running in another AWS account. If not specified - a default one is used (`arn:aws:iam::{account_id}:role/role-salmon-cross-account-extract-metrics-dev`). \
To add additional monitored environments, simply append another dictionary block with the same structure. 

**Delivery Methods Configuration**:
- in the `name` parameter, enter the name of your delivery method.
- in the `delivery_method_type` parameter, enter a delivery method type (AWS_SES, SMTP).
- in the `sender_email` parameter, enter the sender email for notifications and digests.
To add additional delivery method, simply append another dictionary block with the same structure.

### 3. Configure Monitoring Groups  <a name="configure-monitoring-groups"></a>
The `monitoring_groups.json` configuration file lists all resources to be monitored, grouped logically. For example, all glue jobs and lambdas related to Data Ingestion Pipeline.
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
**Monitoring Groups Configuration**: \
Inside group we list group elements with their properties. Properties list depends on the element type (e.g. Glue Job can have properies such as `name`, `sla_seconds`, `minimum_number_of_runs`). If you would like to monitor the resources with the same prefix (e.g., glue-pipeline1-ingest, glue-pipeline1-cleanse, glue-pipeline1-staging), you can simply describe those using wildcards: `glue-pipeline1-*`.
- in the `group_name` parameter, specify the name of your monitoring pipeline (group).
- the element `glue_jobs` should be adjusted in accordance with the monitoring resource type (e.g., `glue_jobs`, `step_functions`, `lambda_functions`, `glue_workflows`, `glue_catalogs`, `glue_crawlers`). 
- in the `name` parameter, specify the resource name to be monitored (e.g., Glue Job name).
- in the `monitored_environment_name` parameter, enter the name of your monitored environment (listed in the general settings).
- in the `sla_seconds`, enter If execution time exceeds "sla_seconds", run will be counted as "sla miss" in report. If omitted or equals to 0 - not checked.
- in the `minimum_number_of_runs`, enter Least number of runs expected. If it's less, then comments is shown in report and job is marked as problematic. If omitted or equals to 0 - not checked. 

### 4. Specify Recipients and Subscriptions  <a name="specify-recipients-and-subscriptions"></a> 
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
- in the `recipient` parameter, enter an e-mail address of a person / delivery list to receive glue job failure notifications or daily digest reports.   
**Note**: e-mail address must be verified in AWS SES.
- in the `delivery_method` parameter, specify the delivery method name.
- in the `monitoring_group` parameter, enter the ID of your management account.
- in the `alerts`, enter whether recipient wants to receive e-mail on glue job failure (true/false)
- in the `digest`, subscription flag for daily digest (true/false)

### 5. [Optional] Provide Replacements for Rlaceholders <a name="provide-replacements-for-placeholders"></a> 
The `replacements.json` file provides replacements list for placeholders in other setting JSON files. Placeholders inside general and other settings should be in double curly brackets (e.g. `<<value>>`). For example, we defined the value for `<<env>>` as `dev`. This means that during the deployment, wherever the `<<env>>` placeholder is used, it will be replaced with `dev`.

```json
{
    "<<env>>": "dev",
    "<<tooling_account_id>>": "323432554",
    "<<sender_email>>": "salmon-no-reply@soname.de"
}
```
Using the placeholders provides flexibility and consistency in the configuration management. It allows you to define the generic configurations that can be easily customized for different environments or scenarios. This helps to streamline the deployment process and ensures that configurations remain consistent across different deployments.
