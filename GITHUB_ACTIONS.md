# Github Action
Github action triggers with an Event. An Event could be a Pull Request, Push, Issue (created or Closed), etc...
Events are tied to Workflow and Workflow has a following typical structure:

**Workflow**
----/Job1    ---> Runs on a Runner1 (or A run is called as an Agent in Azure DevOps)
-------/Step1: Action
-------/Step2: Shell Command
-------/Step3: Action
----/Job2    ---> Runs on a Runner2
-------/Step1: Shell Command
-------/Step2
-------/Step3
-------/Step4

**Notes**
- Steps under a Job always runs in sequence from top to bottom.
- Jobs consumes independent Runners. Hence, they can be configured to run in Parallel or in Sequence. (Default behaviour is Parallel Run)
- A workflow must be defined within a repository. There is no option to define at the organization level.

## Runners
Runners come in 2 flavours (Github-hosted runners or Self-hosted runners).
- **Github-hosted runners** come in 3 types ubuntu, MacOS and Windows. While desiging the workflow, you can define in which type of runners, the Job will run.
- **Self-hosted runners** are setup by you so you can define which OS you want to have on it.

## Workflow Syntax



