## CloudFormation template for ECS pipeline IAM roles

The ECS Pipeline deploys your application to containers in ECS clusters.
To do so, CloudFormation needs set of IAM Roles and IAM Policies that work together to enable pipeline to create
CloudFormation stacks as well as to operate on ECS services. 

Running this CloudFormation template in your AWS Account enables you to create required IAM Roles and Policies
in an easy and automated way.

### Template parameters

| Parameter Name    | Description                                       | Required | Default    | Example            |
|-------------------|---------------------------------------------------|----------|------------|--------------------|
| TeamPrefix        | A unique prefix for all ECS services used by your team | Y   |            | myTeam             |
| CreateIamForEc2BackedEcsClusters | Whether to create IAM-related resources for running an EC2-backed ECS cluster | N | false | |
| BmxBaseRoleArns   | List of Brewmaster Base Role ARNs separated by comma. This parameter allows specific role to be assumed by listed Brewmasters (only one side of relationship). | Y | |arn:aws:iam::046979685931:role/brewmaster/base/brewmaster-base-mario,arn:aws:iam::046979685931:role/brewmaster/base/brewmaster-base-pipelines |
| VpcId             | id of the VPC to which the IAM roles have access  | Y        |            | vpc-1234           |


### Template outputs

The following attributes are available to consumers of the template:

| Output              | Description                                     |
|---------------------|-------------------------------------------------|
| TeamPrefix          | TeamPrefix input variable. Can be passed down down to service pipelines |
| HostedZoneId        | The Route53 hosted zone id |
| ExecutionRoleArn    | Arn of the service execution role |
| LoadBalancerLogsBucketName | The S3 bucket where logs for all of the team's apps / services will be located |
| InstanceProfileArn  | Arn of the instance profile for the team's ECS clusters |
| BrewmasterEcsRoleArn | Arn of the ECS role assumed by brewmaster to perform deployment-related operation for ECS services and related resources |
| HostedZoneDnsBase   | DNS base name to use when creating records for the team |

