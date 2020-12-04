# AWS CloudFormation Stacks
Generic CloudFormation stacks for personal projects

## Personal Security Groups

Example CLI query to install all Personal Security Groups

```bash
aws cloudformation create-stack --stack-name personal-security-groups \
    --template-body file://personal-security-groups.yml \
    --parameters \
    ParameterKey=SelectedVPC,ParameterValue="" \
    ParameterKey=SecurityGroupIngressCIDRHome,ParameterValue="" \
    ParameterKey=SecurityGroupIngressCIDRWork,ParameterValue=""
```

Alternatively, using a file (which is ignored from this repo on account of obscuring CIDR):

```json
[
  {
    "ParameterKey": "SelectedVPC",
    "ParameterValue": ""
  },
  {
    "ParameterKey": "SecurityGroupIngressCIDRHome",
    "ParameterValue": ""
  },
  {
    "ParameterKey": "SecurityGroupIngressCIDRWork",
    "ParameterValue": ""
  }
]
```

Thence,

```bash
aws cloudformation create-stack --stack-name personal-security-groups \
    --template-body file://personal-security-groups.yml \
    --parameters file://personal-security-groups-parameters.json
```

For updates, perform the following; rinse, and repeat:

```bash
aws cloudformation update-stack --stack-name personal-security-groups \
    --template-body file://personal-security-groups.yml \
    --parameters file://personal-security-groups-parameters.json
```

## HTTP Security Groups

Security Groups for common HTTP(S) applications.

```bash
aws cloudformation create-stack --stack-name http-security-groups \
    --template-body file://http-security-groups.yml \
    --parameters file://http-security-groups-parameters.json
```

## ECS Profiles

Manages Instance Profiles (Roles) for Common ECS Usage

```bash
aws cloudformation create-stack --stack-name ecs-profiles \
    --template-body file://ecs-profiles.yml \
    --capabilities CAPABILITY_IAM
```

## New Relic IAM Role

New Relic is capable of monitoring AWS infrastructure with basic metrics if given proper permission.
As per [these instructions](https://docs.newrelic.com/docs/integrations/amazon-integrations/get-started/connect-aws-infrastructure), an IAM Role must be created for New Relic to assume.  Such a ReadOnly role for New Relic can
be created with this template.

```bash
NEW_RELIC_ACCOUNT_ID="Enter NR Account ID Here"
aws cloudformation create-stack --stack-name iam-new-relic-role \
  --template-body file://iam-new-relic-role.yml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters \
    ParameterKey=NewRelicAccountId,ParameterValue=$NEW_RELIC_ACCOUNT_ID

```
## Additional Infrastructure Monitoring
New Relic Infrastructure Monitoring can be added to any cluster via a container that attaches to the EC2 host(s) of said cluster.
By specifying a "Target Cluster", as well as a path to the "New Relic API Key Secret", the infrastructure instrumentation
becomes possible by creating a generic stack from this project:

```bash
STACK_NAME=""
TARGET_CLUSTER=""
SSM_LICENSE_PATH=""
aws cloudformation create-stack --stack-name $STACK_NAME-monitoring \
  --template-body file://ecs-monitoring.yml \
  --capabilities CAPABILITY_IAM \
  --parameters \
  ParameterKey="NewReliceLicenseSecretPath",ParameterValue=$SSM_LICENSE_PATH \
  ParameterKey="TargetClusterName",ParameterValue=$TARGET_CLUSTER
```

NOTE: The `$STACK_NAME` parameter has no bearing on actual functionality, but simply affects the name of the monitoring stack in CloudFormation.

## Databases

Generic, shared Databases Instances are made available in this repository.&dagger;  They use AWS Secrets Manager
to create (and cycle) Master passwords with the username "root".  Retrieve the Secret from the
[AWS Console](https://console.aws.amazon.com/secretsmanager/home?region=us-east-1#/listSecrets)
and use that to log into the Instance and create individual databases with separate credentials
as needed (presumably stored in SSM).  To install a Database Instance, perform the following (e.g., for MySQL):

```bash
aws cloudformation create-stack --stack-name database-shared-mysql \
    --template-body file://database-mysql-stack.yml \
    --parameters file://database-mysql-parameters.json
```

&dagger;Note that the purpose of a shared Database Instance is for cost-savings during prototyping,
experimentation, development, etc. Projects of scale or public consumption are strongly encouraged
to use dedicated, isolated instances.

## EKS Cluster
```
aws cloudformation create-stack --stack-name eks-generic-cluster --template-body file://eks-cluster.yml --parameters file://eks-cluster-parameters.json --capabilities CAPABILITY_IAM
```

Update any Database Stacks (if necessary)...
For now, manually grab the Security Group ID automatically created by AWS.
```
aws cloudformation update-stack --stack-name database-shared-mysql \
    --template-body file://database-mysql-stack.yml \
    --parameters file://database-mysql-parameters.json
```

## CloudFormation CI Pipeline

As CloudFormation promotes managing infrastructure with code, it is possible
to version this code in an SCM (like GitHub) and, consequently, test and release
Change Sets to CloudFormation Stacks.

This template creates a CodePipeline project which pulls changes from a GitHub
repository, deploys the Stack changes for review and, if approved, creates a
Change Set which can then be applied to the "Production" Stack. The test
Stack is destroyed after approval.

Example CLI query to create this Stack (using Change Sets):

```bash
aws cloudformation create-change-set --stack-name cf-ci-pipeline \
    --template-body file://cf-ci-pipeline.yml \
    --parameters file://cf-ci-pipeline-parameters.json \
    --capabilities CAPABILITY_IAM \
    --change-set-type CREATE --change-set-name "InitialRevision"
```

*Note:* The following two examples could use either the
`change-set-name/stack-name` combination, or the `ARN`  value only

Review the requested change set:

```bash
aws cloudformation describe-change-set --change-set-name "InitialRevision" --stack-name cf-ci-pipeline
```

Execute the change set:

```bash
aws cloudformation execute-change-set --change-set-name ARN_GOES_HERE
```


For updates, perform the following; rinse, and repeat:

```bash
aws cloudformation create-change-set --stack-name cf-ci-pipeline \
    --template-body file://cf-ci-pipeline.yml \
    --parameters file://cf-ci-pipeline-parameters.json \
    --capabilities CAPABILITY_IAM \
    --change-set-type UPDATE --change-set-name "NewRevision"
```
