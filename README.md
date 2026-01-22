# master-child-stepfunction

# input when starting the Step Function.
{
  "threadId": "T1",
  "glueJobs": [
    "glue-job-a",
    "glue-job-b",
    "glue-job-c"
  ]
}

# Execution Input (Start Master Step Function)
{
  "threads": [
    { "threadId": "T1", "glueJobs": ["job1","job2","job3"] },
    { "threadId": "T2", "glueJobs": ["job4","job5","job6"] },
    { "threadId": "T3", "glueJobs": ["job7","job8","job9"] },
    { "threadId": "T4", "glueJobs": ["job10","job11","job12"] },
    { "threadId": "T5", "glueJobs": ["job13","job14","job15"] }
  ]
}

# Execution Timeline 
T+00s  All 5 children start
T+02m  Child-1 Glue job fails
T+02m  Child-1 execution = FAILED
T+02m  Master catches failure, marks child-1 FAILED
T+15m  Other children finish
T+15m  Parallel completes
T+15m  Master sends SNS notification
T+15m  Master execution = SUCCESS


# Final Answer (Straight)

# When Glue fails in Child-1:

✔ Child Step Function execution = FAILED
✔ Other child threads keep executing
✔ Master waits for all children
✔ Master execution = SUCCESS
✔ Master sends notification identifying failed child

This is the cleanest, AWS-recommended orchestration pattern.


repo-root/
│
├─ cloudformation/
│   └─ pipeline.yaml          # CFN template to create CodePipeline
│
├─ SAM-template.yaml          # Step Functions + IAM + SNS
│
└─ buildspec.yml              # CodeBuild deploy logic


 1 Master step function
 Master step function will execute 5 child step functions in parallel
 All 5 child step functions start together
 each child step function will invoke 4-5 glue jobs (dont create gluejob , just put a placeholder for now)
 child step function triggers cloudwatch alarm incase of any failure
 Master step function catches failure
 child step functions should finish successfully if there is no error
 Parallel completes
 
 # Deploy with default job counts
sam deploy

# Customize jobs per child
sam deploy --parameter-overrides \
  Child1GlueJobs="etl-job-1,etl-job-2" \
  Child2GlueJobs="transform-a,transform-b,transform-c,transform-d,transform-e" \
  Child3GlueJobs="load-job-1" \
  Child4GlueJobs="process-x,process-y,process-z" \
  Child5GlueJobs="final-job"
  ### ===================================================================

  # AWS CLI Commands to Deploy SAM Template using CloudFormation

## Prerequisites
- AWS CLI installed and configured
- Appropriate IAM permissions for CloudFormation, Step Functions, IAM, CloudWatch, and Glue

---

## Option 1: Using `aws cloudformation package` and `deploy` (Recommended for SAM templates)

### Step 1: Package the template
This transforms SAM-specific resources and uploads artifacts to S3.

```bash

# quick deploy 
  aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name master-step-function-stack \
  --capabilities CAPABILITY_NAMED_IAM

### Step 2: Deploy the packaged template

```bash
aws cloudformation deploy \
  --template-file packaged-template.yaml \
  --stack-name master-step-function-stack \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    Environment=dev \
    AlarmSNSTopicArn="" \
    Child1GlueJobs="glue-job-1-a,glue-job-1-b,glue-job-1-c" \
    Child2GlueJobs="glue-job-2-a,glue-job-2-b,glue-job-2-c,glue-job-2-d" \
    Child3GlueJobs="glue-job-3-a,glue-job-3-b,glue-job-3-c,glue-job-3-d,glue-job-3-e" \
    Child4GlueJobs="glue-job-4-a,glue-job-4-b" \
    Child5GlueJobs="glue-job-5-a,glue-job-5-b,glue-job-5-c,glue-job-5-d,glue-job-5-e,glue-job-5-f"
