## CloudFormation template for ECS service infrastructure

When deploying your application to ecs you will need to have an load balancer upstream from your containers for
distributing the workload.  This CloudFormation template stands up an application load balancer for you.
The template will output the load balancers DNS name, among other values.

### Template parameters

| Parameter Name | Description                                                                       | Required | Default | Example                        |
|----------------|-----------------------------------------------------------------------------------|----------|---------|--------------------------------|
| TeamPrefix     | A unique prefix for all ECS services used by your team.                           | Y        |         | myTeam                         |
| AppName        | The name of your app                                                              | N        |         | myApp                          |
| VpcId          | The id of the VPC where the load balancer resides                                 | Y        |         | vpc-abcd                       |
| SecurityGroups | The ids of the security groups for the loadbalancer associated with the ECS tasks | Y        |         | ['subnet-1234', 'subnet-2356'] |
| LogsBucketName | The name of the S3 bucket to forward load balancer logs                           | N        |         | myApp-lb                       |
| ListenerPort   | The port which the load balancer will listen on                                   | N        | 80      | 443                            |

### Template outputs

The following attributes are available to consumers of the template:

| Output       | Description                                          |
|--------------|------------------------------------------------------|
| Arn          | ARN of the ALB                                       |
| DnsName      | The DNSName of the ALB                               |
| ZoneId       | The zone id of the ALB                               |
| ListenerArn  | ARN of the configured ALB listener                   |
| ListenerPort | Port of the configured ALB listener                  |
| VpcId        | The id of the VPC where the load balancer resides    |
| Subnets      | The ids of the subnets in which to run the ECS tasks |
| FullName     | The name of the load balancer (last part of arn)     |

