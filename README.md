# virtualNetwork-rg-mktDp-eastus-prod-7yt56-IaC


Steps:

- Use the Uplaod File fetaure to upload a working ARM Template JSON File
- Create a Resource group in your Azure portal using the following name for the Resource group: rg-mktDp-eastus-prod-7yt56
- Create a Service Principal for GitHub
	1.	Run the below script in aBash session using Azure CloudShell. Be sure to replace the script below with your Subscription ID and Resource Group where you want your IaC ARM Template Cloud resource to be deployed:
       az ad sp create-for-rbac --name "winVM1ServicePrincipal" --role contributor --scopes /subscriptions/your-subscription-id/resourceGroups/your-resource-group-name --json-auth
	2.	Copy the output and paste in the main GitHub repository to be used as a value under Settings -> Security -> Actions -> New repository secret with name field as AZURE_CREDENTIALS
- Navigate to the "Actions" Tab of the Repository and click "set up a workflow yourself" to create a YAML File.
- Update the name of the YAML File as: winVM1-IaC-ARM
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
          template: ./WindowsVM_ARMTemplate.json
          parameters: adminUsername=useradmin1
                      adminPassword=Password123$
                      location=eastus
                      dnsLabelPrefix=winvmeastus7yt56

        # output containerName variable from template
      - run: echo ${{ steps.deploy.outputs.containerName }}


- From the YAML contant above, "WindowsVM_ARMTemplate.json" is the name of your ARM Template file and "winvmeastus7yt56" is used to definte the DNS Name label for the Virtual machine being deployed.
- Parameters that can be added in the YAML File above for a Virtual Machine ARM Deployment are: OSVersion, adminPassword, adminUsername, dnsLabelPrefix, location, publicIPAllocationMethod, publicIpName, publicIpSku, securityType, vmName, vmSize.