```

---

## Option 2: Using `aws cloudformation create-stack` (Direct deployment)

### For new stack creation:

```bash
aws cloudformation create-stack \
  --stack-name master-step-function-stack \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters \
    ParameterKey=Environment,ParameterValue=dev \
    ParameterKey=AlarmSNSTopicArn,ParameterValue="" \
    ParameterKey=Child1GlueJobs,ParameterValue="glue-job-1-a,glue-job-1-b,glue-job-1-c" \
    ParameterKey=Child2GlueJobs,ParameterValue="glue-job-2-a,glue-job-2-b,glue-job-2-c,glue-job-2-d" \
    ParameterKey=Child3GlueJobs,ParameterValue="glue-job-3-a,glue-job-3-b,glue-job-3-c,glue-job-3-d,glue-job-3-e" \
    ParameterKey=Child4GlueJobs,ParameterValue="glue-job-4-a,glue-job-4-b" \
    ParameterKey=Child5GlueJobs,ParameterValue="glue-job-5-a,glue-job-5-b,glue-job-5-c,glue-job-5-d,glue-job-5-e,glue-job-5-f"
```

### For updating an existing stack:

```bash
aws cloudformation update-stack \
  --stack-name master-step-function-stack \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters \
    ParameterKey=Environment,ParameterValue=dev \
    ParameterKey=AlarmSNSTopicArn,ParameterValue="" \
    ParameterKey=Child1GlueJobs,ParameterValue="glue-job-1-a,glue-job-1-b,glue-job-1-c" \
    ParameterKey=Child2GlueJobs,ParameterValue="glue-job-2-a,glue-job-2-b,glue-job-2-c,glue-job-2-d" \
    ParameterKey=Child3GlueJobs,ParameterValue="glue-job-3-a,glue-job-3-b,glue-job-3-c,glue-job-3-d,glue-job-3-e" \
    ParameterKey=Child4GlueJobs,ParameterValue="glue-job-4-a,glue-job-4-b" \
    ParameterKey=Child5GlueJobs,ParameterValue="glue-job-5-a,glue-job-5-b,glue-job-5-c,glue-job-5-d,glue-job-5-e,glue-job-5-f"
```

---

## Option 3: Create Change Set (For reviewing changes before deployment)

### Step 1: Create the change set

```bash
aws cloudformation create-change-set \
  --stack-name master-step-function-stack \
  --change-set-name my-change-set \
  --template-body sam-template.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --change-set-type CREATE \
  --parameters \
    ParameterKey=Environment,ParameterValue=dev \
    ParameterKey=AlarmSNSTopicArn,ParameterValue="" \
    ParameterKey=Child1GlueJobs,ParameterValue="glue-job-1-a,glue-job-1-b,glue-job-1-c" \
    ParameterKey=Child2GlueJobs,ParameterValue="glue-job-2-a,glue-job-2-b,glue-job-2-c,glue-job-2-d" \
    ParameterKey=Child3GlueJobs,ParameterValue="glue-job-3-a,glue-job-3-b,glue-job-3-c,glue-job-3-d,glue-job-3-e" \
    ParameterKey=Child4GlueJobs,ParameterValue="glue-job-4-a,glue-job-4-b" \
    ParameterKey=Child5GlueJobs,ParameterValue="glue-job-5-a,glue-job-5-b,glue-job-5-c,glue-job-5-d,glue-job-5-e,glue-job-5-f"
```

### Step 2: Review the change set

```bash
aws cloudformation describe-change-set \
  --stack-name master-step-function-stack \
  --change-set-name my-change-set
```

### Step 3: Execute the change set

```bash
aws cloudformation execute-change-set \
  --stack-name master-step-function-stack \
  --change-set-name my-change-set
```

---

## Useful Monitoring Commands

### Check stack status:
```bash
aws cloudformation describe-stacks \
  --stack-name master-step-function-stack \
  --query 'Stacks[0].StackStatus'
