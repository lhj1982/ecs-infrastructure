@Library(['cop-pipeline-bootstrap']) _
//String pipelineVersion = (env.BRANCH_NAME && !(env.BRANCH_NAME).equals('master') && !(env.BRANCH_NAME).contains('PR-')) ? env.BRANCH_NAME : 'regression/candidate'
loadPipelines()

def config = [
    usePraDispatch: false,
    tags: [
        'Name': 'ecs-infrastructure',
        'classification': 'sliver',
        'email': 'bradley.shao@nike.com',
        'owner': 'gc-cdn-antibots',
        'nike-department': 'platform engineering - gc launch',
        'nike-domain': 'gc-cdn-antibots',
        'nike-application': 'ecs-infrastructure',
        'nike-distributionlist': 'Lst-gc-cdn-antibots.admin@nike.com',
        'nike-owner': 'frank.zhao@nike.com',
        'nike-environment': 'test'
    ],
    branchMatcher: [
      RELEASE         : ['main'],
      DEVELOPMENT     : ['^(?!main$).*$'],
    ],
    qma: [ configFile: 'quality-config.yaml' ],

    deploymentEnvironment: [
        rolesInfrastructure: [
            agentLabel: 'china',
            deployFlow: [
                ECS_INFRASTRUCTURE: ['Archive Current State', 'Deploy Infrastructure'],
            ],
            aws: [
                role: "NIKE.cicd.tool",
                roleAccount: "439314357471",
                region: "cn-northwest-1",
            ],
            cf: [
                stackName: "webb-portal-ecs-roles-infra-test",
                templateFile: "roles.yaml",
                parameters: [
                    BmxBaseRoleArns: 'arn:aws-cn:iam::108851027208:role/brewmaster-base-gc-cdn-antibots',
                    TeamPrefix: 'gcantibots',
                    VpcId: 'vpc-0f9779e69a780c25e',
                ]
            ],
        ],
        clusterInfrastructure: [
            agentLabel: 'china',
            deployFlow: [
                ECS_INFRASTRUCTURE: ['Archive Current State', 'Deploy Infrastructure'],
            ],
            aws: [
                role: "NIKE.cicd.tool",
                roleAccount: "439314357471",
                region: "cn-northwest-1",
            ],
            cf: [
                stackName: "webb-portal-ecs-cluster-infra-test",
                templateFile: "ec2-cluster.yaml",
                parameters: [
                    ECSClusterName: 'webb-portal-frontend-cluster-test',
                    IamRoleInstanceProfile: 'arn:aws-cn:iam::439314357471:instance-profile/ecsInstanceRole',
                    LatestECSOptimizedAMI: '/aws/service/ecs/optimized-ami/amazon-linux-2/recommended/image_id',
                    SecurityGroupIds: 'sg-0676976675a94e49e',
                    SubnetIds: 'subnet-075e204cb71470d70,subnet-0e64f859c59876536,subnet-0f02dd76225273efa',
                    VpcId: 'vpc-0f9779e69a780c25e'
                ],   
            ],    
        ],
        ecrInfrastructure: [
            agentLabel: 'china',
            deployFlow: [
                ECS_INFRASTRUCTURE: ['Archive Current State', 'Deploy Infrastructure'],
            ],
            aws: [
                role: "NIKE.cicd.tool",
                roleAccount: "439314357471",
                region: "cn-northwest-1",
            ],
            cf: [
                stackName: "webb-portal-ecr-infra-test",
                templateFile: "repository-infrastructure.yaml",
                parameters: [
                    TeamPrefix: 'gcantibots',
                    AppName: 'webb-portal-frontend',
                    // ImageName: 'webb-portal-frontend-ecr',
                ],   
            ],    
        ],
    ],
]


cloudformationPipeline(config)
