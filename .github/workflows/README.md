# Steps to create an End-to-End Sample Workflow

## Pre-requisites
* Create a new Web App in Azure Portal with runtime stack as .NETCore and OS as Linux/Windows
* Copy Publish Profile Settings of the app

## Create a workflow file
* In the GitHub repo with Application code, Click on **Actions** tab and click on **New workflow** in the left pane to view the getting started experience.
* Select an existing template from the **Popular continuous integration workflows** or just select **Set up a workflow yourself** on the top right
* A new file called **.github/workflows/main.yml** gets created displaying a starter workflow
* Copy the sample YAML template from https://github.com/Azure/actions-workflow-samples/blob/master/asp.net-core-webapp-on-azure.yml
* Replace the `app-name` to your Web app name.
* Commit the workflow file

## Configure secrets in the GH repo:
* In the GH repo with Application code, Define a new secret under repository by navigating to **settings** > **secrets** > **Add a new secret** 
* Paste the contents for the downloaded publish profile file into the secret's value field
* Now in the workflow file in your branch: `.github/workflows/workflow.yml` replace the secret for the input `publish-profile:` of the deploy Azure WebApp action

## Demo Steps
* Commit a change in the [index file](https://github.com/bbq-beets/ignite/blob/ActionsDemo/Views/Home/Index.cshtml)
* You should see a new GitHub Action initiated in **Actions** tab.
* At the end of the execution, navigate to the App URL to visualise the change introduced

## Workflow YAML explained

* [Checkout](https://github.com/actions/checkout) Checkout your Git repository content into Github Actions agent.
* Environment setup using [Setup DotNet](https://github.com/actions/setup-dotnet) - Sets up a dotnet environment by optionally downloading and caching a version of dotnet by SDK version and adding to PATH .
* DotNet Build & Publish
* Deploy to App service using azure/webapps-deploy@v1 action which authenticates using [Azure Web App Publish Profile](https://github.com/projectkudu/kudu/wiki/Deployment-credentials#site-credentials-aka-publish-profile-credentials)
which we configured using the secret set up at the repo level

 
```yaml 
name: ASP.NET Core app deploy to Azure

on: [push,pull_request]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:

    # checkout the repo
    - uses: actions/checkout@master
    
    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 2.2.108 # Replace with specific Node version

    
    # dotnet build and publish
    - name: Build with dotnet
      run: dotnet build --configuration Release
    - name: dotnet publish
      run: |
        dotnet publish -c Release -o myapp 
        
    - name: 'Run Azure webapp deploy action using publish profile credentials'
      uses: azure/webapps-deploy@v1
      with: 
        app-name: dotnetcoreGH # Replace with your app name
        publish-profile: ${{ secrets.azureWebAppPublishProfile }} # Define secret variable in repository settings as per action documentation
        package: './myapp'
 ```

