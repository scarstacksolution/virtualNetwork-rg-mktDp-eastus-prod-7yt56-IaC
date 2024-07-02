# virtualNetwork-rg-mktDp-eastus-prod-7yt56-IaC


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

on: [push]
name: Azure ARM
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

        # output containerName variable from template
      - run: echo ${{ steps.deploy.outputs.containerName }}

- Learn more at: https://github.com/marketplace/actions/deploy-azure-resource-manager-arm-template
