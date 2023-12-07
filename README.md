# Azure Pipeline for MkDocs Site Deployment

## Overview
This Azure Pipeline configuration is designed to automatically build and deploy a MkDocs site. It is split into two main jobs: 
1. Building the MkDocs site
2. Deploying the site to a Kubernetes cluster

## Prerequisites
- An Azure DevOps account and a project set up.
- A Kubernetes cluster with access configured via a service connection in Azure DevOps.
- A variable group named `wiki-token` with necessary tokens (user PAT token to Azure DevOps account).
- The `manifest.yml` file for Kubernetes deployment must be properly configured in the source code.

## Pipeline Configuration

### Triggers
- The pipeline is manually triggered (`trigger: []`).

### Variables
- `site`: Name of the MkDocs site.
- `site_url`: URL where the site will be hosted.
- `namespace`: Kubernetes namespace for deployment.

### Jobs

#### 1. Build Job (`azure_devops_agent`)
- Runs on an Ubuntu latest VM.
- Installs Python and Pip, and then MkDocs with necessary extensions.
- Builds the MkDocs site.

#### 2. Deployment Job (`self_signed_agent`)
- Depends on the build job's completion.
- Uses a Kubernetes service connection to log in to the cluster.
- Applies the Kubernetes manifest (`manifest.yml`) to deploy the site.
- Optionally adds a version tag on a pull request to the development branch.

### Steps

#### Building the MkDocs Site
1. Install dependencies and build tools.
2. Build the MkDocs site.
3. Publish the built site as a build artifact.

#### Deploying the Site
1. Log in to the Kubernetes cluster.
2. Apply the Kubernetes manifest to deploy the site.
3. (Optional) Tag the current build in the repo on successful deployment to the `dev` branch.

## Usage
1. **Setup**: Ensure all prerequisites are met.
2. **Run**: Trigger the pipeline manually in Azure DevOps.
3. **Monitor**: Check the build and deployment logs for any errors or successful deployment messages.
4. **Verify**: Once deployed, the site should be accessible at the configured `site_url`.