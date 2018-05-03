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
aws cloudformation create-stack  --stack-name personal-security-groups \
    --template-body file://personal-security-groups.yml \
    --parameters file://personal-security-groups-parameters.json
```
