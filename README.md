# Power On/Off on schedule for Azure Virtual Machines

While migrating from on-premise to the cloud, especially on scale lift and shift, we inevitably face the exponential growth of cloud cost. Considerable piece of this growth migh t be caused by VMs running cost. While VM-hosted workload is still essential for a lot of businesses and might bring major share of cloud costs, a lot of those VMs are not required to be enabled 24/7. Having a workload schedule for this kind of VMs might be a huge cost saver.

This project is intended to provide a set up of schedled workloads and VMs on/off in Azure Cloud. While the schedule is automated for the most part, providing the self-service is essentials for teams to enable their VMs on demand, whenever they'll need it, without having to deeply involve the Ops teams. For this set up and use case we'll consider the following options to provide the self-service mechaics:
1. Enabling the neded VM by sending the request to the Cloud On-Schedule engineer - pros: easy to implement, cons: potentially long response time.
2. Providing the way to enable the VM through the access to Azure Poral UI - pros: relatively easy to implement, convenient for the most users that have previous experience with Azure Cloud; cons: complex permissions set up needs to be configured, need to educate enginers, who are not amiliar with Azure Cloud Portal yet.

## Steps that need to be implemented to set this up

1. Set up all acesses, configure the toolchain - https://medium.com/@yoursshaan2212/terraform-to-provision-multiple-azure-virtual-machines-fab0020b4a6e 
2. Automatically Provision infrastructure in Azure (VMs) to test the automation schedule.- https://medium.com/@yoursshaan2212/terraform-to-provision-multiple-azure-virtual-machines-fab0020b4a6e
3. Install and configure the Azure Start/Stop VMs v2 feature https://learn.microsoft.com/en-us/azure/azure-functions/start-stop-vms/overview
4. Configure the on/off schedules for VMs based on its grouping criteria
5. Configure the permission goup set for the self-service access (enabling VMs)
6. Document the architecture and the whole configuration flow
7. Present the flow to AlexeyT and VitaliyG to gain the approval
8. Create tasks for the team to implement.


### Settig up access and the toolchain
1. Terraform - Did so using the guide here https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli. Mac with homebrew. No issues so far.
2. Microsoft Azure account - https://azure.microsoft.com/en-in/free/ - did pay as you go, no issues.
3. Azure Account configuration - go to Azure Active Directory and add the app named "terraform", then go to Suscriptions>[Sub Name]>IAM and assign the Contributor role for this app.
4. Teraform Code - all good. terraform fmt - do not like the result, reverted the change.
5. terraform plan - shows error. The error is:
```terraform
Error: Error ensuring Resource Providers are registered.
???
??? Terraform automatically attempts to register the Resource Providers it supports to
??? ensure it's able to provision resources.
???
??? If you don't have permission to register Resource Providers you may wish to use the
??? "skip_provider_registration" flag in the Provider block to disable this functionality.
???
??? Please note that if you opt out of Resource Provider Registration and Terraform tries
??? to provision a resource from a Resource Provider which is unregistered, then the errors
??? may appear misleading - for example:
???
??? > API version 2019-XX-XX was not found for Microsoft.Foo
???
??? Could indicate either that the Resource Provider "Microsoft.Foo" requires registration,
??? but this could also indicate that this Azure Region doesn't support this API version.
???
??? More information on the "skip_provider_registration" flag can be found here:
??? https://www.terraform.io/docs/providers/azurerm/index.html#skip_provider_registration
???
??? Original Error: Cannot register provider Microsoft.ServiceFabric with Azure Resource Manager: resources.ProvidersClient#Register: Failure responding to request: StatusCode=403 -- Original Error: autorest/azure: Service returned an error. Status=403 Code="AuthorizationFailed" Message="The client 'ba2da2d0-56ba-4b8b-ab95-a94c06af7a20' with object id 'ba2da2d0-56ba-4b8b-ab95-a94c06af7a20' does not have authorization to perform action 'Microsoft.ServiceFabric/register/action' over scope '/subscriptions/ea1e5602-2bef-47c2-8355-74d08c45f9d5' or the scope is invalid. If access was recently granted, please refresh your credentials.".
???
???   with provider["registry.terraform.io/hashicorp/azurerm"],
???   on main.tf line 2, in provider "azurerm":
???    2: provider "azurerm" {

```

