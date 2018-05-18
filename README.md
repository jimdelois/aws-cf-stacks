# AWS CloudFormation Stacks
Generic CloudFormation stacks for personal projects

## Personal Security Groups

Example CLI query to install all Personal Security Groups

```bash
aws cloudformation create-stack  --stack-name personal-security-groups \
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
aws cloudformation create-change-set --stack-name personal-security-groups \
    --template-body file://personal-security-groups.yml \
    --parameters file://personal-security-groups-parameters.json \
    --change-set-type CREATE --change-set-name "InitialRevision"
```

Review the requested change set (or use the ARN):

```bash
aws cloudformation describe-change-set --change-set-name "InitialRevision" --stack-name personal-security-groups
```

Execute the change set:

```bash
aws cloudformation execute-change-set --change-set-name ARN_GOES_HERE
```

For updates, perform the following, rinse, and repeat:

```bash
aws cloudformation create-change-set --stack-name personal-security-groups \
    --template-body file://personal-security-groups.yml \
    --parameters file://personal-security-groups-parameters.json \
    --change-set-type UPDATE --change-set-name "NewRevision"
```

## HTTP Security Groups

Security Groups for common HTTP/S applications.

```base
aws cloudformation create-change-set --stack-name http-security-groups \
    --template-body file://http-security-groups.yml \
    --parameters file://http-security-groups-parameters.json \
    --change-set-type CREATE --change-set-name "InitialRevision"
```

## ECS Profiles

Manages Instance Profiles (Roles) for Common ECS Usage

```base
aws cloudformation create-change-set --stack-name ecs-profiles \
    --template-body file://ecs-profiles.yml \
    --capabilities CAPABILITY_IAM \
    --change-set-type CREATE --change-set-name "InitialRevision"
```
