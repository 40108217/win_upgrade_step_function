# Chained Step Function for Windows Upgrade

This chained Step Function orchestrates Windows 2016 to 2019 upgrade by calling the direct upgrade Step Function.

## Files

- `chained-step-function-input.json` - Input parameters
- `step-function-windows-2016-to-2019-direct.json` - Direct upgrade Step Function
- `stepfunctions-execution-policy.json` - Required IAM permissions

## Architecture

```
Chained Step Function → Direct Upgrade Step Function → Windows 2019 Instance
```

## Input Parameters

```json
{
  "InstanceId": "i-06c1563b39f5924ca",
  "IamInstanceProfile": "veru_ssm",
  "SubnetId": "subnet-0f18c72cd6a67f602",
  "TargetWindowVersion": "2019",
  "InstanceType": "t2.medium",
  "AutomationAssumeRole": "arn:aws:iam::851725307706:role/veru_ssm_mgr"
}
```

## Prerequisites

1. IAM role with Step Functions execution permissions
2. Windows 2016 Datacenter instance with SSM Agent
3. VPC subnet with internet access

## Setup

1. **Create IAM Policy**:
```bash
aws iam create-policy --policy-name StepFunctionsExecutionPolicy --policy-document file://stepfunctions-execution-policy.json
```

2. **Attach Policy to Role**:
```bash
aws iam attach-role-policy --role-name StepFunctions-win_upgrade-role-92c2utboe --policy-arn arn:aws:iam::730335587648:policy/StepFunctionsExecutionPolicy
```

3. **Create Step Function**:
```bash
aws stepfunctions create-state-machine --name Windows2016To2019DirectUpgrade --definition file://step-function-windows-2016-to-2019-direct.json --role-arn arn:aws:iam::730335587648:role/StepFunctions-win_upgrade-role-92c2utboe
```

## Execution

```bash
aws stepfunctions start-execution --state-machine-arn arn:aws:states:us-east-1:730335587648:stateMachine:Windows2016To2019DirectUpgrade --input file://chained-step-function-input.json
```

## Process Flow

1. **Backup**: Creates AMI from source Windows 2016 instance
2. **Launch**: Starts temporary upgrade instance
3. **Attach Media**: Mounts Windows 2019 installation media
4. **Upgrade**: Executes PowerShell upgrade script
5. **Verify**: Confirms Windows 2019 upgrade success
6. **Create AMI**: Generates upgraded Windows 2019 AMI
7. **Launch Final**: Deploys final Windows 2019 instance
8. **Cleanup**: Terminates temporary resources

## Output

- New Windows 2019 instance
- Windows 2019 AMI
- Original Windows 2016 instance preserved

## Troubleshooting
<img width="497" height="641" alt="part_1_stepfunctions_graph" src="https://github.com/user-attachments/assets/234a18ab-6e82-4ce4-b3c8-930ebc34b04f" />



- **Access Denied**: Check IAM permissions
- **Instance Not Ready**: Verify SSM Agent status
- **Upgrade Failed**: Check PowerShell execution logs
- **Media Not Found**: Ensure Windows 2019 media available in region
