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
