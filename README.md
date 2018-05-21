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
  },
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

```base
aws cloudformation create-stack --stack-name http-security-groups \
    --template-body file://http-security-groups.yml \
    --parameters file://http-security-groups-parameters.json
```

## ECS Profiles

Manages Instance Profiles (Roles) for Common ECS Usage

```base
aws cloudformation create-stack --stack-name ecs-profiles \
    --template-body file://ecs-profiles.yml \
    --capabilities CAPABILITY_IAM
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
