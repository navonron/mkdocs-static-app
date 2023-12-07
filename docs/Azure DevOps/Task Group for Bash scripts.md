## Description

Task groups do not detect as parameters to the task group variables that are supplied to the Environment Variables section of a task (Checked with PowerShell & Bash & Command Line).

I discovered it when I was making the DevOps 'scripts' git repo for common tools for the Azure DevOps Agent (Tasks and other tools).

[Link to a GitHub issue describing the issue as Microsoft doesn't do anything about it](https://github.com/MicrosoftDocs/azure-devops-docs/issues/3355)

## Steps to Reproduce
1. Create a Task Group from Azure DevOps Steps
2. In the Environment Variables put the name of the variables that you want to be set as parameters outside (Parameters are defined like: `$(PARAMETER_NAME)`.
3. Watch as nothing happens outside the Task Group because Azure DevOps is dumb.

## Solution
The solution is kind of a workaround.

If you want to run a bash script for example you'll have to use the inline script mode for parameters to be exported outside the Task Group.
Since inline mode defeats the point of scripts managed in a repo, I used this workaround:

1. Create a Task Group from a "Command Line" task in Azure DevOps.
2. In the "Script" box export the variables into the environment variables of the agent like so:
	
	![](Task%20Group%20for%20Bash%20scripts%202.png)

3. Run the bash script using `bash <Script Path>` or `source <Script Path>` etc.
4. Go outside the task to the Task Group definition and watch the variables as parameters (set defaults and descriptions as needed)
	![](Task%20Group%20for%20Bash%20scripts%201.png)

