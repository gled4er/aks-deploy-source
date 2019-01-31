# aks-deploy-source
Source repo that triggers CI/CD and fabrikate generation into next repo. An example of this destination repo is location [here](https://github.com/samiyaakhtar/aks-deploy-destination). 

# Setup

First we need to create a personal access token which will be used to push to the repository. Go to github settings > Developer settings > Personal access token. There should be a list of permissions to grant this new token, at the minimum it should have rights to read and write to a repository. Create the token and store it somewhere as you will not be able to view it again.

Setup azure pipelines on the source repository by creating a new build pipeline in Pipelines > Builds:

1. Copy `generate.sh` into root folder of your project
2. Go into pipeline settings and add a new variable called `accesstoken` and set the value to your personal access token. Make sure the variable is set to secret. 
3. Add a variable `destination_repo_url` and set it to the destination repo, for eg. `samiyaakhtar/aks-deploy-destination`
4. Update the azure-pipelines.yml file to look like the following

```
trigger:
- master

pr: none

pool:
  vmImage: 'Ubuntu-16.04'

steps:
- checkout: self
  persistCredentials: true
  clean: true

- task: ShellScript@2
  inputs:
    scriptPath: verify.sh
  condition: eq(variables['Build.Reason'], 'PullRequest')

- task: ShellScript@2
  inputs:
    scriptPath: generate.sh
  env:
    ACCESS_TOKEN: $(accesstoken)
    COMMIT_MESSAGE: $(Build.SourceVersionMessage)
    AKS_MANIFEST_REPO: $(destination_repo_url)

```
This makes sure after every commit the source code will be checked out, yaml generated and the files will be placed in the second repo. 

4. Make a test commit to the source repo and make sure the pipeline gets executed, and files end up in the destination repo. 