In order to fix it, we need to make sure the AD app terraform has a Contributor role assigned for our current subscription.

### Automatically Provision infrastructure in Azure (VMs)
Generally following [the guide](https://medium.com/@yoursshaan2212/terraform-to-provision-multiple-azure-virtual-machines-fab0020b4a6e)

1. After applying the change, make sure you destroy all the set up if you do not need it any more to avoid unnecesary $ spending:
```terraform
terraform plan -destroy -out main.destroy.tfplan
terraform apply main.destroy.tfplan
```
2. While applying the change, you can face with an error that the resources of certain size are not available at the certain region. Error message exmple:
```terraform
??? Error: compute.VirtualMachinesClient#CreateOrUpdate: Failure sending request: StatusCode=0 -- Original Error: autorest/azure: Service returned an error. Status=<nil> Code="SkuNotAvailable" Message="The requested size for resource '/subscriptions/ea1e5602-2bef-47c2-8355-74d08c45f9d5/resourceGroups/linuxnode-RG/providers/Microsoft.Compute/virtualMachines/linuxnode-01' is currently not available in location 'eastus2' zones '' for subscription 'ea1e5602-2bef-47c2-8355-74d08c45f9d5'. Please try another size or deploy to a different location or zones. See https://aka.ms/azureskunotavailable for details."
```
Try to change region or VM size to mitigate. You can do so by changing ```node_location``` and ```vm_size``` respectively at ```terraform.tfvars```.

### Deploying Start/Stop VMs during off-hours feature into Azure
Generally following [the guide here](https://github.com/microsoft/startstopv2-deployments)

The deployment was done using [this guide](https://learn.microsoft.com/en-us/azure/azure-functions/start-stop-vms/deploy) by pressing the *Deployto Azure* button in [the GitHub Repo](https://github.com/microsoft/startstopv2-deployments/blob/main/README.md) (Global Azure) section.

Parameters used for the deployment:
![Parameters used for deployment](img/Create_Start_Stop_VMs_during_off_hours_-_V2_-_Microsoft_Azure-2.png)

### Configuring the Start/Stop VMs during off-hours feature into Azure

Generally following [the guide,Configure schedules overview section and below](https://learn.microsoft.com/en-us/azure/azure-functions/start-stop-vms/deploy)

1. In order to STOP machines during off-hours schedule, configure the ```ststv2_vms_Scheduled_stop``` Logic App, every day at 23:00, Function-Try JSON is below:
```json
{
  "Action": "stop",
  "EnableClassic": false,
  "RequestScopes": {
    "ExcludedVMLists": [],
    "ResourceGroups": [
      "/subscriptions/ea1e5602-2bef-47c2-8355-74d08c45f9d5/resourceGroups/linuxnode-RG/" /// Resource group's GUID
    ]
  }
}
```

2. In order to START machines on schedule, configure the ```ststv2_vms_Sequenced_start``` Logic App, every day at 7:00, Function-Try JSON is below:
```json
{
  "Action": "start",
  "EnableClassic": false,
  "RequestScopes": {
    "ExcludedVMLists": [],
    "ResourceGroups": [
      "/subscriptions/ea1e5602-2bef-47c2-8355-74d08c45f9d5/resourceGroups/linuxnode-RG/" /// Resource group's GUID
    ]
  }
}
```

3. In order to STOP unused machines based on the avg Percentage CPU utilization, configure the ```ststv2_vms_AutoStop``` Logic App, every hour interval, Function-Try JSON is below:
```json
{
  "Action": "stop",
  "AutoStop_Condition": "LessThan",
  "AutoStop_Description": "Alert to stop the VM if the CPU % falls below the threshold",
  "AutoStop_Frequency": "00:05:00",
  "AutoStop_MetricName": "Percentage CPU",
  "AutoStop_Severity": "2",
  "AutoStop_Threshold": "5",
  "AutoStop_TimeAggregationOperator": "Average",
  "AutoStop_TimeWindow": "01:00:00",
  "EnableClassic": true,
  "RequestScopes": {
    "ExcludedVMLists": [],
    "ResourceGroups": [
      "/subscriptions/ea1e5602-2bef-47c2-8355-74d08c45f9d5/resourceGroups/linuxnode-RG/" /// Resource group's GUID
    ]
  }
}
```
The above means that the VM will be stopped if in the previous hour it's average Percentage CPU utilization was below 5%.