## Background

We want to have a Release Pipeline for our entire infrastructure.

1. Dependencies should be supported (Native to Azure DevOps Release / Pipeline in yaml).
2. The ability to trigger Azure DevOps Pipelines / Releases (Using the Python script from DevOps)
3. The ability to gate steps (Native approval support in both Azure DevOps Release / Pipeline in yaml).

## How does the script work?

Basically the script reads arguments that are supplied as arguments to the script OR read from environment variables.

It's capable of triggering both Azure DevOps Pipelines **AND** Azure DevOps Releases.

After triggering the job, the will monitor the job it triggered and wait for it to finish.
When the job finishes, the script will exit with the same result as the triggered job (failure on job failure and success on success).

## Creating a Super PAT

When using the Python script, you'll need a PAT if we want to be able to trigger Pipelines and Releases.

You CAN create a personal access token (PAT) for all your organizations at once (this is more convenient but less secure) or you can have multiple PATs for each environment / organization because each time you run the script you'll need to provide a PAT that can trigger the Pipeline / Release...

![](Pasted%20image%2020231004185814.png)

> ⚠️ NOTE: You can always use the $(System.AccessToken) in Azure DevOps to allow the build pipeline agent to be the one triggering the jobs, but you'll have to give it access to the appropriate environments (which haven't been tested yet).
> 
>If you don't you'll get the error message `azure.devops.exceptions.AzureDevOpsServiceError: TF401019: The Git repository with name or identifier <the-identifier-guid> does not exist or you do not have permissions for the operation you are attempting.`
![](Pasted%20image%2020231005121547.png)
## Setting up a Release / Pipeline

To use the script of the flow manager, you'll need to add the `devops-flow-manager` repository as an artifact.

To add the repo in a yaml pipeline add the following:

```yaml
# Add this at the begining of the pipeline (just above the tasks) to import the devops-flow-manager repo.
resources:
  repositories:
    - repository: devops-flow-manager  # The alias of the repo in this pipeline
      type: git                        # A git repository
      name: DevOps/templates           # project/repository
      ref: refs/heads/master           # What branch / tag to get.
```

In a Release pipeline, add a new "Azure Repos Git" artifact .
- Project                        = "DevOps"
- Source (repository)   = "devops-flow-manager"
- Default branch           = main

Don't touch anything else and click on "Add".

This will allow you to use the Task Group "Trigger Azure DevOps Release".

![](Pasted%20image%2020231005094157.png)
![](Pasted%20image%2020231005094212.png)

You can always use the scripts without the Task Group if preferred or needed.

## **Usage Options**

The Python script can be utilized in multiple ways to achieve the GitOps goal.

### Using Azure Release Pipeline

The point here is that you have a Release Pipelines.
Each "Stage" (Like "Scan Pre Prod") is a trigger to a release pipeline, and dependencies are managed using the Stage dependencies provided to stages in the release pipeline.

### Using Azure Release Pipeline Stages

The idea is the same as "Using Azure Release Pipeline" but with steps within stages.
So for example a Stage can be "Release Development" and within the stage the are steps, each of them triggers a pipeline like so:

![Image|1000](Pasted%20image%2020231004182619.png)

Here, you can have multiple dependencies easily but, you can have dependencies between steps and have multiple stages.

### Using Azure Pipeline (Yaml)

Since the trigger script is a Python script, you can use it from a Azure Pipeline yaml as well, and utilize the same capabilities as the Release Pipeline, but with yaml (which will allow versioning in Git as well as a bonus).

This version is longer because it's a yaml file, and GUI is prettier but this yaml below does the exact same thing as the release pipeline.

