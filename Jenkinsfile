@Library(['cop-pipeline-bootstrap']) _
//String pipelineVersion = (env.BRANCH_NAME && !(env.BRANCH_NAME).equals('master') && !(env.BRANCH_NAME).contains('PR-')) ? env.BRANCH_NAME : 'regression/candidate'
loadPipelines()

def config = [
    usePraDispatch: false,
    tags: [
        'Name': 'webb-frontend-ecs-test',
        'costcenter': '104420',
        'classification': 'Silver',
        'email': 'lst-gc-cdn-antibots.admin@nike.com',
        'owner': 'frank.zhao@nike.com',
        'nike-application': 'ecs-infrastructure',
        'nike-department': 'Web Eng - nike.com Cloud Capability',
        'nike-domain': 'gc-cdn-antibots',
    ],
    branchMatcher: [
      RELEASE         : ['main'],
      DEVELOPMENT     : ['^(?!main$).*$'],
    ],
    qma: [ configFile: 'quality-config.yaml' ],

    deploymentEnvironment: [
        rolesInfrastructure: [
            deployFlow: [
                ECS_INFRASTRUCTURE: ['Archive Current State', 'Deploy Infrastructure'],
            ],
            aws: [
                awsRole: "NIKE.cicd.tool",
                accountId: "439314357471",
                region: "cn-northwest-1",
            ],
            cf: [
                stackName: "webb-portal-ecs-roles-infra-test",
                templateFile: "roles.yaml",
                parameters: [
                    BmxBaseRoleArns: 'arn:aws-cn:iam::108851027208:role/brewmaster-base-gc-cdn-antibots',
                    CreateIamForEc2BackedEcsClusters: 'false',
                    HostedZoneDnsBase: 'nike.internal',
                    TeamPrefix: 'gc-cdn-antibots',
                    VpcId: 'vpc-0f9779e69a780c25e',
                ]
            ],
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
                    stackName: "<team-name>-<stack-name>",
                    templateFile: "regression/infrastructure/ecs/ecs-cluster.yaml",
                    parameters: [
                        ClusterName: '<team-name>-<cluster-name>',
                    ],
                ],
            ],
            loadBalancerInfraStructure: [
                deployFlow: [
                    ECS_INFRASTRUCTURE: ['Archive Current State', 'Deploy Infrastructure'],
                ],
                aws: [
                    awsRole: "<provided aws role>",
                    accountId: "<your accountId>",
                    region: "<specified region>",
                ],
                cf: [
                    stackName: "<team-name>-<stack-name>",
                    templateFile: "ecs-load-balancer.yaml",
                    parameters: [
                        TeamPrefix: '<team-name>',
                        AppName: '<suffix for target group and LB>',
                        SubnetIds: 'subnet-0ad62cef2459915af,subnet-07aa814bbf0c156f9,subnet-027486f998ae8c920',
                        VpcId: 'vpc-0fc24accfb15de36d',
                        SecurityGroups: ''
                    ],
                ],
            ],
        ],
    ],
]

cloudformationPipeline(config)
