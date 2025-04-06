# Meta-Driven-Checkpoint-Orchestration-in-Azure-Data-Factory

Meta-Driven Checkpoint Orchestration in Azure Data Factory

🚀 Overview

This repository introduces a production-grade pattern for implementing activity-level checkpoint orchestration in Azure Data Factory (ADF). Unlike traditional pipeline-level retries, this approach provides granular, metadata-controlled execution logic that skips already-successful activities—improving performance, cost-efficiency, and resilience.

🎯 Use Case

Many ADF pipelines reprocess data unnecessarily or require manual reruns on failure. With this pattern, we can:

Skip previously successful activities

Resume pipelines from the point of failure

Drive orchestration using metadata tables

Avoid duplicating logic or hardcoded branching

🧠 Core Concepts

1. Checkpoint Metadata Table (Dynamic Table)

A table that tracks executed activities per pipeline.

CREATE TABLE [dbo].[ADFCheckpoint] (
  PipelineName     VARCHAR(100),
  ActivityName     VARCHAR(100),
  Status           VARCHAR(50),
  LastExecutionTime DATETIME
);

2. Pipeline Sequence Metadata Table (Static Table)

Defines the sequence and logic of activity execution.

CREATE TABLE [dbo].[PipelineExecutionPlan] (
  PipelineName     VARCHAR(100),
  ActivityName     VARCHAR(100),
  SequenceOrder    INT
);

3. Lookup Activity

Retrieves the last executed activity for a given pipeline:

"sqlReaderQuery": "SELECT TOP 1 ActivityName FROM ADFCheckpoint WHERE PipelineName = 'myPipeline' AND Status = 'Succeeded' ORDER BY LastExecutionTime DESC"

4. If Condition Activities

Each If Condition checks whether an activity already ran:

"expression": "@not(equals(first(activity('Lookup_Checkpoint').output.value).ActivityName, 'ActivityA'))"

If the activity hasn't run yet, it proceeds to execute it. Otherwise, it skips to the next.

5. Logging Activity Status

A Stored Procedure activity logs each step after execution.

EXEC usp_logactivity @PipelineName = 'myPipeline', @ActivityName = 'ActivityA', @Status = 'Succeeded';

🛠️ How It Works

Start pipeline: Lookup last successful activity from dynamic checkpoint table.

Evaluate with If Condition: Compare each upcoming activity name with the checkpoint.

Execute activities conditionally: If the activity is new, run it and log it.

Update checkpoint table: After success, write status to the dynamic table.

Repeat: Move to the next activity using the same logic.

🧩 Benefits

✅ Fine-grained control over task execution✅ Efficient handling of failures without reruns✅ Centralized execution logic via metadata✅ No redundant processing or logic duplication✅ Plug-and-play orchestration across pipelines

📂 Repository Structure

meta-checkpoint-orchestration/
├── pipelines/
│   └── example-pipeline.json           # Sample ADF pipeline using checkpoint logic
├── sql/
│   ├── create_checkpoint_table.sql     # Dynamic log table
│   └── create_execution_plan_table.sql # Static orchestration plan
├── docs/
│   └── architecture.png                # Visual diagram
│   └── process-flow.md                 # Step-by-step flow
├── README.md                           # This file

📸 Architecture Diagram

(Include your PowerPoint image or a simplified PNG of the control flow)

📈 Future Enhancements

Add email alerts or failure notifications

Integrate with Azure DevOps/GitHub CI/CD

Extend to data-driven (parameterized) pipelines

Use Logic Apps for external retry control

🧑‍💼 About the Author

This approach was discovered by me
