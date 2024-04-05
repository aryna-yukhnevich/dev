# Grafana

* [Overview](#overview)
    * [Quick Start](#quick-start)
    * [Configuration Structure](#conf-structure)

## Overview
This article provides an overview of the Grafana instance and the pre-provisioned dashboards included with its deployment.

###  Grafana Instance Deployment:
The Grafana stack is an optional one, so when you decide to deploy it, you will receive a Grafana instance along with the default SALMON dashboards.
Refer to our [Configuration guide](docs/configuration.md) for more details on Grafana configuration.

Upon launching the Grafana instance, the default dashboards will be accessible within a dedicated folder named "Default SALMON Dashboards". These dashboards include:
 - **CloudWatch Logs Dashboard with Alert Events**: This dashboard provides insights into CloudWatch logs with alert events.
 - **Timestream Metrics Dashboard**: This dashboard is based on the Timestream table with Metrics and implemented for each Resource type (Glue Jobs, Glue Workflows, Lambda Functions, Step Functions in the ver 1.0).

### Customization and Administration
The Grafana Admin can also create additional users and assign necessary permissions. Moreover, these pre-provisioned dashboards are customizable to meet specific customer requirements. Adjustments can be made and saved, and the dashboards can be easily re-created through the export and import of the required JSON dashboard model.


### Sign In
To sign in to Grafana, go to `http://<grafana-instance-public-ip>:3000` (the URL will be shown in the output during the cdk deployment). You can also navigate to the EC2 instance named as `ec2-slamon-grafana-instance-<<stage-name>>` to check the public IP. \
The Grafana password and username are stored in the corresponding secret named as `secret-salmon-grafana-password-<<stage-name>>` in AWS Secrets Manager.


## Timestream Dashboards
Timestream dashboards based on the Timestream tables as a datasource:
\
    ![Timestream Dashboard](images/timestream-dashboard.png "Timestream Dashboard sample") 


These dashboards offer:
- **Filtering Options**: Users can filter data based on Measure Name, Monitored Env, and Resource name to focus on specific metrics and resources.
- **Time Range**: By default, the dashboards display data for the last 24 hours in Coordinated Universal Time (UTC), but users have the flexibility to adjust the time range as needed.
- **Customized Dashboards**: Similar dashboards are created for Glue Workflows, Lambda Functions, and Step Functions, each tailored to showcase corresponding metrics.

## CloudWatch Dashboard
CloudWatch dashboard utilizes CloudWatch log group as a datasource:
\
    ![CloudWatch Dashboard](images/cloudwatch-dashboard.png "CloudWatch Dashboard sample")

Key features include:
- **Filtering Options**: Users can filter data by Resource type, Event Result, Event Status, and Resource name, enabling precise analysis of logs.
- **Execution Information**: The dashboards include an Execution Info column providing links to specific resource runs or log groups, such as for Lambda functions.
- **JSON View**: Event Details can be inspected in JSON format for detailed examination of log data.
- **Time Range**: Similar to Timestream dashboards, the default time range for CloudWatch dashboards is the last 24 hours in UTC, but users have the flexibility to adjust it according to their requirements.
