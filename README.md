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
5. terraform plan - shows error ///TBD: resolve it.
