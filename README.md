# ECS Pipeline Infrastructure
The following guide contains all of the instructions to set up and create the infrastructure
required to deploy ECS applications via Cloud Acceleration BMX Pipelines. This guide assumes no prior
knowledge of Continuous Integration (CI) best practices or, AWS ECS architectures. 

By the end of this guide you will have:
* AWS IAM roles and policies to allow seamless CI.
* An ECS Cluster to deploy applications.
* load balancer to serve containerized applications
* CloudFormation stacks to allow for repeatable infrastructure builds.
* A Jenkins build job to automate all infrastructure tasks.

<TL/DR>
If you're pretty comfortable with ECS Architecture, Pipelines CloudFormation builds and templates, then the accompanying 
Jenkinsfile can be populated with requisite values an run in one go. Make sure the `teamName` parameter is consistent
across configurations or you may be unable to deploy using the ECS pipeline. You will need to add your own ECR configurations
but the template and doc are provided. 

# Prerequisites:
Before following the steps in this guide, please ensure you have the following capabilities:
* A GitHub organization provisiond by the BAAT Team (reach out to [#inf-buildtools](https://nikedigital.slack.com/archives/C0JSX3QSZ))
* A working AWS account with administrative privileges(AWS accounts now can be provisioned via [Waffle Iron][WaffleIron])
* An established VPC within this account set up in accordance with Nike Best Practices.
* An operational BMX Jenkins instance with an Active Directory Service User. Follow the steps at: [BMX guide][BMXGuide]
* Credentials established to fetch from your GitHub organization in Jenkins. Docmented [Here](https://bmx.auto.nikecloud.com/configuration/credentials/#using-github-app)

# Step One: Setup Repository
## 1. Create a repository on your local development environment
```shell script
$ mkdir ecs_infrastructure
$ cd ecs_infrastructure
$ git init
$ git remote add -f origin https://github.com/nike-cop-pipeline/cop-pipeline-cookbook.git
$ git config core.sparseCheckout true
$ echo ecs-infrastructure >> .git/info/sparse-checkout
$ git checkout main
$ git pull origin main
```
This will only pull this directory from git, leaving the other directories above this out.

## 2. Create a repository in your oganization
In this repository, blow away git references:
`$ rm -rf .git`
In GitHub, create a blank repo.
Follow the instructions provided to push to main.

# Step Two: Create Roles
This step allows us to provide access roles via IAM to allow our BMX to deploy infrastructure. This is the "two-way"
street mentioned when you added roles to your BMX. It also creates a Task Executor role. This is required to manage your 
ECS cluster. 

## 1. Populate The Jenkinsfile
Update the `teamsInfrastructure` stanza in your deploy configuration with values described by the [teams document][teams]
document.
```groovy
teamsInfrastructure: [
    deployFlow: [
        ECS_INFRASTRUCTURE: ['Archive Current State', 'Deploy Infrastructure'],
    ],
    aws: [
        awsRole: "<provided aws role>",
        accountId: "<your accountId>",
        region: "<specified region>",
    ],
    cf: [
        stackName: "",
        templateFile: "team.yaml",
        parameters: [
            bmxBaseRoleArns: [
                '<run /bmx info ... in #auto-bot>',
            ],
            createIamForEc2BackedEcsClusters: 'false',
            hostedZoneDnsBase: 'nike.internal',
            teamPrefix: '',
            vpcId: '',
        ]
    ]
] 
```
The section of interest here is the `cf` entry. These are the parameters that are used to create a CloudFormation stack. Please see 
[cloud formation docs][cfdocs]
for more details.

Other fields to note:
* `teamName`: Most of your resources will either be discovered or created with the teamName as a prefix. This is used
throughout the infrastructure and ecs builds.
* `stackName`: This will be the name of the CloudFormation stack (resource set) generated.
* `vpcId` : this should be the vpc created in your account and should have at least 2 sub-zones mapped to availability regions.

### Trust is a two way street
prior to running your build, you will need to know which role / roles need to be assumed by your BMX. In our case, we will be 
performing Cloud Formation updates so, it stands to reason that we will need to assume a role which can perform cloud formation tasks.
The `awsRole` in the deployConfig should be enabled to perform these tasks. Also, ensure that this role has been added to your BMX
(using the autobot command) For detailed instructions on setting your role, visit our
[iam roles documentation](https://bmx.auto.nikecloud.com/configuration/iam-roles/)


## 2. Run the roles.yaml stack build
Create a [Multi-branch Pipeline][mbp] for your repo.
Once it creates, it will scan your repository for branches that have a Jenkinsfile.

Builds will automatically happen but don't worry. This Jenkinsfile is set up to be run manually. These build(s) will essentially
be no-ops that won't build or do anything except "prime" your build environment with the necessary parameters.

### Set Manual Build Parameters and Deploy
* At the branch level of your Jenkins Job, select `Build With Parameters`
* Select `ECS_INFRASTRUCTURE` as the Deploy Flow
* Select `rolesInfrastructure` as the build environment

Click "Build Now"

Once your build succeeds, Feel free to log in to the AWS Console and check the CloudFormation stack that was created. Of note will be
the Outputs tab. Notice the following roles have been generated:

| Key                  | Value                                                  | Description                               |
|----------------------|--------------------------------------------------------|-------------------------------------------|
| BrewmasterEcsRoleArn | arn:aws:iam::1234567890:role/my-team-BrewmasterEcsRole | Arn of the ECS role assumed by brewmaster |
|                      |                                                        | to perform deployment-related operation   |
|                      |                                                        | for ECS services and related resources.   |
| ExecutionRoleArn     | arn:aws:iam::1234567890:role/my-team-TaskExecutionRole | Arn of the service execution role         |

Check the [teams][teams] docs for more details on outputs.
 
# Step Three: Create and Deploy Cluster
In this step, we will create our core ECS infrastructure component, the ECS cluster. An ECS Cluster is a
logical space which encompasses containerized applications please familiarize yourself with the
[AWS Docs][clusterdocs] for further reference.

There are 2 types of cluster instances: EC2 backed and Fargate Clusters. Fargate is a "managed" computational resource for ECS. It basically provides
logical space for your host running containers. This guide presumes setting up ECS via Fargate.
We will be populating the Jenkinsfile we touched earlier with our first resource and build that.

When using Fargate, the only parameter required to spin up a cluster is a cluster name (The instance size defaults to `t2.micro` however)
Please familiarize yourself with the additional cluster parameters described [here][clusters].

Ensure your Jenkinsfile captures the following `cf` parameters:
```groovy
clusterInfrastructure: [
    deployFlow: [
            ECS_INFRASTRUCTURE: ['Archive Current State', 'Deploy Infrastructure'],
    ],
    aws: [
            awsRole: "<provided aws role>",
            accountId: "<your accountId>",
            region: "<specified region>",
    ],
    cf: [
        stackName: "<your stack name>",
        templateFile: "regression/infrastructure/ecs/ecs-cluster.yaml",
        parameters: [
            ClusterName: '<team-name>-<cluster-name>',
        ],
    ],
]
```
### Set Manual Build Parameters and Deploy
* At the branch level of your Jenkins Job, select `Build With Parameters`
* Select `ECS_INFRASTRUCTURE` as the Deploy Flow
* Select `clusterInfrastructure` as the build environment

Click "Build Now"

Your build will create a solitary ECS Cluster ready to be populated with services and tasks.

# Step Four: Create the Load Balancer

The load balancer templates will install an Application Load Balancer(ALB) to your account. While not specifically required,
ALB's will allow a great deal of flexibility in your application performance. If you plan on using Referee, or Automatic 
Scaling, your application will require and ALB

More about ALB's with ECS can be found [here][albs].

Just like our Teams and Cluster Jenkinsfile configurations, we will be populating the following for our Load Balancer configuration:
```groovy
loadBalancerInfrastructure: [
    deployFlow: [
            ECS_INFRASTRUCTURE: ['Archive Current State', 'Deploy Infrastructure'],
    ],
    aws: [
            awsRole: "<provided aws role>",
            accountId: "<your accountId>",
            region: "<specified region>",
    ],
    cf: [
        stackName: "<team-name>-load-balancer",
        templateFile: "load-balancer.yaml",
        parameters: [
            TeamPrefix: '<team-name>',
            AppName: 'test',
            SubnetIds: '<valid subnets from your VPC',
            VpcId: '<working VPC>',
            SecurityGroups: ''
        ],
    ],
]
```
The `AppName` is appeneded to your listenerARN & your LoadBalancerARN. The team name is prepended. So if you had a a team name 
of `my-team` and and an AppName of `my-app` your ARNS would be: `my-team-my-app`. Check the load-balancer.yaml and [docs][alb] file for details.

Run this the same way you ran the cluster deploy flow by selecting the "refresh parameters" selection and rebuilding (See above)

# Step 5: Create Your ECR
If you're ready to deploy images, you will need to create ECR's for them to reside. Following the same pattern as above, create 
a `repositoryInfrastructure` deployFlow just like the others and populate it with the parameters described by [the docs][ecr].

A few notes on some params:
* TeamPrefix - as mentioned earlier, this will lock in a pattern for your ECS assets. The ECS Pipeline will look for images with this pattern
* PagerDutyIntegrationKey - This is optional but creates an SNS topic. But nobody really knows why this is here in the first place.
* Subnets - As a rule, place your application within private subnets within your VPC. Please see the Additional Considerations
  Section
* VPC - The subnets selected will need to belong to the VPC you provided

You can add ECR Blocks to this Jenkins file or keep them separate. You want one ECR per image so you may want to maintain ECR creation
as it's own Pipeline. The ECS Pipeline _does not_ create ECRs for your images. They do need to be stood up prior to running.
The ECS Pipeline _does_ push images to your ECRs if provided.

# Working Example
The [waffle-ci-test.jenkinsfile](./waffle-ci-test.jenkinsfile) is a working example created in the `cicd-tools-test` waffle 
account (Where we run all of our test and regression pipelines). 

# Networking and Discoverability
When you stand up a Waffle Iron account, it's VPC is very limited in terms of Network connectivity. Visit the [Waffle Iron FAQs][WI]
for instructions on enabling your WI account for VPC connectivity to the Nike Internal network and beyond.


# Conclusion
You now have a portable, buildable, repeatable ECS Stack ready to serve your containers. You are now ready to utilize ECS Pipelines
to deploy your containers to serve your apps.

* We used AWS Cloud Formation templates coupled with Jenkins to build resources from a single source of truth.
* Our CloudFormation templates are driven by configuration with Jenkins as one option among many to populate.
* Our CloudFormation stacks now have a historical record of when they were built, with what and how. 
* We have learned how Jenkins integrates with AWS and how Pipelines help serve and mitigate Automation.

[WaffleIron]: https://confluence.nike.com/display/NDAM/Waffle+Iron
[BMXGuide]: https://bmx.auto.nikecloud.com/
[teams]: https://github.com/nike-cop-pipeline/cop-pipeline-cookbook/blob/main/ecs-infrastructure/team.md
[cfdocs]: https://github.com/nike-cop-pipeline/cop-pipeline-step/blob/main/vars/cloudformationStack.md
[mbp]: https://pipelines.auto.nikecloud.com/quickstart/project-setup/#setup-multibranch-pipeline
[clusterdocs]: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/clusters.html
[clusters]: https://github.com/nike-cop-pipeline/cop-pipeline-cookbook/blob/main/ecs-infrastructure/cluster.md
[albs]: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-load-balancing.html
[alb]: https://github.com/nike-cop-pipeline/cop-pipeline-cookbook/blob/main/ecs-infrastructure/load-balancer.md
[ecr]: https://github.com/nike-cop-pipeline/cop-pipeline-cookbook/blob/main/ecs-infrastructure/repository-infrastructure.md
[WI]: https://confluence.nike.com/pages/viewpage.action?spaceKey=NDAM&title=Waffle+Iron+-+FAQ#WaffleIronFAQ--HowcanIprovisionaVPCwithNikeinternalnetworkconnectivityinaWaffleIronaccount?-
