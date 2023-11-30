## CloudFormation template for ECS service infrastructure

When deploying your application to ecs you will need to have an Repository for your containers. 
This CloudFormation template stands up an ECR Repository for you.

### Template parameters

| Parameter Name          | Description                                                                                                                          | Required | Default | Example    |
|-------------------------|--------------------------------------------------------------------------------------------------------------------------------------|----------|---------|------------|
| TeamPrefix              | A unique prefix for all ECS services used by your team.                                                                              | Y        |         | myTeam     |
| AppName                 | The name of your app                                                                                                                 | Y        |         | myApp      |
| ImageName               | The AppName param used as an input when building the stack                                                                           | N        |         | dev-branch |
| RepoLifecyclePolicyText | A JSON reflecting the Lifecycle policy for the app's ECS registry                                                                    | N        |         | -TODO-     |


### Template outputs

The following attributes are available to consumers of the template:

| Output                       | Description                                                                |
|------------------------------|----------------------------------------------------------------------------|
| EcrRepositoryName            | The name of the ECR repository to be used by Docker when tagging / pushing |
| EcrRepositoryArn             | The ARN of the ECR repository to be used by Docker when tagging / pushing  |
| ImageName                    | The AppName param that was used as an input when building the stack        |
| AppName                      | The name of the application                                                |
