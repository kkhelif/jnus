# GitHub Action - JSON Variables
Are you tired of repeating and duplicating environment variables for environment specific deployment steps and having to maintain multiple workflows dependent on the same set of variables? The JSON variables action lets you define all you variable in a single json file, scope to environments and substitute/reuse in new variables, hence simplifying variable configuration maintenance.

Based on a json file, this GitHub Action will:
  1. Render variables based on the environment context specified in the workflow
  2. Substitute variables in other variables
  3. Set variables as environment variables.

# Getting started

Add a variable file as json anywhere in the repository.
The format of the file should be:

```json
{
  "Variables": [
    {
      "Name": "HostName",
      "Value": "someDevHostName",
      "Scope": {
        "Environment": [
          "Dev"
        ]
      }
    },
    {
      "Name": "HostName",
      "Value": "someDevTestHostName",
      "Scope": {
        "Environment": [
          "DevTest"
        ]
      }
    }
  ]
}
```


Now in your environment specific job, you want to render this set of variables as environment variables, add the following action. The configFile parameter can either be the full path og just the filename. Scope is the environment name, which is case sensitive.:

```json
- name: Set environment specific variables
  uses: jnus/jsonvariables@v0.1
  with:
    scope: Dev
    configFile: 'variables.minimal.json'
```
 
The environment specific variables defined variables.json will be created as environment variables ready to use for poking e.g. appsettings.json file or as parameters for deploying to misc. compute targets. ~~Note the job must contain an Environment declaration and will only scoped to the particular job.~~

# Variable substitution.
The JSON variables action can do variable substitution, hence simplifying the variable maintenance. It's a flexible concept for adjusting the environment variables based on the environment context of your e.g. deployment job. By using a simple expression syntax, variables can be combined and thereby reducing the complexity of maintaining multiple environments. 

The following variable 'Url' reference the 'HostName' variable and based on the environment context, renders the environment variable as either https://someDevHostName or https://someDevTestHostName. 

```json
{
  "Variables": [
    {
      "Name": "HostName",
      "Value": "someDevHostName",
      "Scope": {
        "Environment": [
          "Dev"
        ]
      }
    },
    {
      "Name": "HostName",
      "Value": "someDevTestHostName",
      "Scope": {
        "Environment": [
          "DevTest"
        ]
      }
    },
    {
      "Name": "Url",
      "Value": "https://#{HostName}.com",
      "Scope": {}
    }
  ]
}
```

# How to use environment specific environment variables
The context specific environment variables, rendered in this action, can be reference as any other environment variable in the current job and implicitly be used by other actions such as microsoft/variable-substitution.

The following job, deploys a web app and uses the rendered 'HostName' variable to target a particular resource in Azure and 'Url' as the fqdn for the web app:

```yaml
deploy_to_dev:
    runs-on: ubuntu-latest
        ...
    environment: 
      name: Dev
      url: ${{env.Url}}
    steps:
      - name: Set environment specific variables
        uses: jnus/jsonvariables@v0.1
        with:
            scope: Dev
            configFile: 'variables.minimal.json'
        ...
      - name: Deploy to Azure WebApp
        uses: azure/webapps-deploy@v2
        id: web-app
        with:
          app-name: ${{env.HostName}}-api-wa
          package: ./webapp/bin/Release/net5.0/publish
```

# Status
Action is not yet published and currently being tested. It is not intended for production usage yet. Certain features are missing before v1.0 can be created
- Support org and repo secret substitution
- Read environment context directly in action and not as a parameter

Topic regarding this -> https://github.community/t/variable-management-in-github/200400?u=jnus
