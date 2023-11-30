## CloudFormation template for ECS pipeline Cluster

When deploying your application to ECS, you may want to define an ECS cluster. 
This CloudFormation template stands up an ECS cluster for you.

### Template parameters

| Parameter Name    | Description                                       | Required | Default    | Example            |
|-------------------|---------------------------------------------------|----------|------------|--------------------|
| LaunchType        | [ EC2 | Fargate ]                                 | N        | Fargate    |                    |
| AmiId             | (EC2 only) ID of the AMI to use for the ECS Container Instances | N |     | ami-6532ad         |
| InstanceType      | (EC2 only) Instance type to use for cluster       | N        | t2.micro   | l1.medium          |
| ClusterSize       | (EC2 only) Maximum cluster size                   | Y        | 2          |                    |
| Subnets           | Comma-separated list of Subnet IDs in which to place the EC2 instances in the ECS cluster | Y |  | ['subnet-1234', 'subnet-2356'] |
| InstanceSecurityGroups | Comma-separated list of Security Group IDs to attach to EC2 instances in the ECS cluster | Y | | ['sg-1234', 'sg-2356' ] |
| ClusterName       | Name of the ECS Cluster                           | Y        |            | myApp-cluster      |
| KeyName           | The name of the keypair to associate with the ECS Container Instances (EC2 Hosts) | Y | |      |
| InstanceProfileArn | ARN of the instance profile to attach to the EC2 instances in the cluster | Y | |             |

### Template outputs

The following attributes are available to consumers of the template:

| Output              | Description                                     |
|---------------------|-------------------------------------------------|
| ClusterName         | The name of the ECS Cluster                     |
