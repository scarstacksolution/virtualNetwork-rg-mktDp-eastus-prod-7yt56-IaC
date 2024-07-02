# virtualNetwork-rg-mktDp-eastus-prod-7yt56-IaC

*Deploy Azure Resource Manager (ARM) Template using GitHub*

Steps:

- Use the "Upload File" fetaure to upload a working ARM Template JSON File
- Create a Resource group in your Azure portal using the following name for the Resource group: rg-mktDp-eastus-prod-7yt56
- Create a Service Principal for GitHub
	1.	Run the below script in a Bash session using Azure CloudShell. Be sure to replace the script below with your Subscription ID and Resource Group where you want your IaC ARM Template Cloud resource to be deployed:
       az ad sp create-for-rbac --name "winVM1ServicePrincipal" --role contributor --scopes /subscriptions/your-subscription-id/resourceGroups/your-resource-group-name --json-auth
	2.	Copy the output result from CloudShell and paste in the main GitHub repository to be used as a value under Settings -> Security -> Actions -> New repository secret with name field as AZURE_CREDENTIALS
- Navigate to the "Actions" Tab of the Repository and click "set up a workflow yourself" to create a YAML File.
- Update the name of the YAML File as: vNet1-IaC-ARM.yml
- Upload the below content into the YAML File main field area:

Here is the YAML formatted content:

```yaml
name: Azure ARM with Cronitor Schedule & Monitor

on:
  schedule:
    - cron: '*/5 * * * *' # Schedules workflow to run every 5 minutes
    # - cron: '30 7 1 6 *'  # Schedules workflow to run on June 1, 7:30AM
    # - cron: '10 9 * * *'  # Schedules workflow to run same day at 9:10AM
  workflow_dispatch:      # Deactivate this if you don't want a manual immediate execution included.
  # Note You can generate/test your Cron expressions at https://crontab.cronhub.io/

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      # Checkout code
      - uses: actions/checkout@main

      # Log into Azure
      - uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      # Deploy ARM template
      - name: Run ARM deploy
        uses: azure/arm-deploy@v1
        with:
          subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION }}
          resourceGroupName: rg-mktDp-eastus-prod-7yt56
          template: ./vNetARMTemplateDeploy.json

      # Output containerName variable from template
      - run: echo ${{ steps.deploy.outputs.containerName }}
```


*References* 

- https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deploy-github-actions?tabs=userlevel
- https://github.com/marketplace/actions/deploy-azure-resource-manager-arm-template
- https://docs.github.com/en/actions/using-workflows/triggering-a-workflow


*Using Cron to Schedule an Automated Workflow Trigger*

Key Things to Note:

1. To trigger a GitHub Actions workflow at a specific date and time, you can use the `schedule` event with a cron expression in your workflow YAML file. 
2. You can generate/test your Cron expressions at https://crontab.cronhub.io/
3. The scheduled event can be delayed during periods of high loads of GitHub Actions workflow runs.
4. * is a special character in YAML so you have to quote the string for your Cron expression schedule.
5. To integrate Cronitor monitoring to your Workflow, you can visit https://cronitor.io and navigate to the "Jobs" left-memnu of the website to follow the provided instructions.

Here is an example of how you can set this up:

1. **Create or Edit your GitHub Actions Workflow YAML File**: This file is usually located in `.github/workflows/`.

2. **Add the Schedule Trigger**: Use the `schedule` event with a cron expression to specify the date and time.

Here is an example of a YAML file that triggers a workflow at a specific time:

```yaml
name: Scheduled Workflow

on:
  schedule:
    # Run at 6:00 AM UTC on July 4, 2024
    - cron: '0 6 4 7 *'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Run a script
      run: echo "This run was triggered by a schedule!"
```

### Explanation:
- The `cron` expression `0 6 4 7 *` means:
  - `0` (minute): 0th minute (i.e., on the hour)
  - `6` (hour): 6 AM (UTC)
  - `4` (day of month): 4th day
  - `7` (month): July
  - `*` (day of week): Any day of the week

### Important Points:
- **UTC Time**: GitHub Actions uses UTC time for the schedule. You may need to adjust the time based on your local time zone.
- **Cron Syntax**: Make sure your cron syntax is correct. You can use an online cron expression generator to verify your syntax.

### Example for Weekly Trigger:
If you want to run a job every week on Monday at 9:00 AM UTC, the cron expression would be:

```yaml
on:
  schedule:
    - cron: '0 9 * * 1'
```

### Example for Daily Trigger:
If you want to run a job every day at 2:30 PM UTC, the cron expression would be:

```yaml
on:
  schedule:
    - cron: '30 14 * * *'
```

*References*

- https://github.com/marketplace/actions/cronitor-for-github-actions
- https://www.tinybird.co/docs/guides/ingesting-data/scheduling-with-github-actions-and-cron


