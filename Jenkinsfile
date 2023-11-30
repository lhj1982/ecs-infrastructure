@Library(['cop-pipeline-bootstrap']) _
//String pipelineVersion = (env.BRANCH_NAME && !(env.BRANCH_NAME).equals('master') && !(env.BRANCH_NAME).contains('PR-')) ? env.BRANCH_NAME : 'regression/candidate'
loadPipelines()

def config = [
    usePraDispatch: false,
    tags: [
        'Name': 'webb-frontend-ecs-test',
        'costcenter': '',
        'classification': '',
        'email': '',
        'owner': '',
        'nike-application': '',
        'nike-department': '',
        'nike-domain': '',
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
                awsRole: "<provided aws role>",
                accountId: "<your accountId>",
                region: "<specified region>",
            ],
            cf: [
                stackName: "",
                templateFile: "roles.yaml",
                parameters: [
                    BmxBaseRoleArns: '<run /bmx info ... in #auto-bot>,<additional roles separated by comma if needed>',
                    CreateIamForEc2BackedEcsClusters: 'false',
                    HostedZoneDnsBase: 'nike.internal',
                    TeamPrefix: '<used throughout the infrastructure>',
                    VpcId: '',
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
