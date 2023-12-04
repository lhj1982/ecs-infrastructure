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
            clusterInfrastructure: [
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
                    templateFile: "cluster.yaml",
                    parameters: [
                        ClusterName: 'webb-portal-frontend-cluster',
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