```yaml
# Add this at the begining of the pipeline (just above the tasks) to import the devops-flow-manager repo.
resources:
  repositories:
    - repository: devops-flow-manager  # The alias of the repo in this pipeline
      type: git                        # A git repository
      name: DevOps/templates           # project/repository
      ref: refs/heads/master           # What branch / tag to get.


stages:
- stage: Release Infrastructure
  displayName: Release the iBI infrastructure
  pool: 'On-prem (CaaS GER)'
  jobs:
  - job: ScanPreProd
    displayName: Scan Pre Prod
    steps:
      # Install the Python requirements for the Python script
      - task: Bash@3
        displayName: Install Python requirements
        inputs:
          targetType: 'inline'
          script: 'python3 -m pip install -r "./Templates/PipelineTriggerAndMonitor/requirements.txt"'
      # Trigger the release
      - task: PythonScript@0
        displayName: "Scan Pre Prod"
        timeoutInMinutes: 60
        inputs:
          scriptSource: 'filePath'
          scriptPath: './Templates/PipelineTriggerAndMonitor/pipeline-trigger-and-monitor.py'
          arguments: "release"
          pythonInterpreter: /usr/bin/python3
        env:
          # Passing JSON string from arguments does not work.
          aztm_organization: 'https://dev.azure.com/PDA-PDSS'
          aztm_project: 'DevOps'
          # Use environment variables to keep the pat out of the command line logs
          aztm_pat: $(System.AccessToken)
          aztm_variables: '{}'
          # Release Specific variables
          aztm_release_id: '13'
          aztm_artifacts: '{}'

  - job: ScanPreProd2
    dependsOn: [ScanPreProd] # Depends on the "ScanPreProd" job from earlier
    displayName: Scan Pre Prod 2
    steps:
      # Install the Python requirements for the Python script
      - task: Bash@3
        displayName: Install Python requirements
        inputs:
          targetType: 'inline'
          script: 'python3 -m pip install -r "./Templates/PipelineTriggerAndMonitor/requirements.txt"'
      # Trigger the release
      - task: PythonScript@0
        displayName: "Scan Pre Prod 2"
        timeoutInMinutes: 60
        inputs:
          scriptSource: 'filePath'
          scriptPath: './Templates/PipelineTriggerAndMonitor/pipeline-trigger-and-monitor.py'
          arguments: "release"
          pythonInterpreter: /usr/bin/python3
        env:
          # Passing JSON string from arguments does not work.
          aztm_organization: 'https://dev.azure.com/PDA-PDSS'
          aztm_project: 'DevOps'
          # Use environment variables to keep the pat out of the command line logs
          aztm_pat: $(System.AccessToken)
          aztm_variables: '{}'
          # Release Specific variables
          aztm_release_id: '13'
          aztm_artifacts: '{}'

  - job: ShouldRun
    dependsOn: [ScanPreProd2] # Depends on the "ScanPreProd2" job from earlier
    displayName: Should Run
    steps:
      # Trigger the release
      - task: Bash@3
        inputs:
          targetType: 'inline'
          script: |
            echo 'I should run'

  # Step to fail on purpose
  - job: PleaseFail
    dependsOn: [ScanPreProd] # Depends on the "ScanPreProd" job from earlier
    displayName: Please Fail # Will fail because of typo
    steps:
      - task: Bash@3
        inputs:
          targetType: 'inline'
          script: |
            exit(1)

  # Shouldn't be run because 'PleaseFail' job should always fail.
  - job: ShouldntBeTriggered
    dependsOn: [PleaseFail] # Depends on the "ScanPreProd2" job from earlier
    displayName: Shouldn't be triggered 
    steps:
      # Trigger the release
      - task: Bash@3
        inputs:
          targetType: 'inline'
          script: |
            echo 'I shouldn't run'
```

### A combination of all the above !

Each of the above options has it's pros and cons.

To maintain control over everything, it's recommended to have a healthy combination of all the above options.

**For example: (This is an example, steps names shouldn't reflect real world usage)**

1. **Release Pipeline 1:**
	1. **Stage 1:** Release to dev
		1. **Job 1:**  Kubernetes & Regression
			1. Release iBI DaaS Gateway Kubernetes Services - Development.
			2. Run regression on iBI DaaS Gateway Kubernetes Services - Development.
		2. **Job 2:** OnPrem Stuff
			1. Release Something in OnPrem (for example)
2. **Release Pipeline 2:**
	1. **Stage 1:** Do something cool
		1. **Job 1:**  Does something cool
3. **Release Pipeline 3:**
	1. **Stage 1:** Release to cloud environment
		1. **Job 1:**  Release App Service
		2. **Job 2:** Release Function App
		3. **Job 3:** Release Kubernetes iBI DaaS Databrick Services - Staging
		4. **Job 4:** Run some tests
5. **Big Boss Release Pipeline:**
	1. **Stage 1:** Trigger Release Pipeline 1
	2. **Stage 2:** Trigger Release Pipeline 2 (Depend on 1)
	3. **Stage 3:** Trigger Release Pipeline 3 (In parallel to 1 & 2 without dependencies)

## The Azure DevOps Release & Pipeline Trigger

The `pipeline-trigger-and-monitor.py` script from [this folder](https://dev.azure.com/PDA-PDSS/DevOps/_git/devops-flow-manager?path=/Templates/PipelineTriggerAndMonitor&version=GBfeature/add_variables_support&_a=contents) is a Python script to allow external triggering of pipelines and release (WIP for some features).

The script takes arguments from either the environment variables or the arguments from command line.

It then:
1. Triggers the pipeline / release using the provided arguments (auth using PAT).
2. Monitors the triggered pipeline / release using the Azure API.
3. Fails or success based on the result of the triggered pipeline.


## Example usage

In the example below we can see a release utilizing the `pipeline-trigger-and-monitor.py` script.

The first step triggers "Scan Pre Prod" - which triggers a Release pipeline.
Since "Scan Pre Prod" succeeded, steps "Scan Pre Prod 2" and "Please Fail" are triggered as well.

"Scan Pre Prod 2" triggers the same release pipeline as "Scan Pre Prod", which succeeds again, thus triggering the "Should Run" step, which is just a step that just prints something and will always work.

"Please Fail" is a step that fails on purpose to demonstrate a release failure - which worked, and thus didn't trigger the "Shouldn't be triggered" step.

![Image|1000](Pasted%20image%2020231004163140.png)