```

### Watch stack events (useful during deployment):
```bash
aws cloudformation describe-stack-events \
  --stack-name master-step-function-stack \
  --query 'StackEvents[*].[Timestamp,ResourceStatus,ResourceType,LogicalResourceId,ResourceStatusReason]' \
  --output table
```

### Wait for stack creation to complete:
```bash
aws cloudformation wait stack-create-complete \
  --stack-name master-step-function-stack
```

### Wait for stack update to complete:
```bash
aws cloudformation wait stack-update-complete \
  --stack-name master-step-function-stack
```

### Get stack outputs:
```bash
aws cloudformation describe-stacks \
  --stack-name master-step-function-stack \
  --query 'Stacks[0].Outputs'
```

---

## Delete Stack

```bash
aws cloudformation delete-stack \
  --stack-name master-step-function-stack
```

### Wait for deletion to complete:
```bash
aws cloudformation wait stack-delete-complete \
  --stack-name master-step-function-stack
```

---

## Notes

1. **CAPABILITY_NAMED_IAM**: Required because the template creates IAM roles with custom names.

2. **S3 Bucket**: For Option 1, you need an S3 bucket to store packaged artifacts. Create one if needed:
   ```bash
   aws s3 mb s3://my-cfn-artifacts-bucket --region us-east-1
   ```

3. **Region**: Add `--region <region>` to commands if not using default region.

4. **Profile**: Add `--profile <profile-name>` if using named AWS profiles.

5. **SAM Transform**: The template uses `Transform: AWS::Serverless-2016-10-31`, which requires packaging (Option 1) or CloudFormation's macro support.
# ===========================================================

Use ONE Step Function definition
Start MULTIPLE executions at the same time
Each execution receives different Glue job names as input
Each execution runs only the Glue jobs passed to it

✅ Yes, you can execute multiple instances of the same Step Function at the same time
✅ Each execution can receive different Glue job names
✅ All executions run in parallel
✅ Glue jobs are started dynamically via input parameters

# pure cloud formation 
 aws cloudformation deploy \
  --template-file master-mutlple-glue-jobs-template.yaml \
  --stack-name master-step-glue-executor-stack \
  --capabilities CAPABILITY_NAMED_IAM

# quick deploy SAM
aws cloudformation package \
  --template-file sam-master-template.yaml \
  --s3-bucket hoth-data-application-artifacts-615299756109-us-east-2 \
  --s3-prefix sam-artifacts \
  --output-template-file packaged-template.yaml
```

aws cloudformation deploy \
  --template-file packaged-template.yaml \
  --stack-name master-step-function-multiple-gluejobs-stack \
  --capabilities CAPABILITY_NAMED_IAM
```
# ============================
SFN_ARN=$(aws cloudformation describe-stacks \
  --stack-name glue-executor-stack \
  --query 'Stacks[0].Outputs[?OutputKey==`StateMachineArn`].OutputValue' \
  --output text)

# Execute A → job1, job2, job3
aws stepfunctions start-execution \
  --state-machine-arn $SFN_ARN \
  --name "execution-a-$(date +%s)" \
  --input '{"glueJobs":"job1,job2,job3"}' &

# Execute B → job4, job5, job6
aws stepfunctions start-execution \
  --state-machine-arn $SFN_ARN \
  --name "execution-b-$(date +%s)" \
  --input '{"glueJobs":"job4,job5,job6"}' &

# Execute C → job7, job8
aws stepfunctions start-execution \
  --state-machine-arn $SFN_ARN \
  --name "execution-c-$(date +%s)" \
  --input '{"glueJobs":"job7,job8"}' &

# Wait for all background jobs
wait
Summary
ComponentCountStep Function1IAM Role1CloudWatch Alarm1Total Resources3
Each execution runs independently and processes only the Glue jobs passed to it.TemplateYAML DownloadClaude is AI and can make mistakes. Please double-check responses.
 