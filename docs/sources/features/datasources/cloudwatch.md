+++
title = "AWS CloudWatch"
description = "Guide for using CloudWatch in Grafana"
keywords = ["grafana", "cloudwatch", "guide"]
type = "docs"
aliases = ["/datasources/cloudwatch"]
[menu.docs]
name = "AWS Cloudwatch"
identifier = "cloudwatch"
parent = "datasources"
weight = 10
+++

# Using AWS CloudWatch in Grafana

Grafana ships with built in support for CloudWatch. You just have to add it as a data source and you will
be ready to build dashboards for you CloudWatch metrics.

## Adding the data source
![](/img/docs/cloudwatch/cloudwatch_add.png)

1. Open the side menu by clicking the the Grafana icon in the top header.
2. In the side menu under the `Dashboards` link you should find a link named `Data Sources`.

    > NOTE: If this link is missing in the side menu it means that your current user does not have the `Admin` role for the current organization.

3. Click the `Add new` link in the top header.
4. Select `CloudWatch` from the dropdown.
    > NOTE: If at any moment you have issues with getting this datasource to work and grafana is giving you undescriptive errors then dont forget to check your log file (try looking in /var/log/grafana/).

Name | Description
------------ | -------------
Name | The data source name, important that this is the same as in Grafana v1.x if you plan to import old dashboards.
Default | Default data source means that it will be pre-selected for new panels.
Auth Provider | Select `Access & secret key`, `Credentials file`, or `ARN`.
Credentials profile name | Specify the name of the profile to use (if you use `~/.aws/credentials` file), leave blank for default. This option was introduced in Grafana 2.5.1
Access key | AWS access key ID. Only shown when `Access & secret key` is selected.
Secret key | AWS secret access key. Only shown when `Access & secret key` is selected.
Default Region | Used in query editor to set region (can be changed on per query basis)
Custom Metrics namespace | Specify the CloudWatch namespace of Custom metrics
Assume Role ARN | Specify the ARN of the role to assume. Only shown when `ARN` is selected.

## Authentication

Grafana calls CloudWatch from the Grafana backend by using the AWS SDK. Select the
authentication provider that matches how the Grafana server receives AWS
credentials.

### Access and secret key

Select **Access & secret key** when you want Grafana to store a specific AWS
access key pair for this data source. The keys are saved as secure data source
settings and are used only by the Grafana backend.

### Credentials file

Select **Credentials file** to use a profile from the AWS credentials file on the
Grafana server. The profile name maps to a section in `~/.aws/credentials` for
the user that runs `grafana-server`; leave the profile name blank to use the
default profile.

Example content:

    [default]
    aws_access_key_id = asdsadasdasdasd
    aws_secret_access_key = dasdasdsadasdasdasdsa
    region = us-west-2

    [production]
    aws_access_key_id = asdsadasdasdasd
    aws_secret_access_key = dasdasdsadasdasdasdsa
    region = us-east-1

> **Note:** If you think you have the credentials file in the right place but
> Grafana still cannot authenticate, check the home directory of the
> `grafana-server` process. Package installs often run Grafana as a service user,
> so the credentials file might need to be readable from that user's home
> directory. The credentials file should have at most `0644` permissions.

### IAM roles

Currently all access to CloudWatch is done server side by the Grafana backend using the official AWS SDK. If your Grafana
server is running on AWS you can use IAM roles and authentication will be handled automatically.

Checkout AWS docs on [IAM Roles](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)

### Assume Role ARN

Select **ARN** when Grafana should call AWS Security Token Service (STS) and
assume a role before querying CloudWatch. The ARN must start with
`arn:aws:iam:`.

Grafana needs source credentials before it can assume the role. It checks these
sources on the server:

1. AWS environment variables.
2. The selected AWS credentials profile.
3. Remote credentials from the AWS runtime:
   - On ECS container tasks, Grafana uses the
     `AWS_CONTAINER_CREDENTIALS_RELATIVE_URI` environment variable and requests
     credentials from `169.254.170.2`.
   - Outside ECS, Grafana falls back to the EC2 instance metadata role provider.

Grafana caches the temporary STS credentials until their AWS expiration time.
When troubleshooting Assume Role failures, verify that the source credentials can
call `sts:AssumeRole`, that the target role trusts the source principal, and that
the CloudWatch data source region matches the AWS region you expect to query.


## Metric Query Editor

![](/img/docs/cloudwatch/query_editor.png)

You need to specify a namespace, metric, at least one stat, and at least one dimension.

## Templated queries
CloudWatch Datasource Plugin provides the following functions in `Variables values query` field in Templating Editor to query `region`, `namespaces`, `metric names` and `dimension keys/values` on the CloudWatch.

Name | Description
------- | --------
`regions()` | Returns a list of regions AWS provides their service.
`namespaces()` | Returns a list of namespaces CloudWatch support.
`metrics(namespace, [region])` | Returns a list of metrics in the namespace. (specify region for custom metrics)
`dimension_keys(namespace)` | Returns a list of dimension keys in the namespace.
`dimension_values(region, namespace, metric, dimension_key)` | Returns a list of dimension values matching the specified `region`, `namespace`, `metric` and `dimension_key`.
`ebs_volume_ids(region, instance_id)` | Returns a list of volume id matching the specified `region`, `instance_id`.
`ec2_instance_attribute(region, attribute_name, filters)` | Returns a list of attribute matching the specified `region`, `attribute_name`, `filters`.

For details about the metrics CloudWatch provides, please refer to the [CloudWatch documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/CW_Support_For_AWS.html).

## Example templated Queries

Example dimension queries which will return list of resources for individual AWS Services:

Service | Query
------- | -----
ELB | `dimension_values(us-east-1,AWS/ELB,RequestCount,LoadBalancerName)`
ElastiCache | `dimension_values(us-east-1,AWS/ElastiCache,CPUUtilization,CacheClusterId)`
RedShift | `dimension_values(us-east-1,AWS/Redshift,CPUUtilization,ClusterIdentifier)`
RDS | `dimension_values(us-east-1,AWS/RDS,CPUUtilization,DBInstanceIdentifier)`
S3 | `dimension_values(us-east-1,AWS/S3,BucketSizeBytes,BucketName)`

## ec2_instance_attribute JSON filters

The `ec2_instance_attribute` query take `filters` in JSON format.
You can specify [pre-defined filters of ec2:DescribeInstances](http://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeInstances.html).
Specify like `{ filter_name1: [ filter_value1 ], filter_name2: [ filter_value2 ] }`

Example `ec2_instance_attribute()` query

    ec2_instance_attribute(us-east-1, InstanceId, { "tag:Environment": [ "production" ] })

![](/img/docs/v2/cloudwatch_templating.png)

## Cost

Amazon provides 1 million CloudWatch API requests each month at no additional charge. Past this,
it costs $0.01 per 1,000 GetMetricStatistics or ListMetrics requests. For each query Grafana will
issue a GetMetricStatistics request and every time you pick a dimension in the query editor
Grafana will issue a ListMetrics request.


