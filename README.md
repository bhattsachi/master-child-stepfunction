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
T+02m  Master catches failure, marks Thread-1 FAILED
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
