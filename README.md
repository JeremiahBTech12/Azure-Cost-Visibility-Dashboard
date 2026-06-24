Azure Cost Visibility Dashboard

- Video walkthrough: Azure Cost Visibility Dashboard

Project Overview
A cost tracking and alerting system that gives business owners real-time visibility into their Azure spend, automatically alerts them before bills become a problem, and presents spending in plain language through a dashboard.

Business Problem
Most small businesses move to the cloud because they're expecting to save money over having to manage their own servers. Then the invoices start arriving full of line items like "Microsoft.Compute/virtualMachines — $340" that nobody in the business can interpret, predict, or justify to anyone else.

This project remedies that completely. I built a system that:
   - Tracks spending across all Azure services and translates it into categories that makes sense to             business owners.
   - Fires automatic alerts when spending hits thresholds ($50, $100, $200)
   - Sends email notifications via Logic Apps when an alert triggers
   - Displays a dashboard in Azure Workbooks showing spend by service, Resource, and resource group.
     

What gets built:
- rg-cost-dashboard-[Jeremiah]
├── Azure Monitor Action Group        → sends email when alert fires
├── Azure Monitor Alert Rules (x3)    → watch for $50 / $100 / $200 spend
├── Log Analytics Workspace           → stores diagnostic and activity data
├── Logic Apps Workflow               → triggered by alert, formats and sends email
└── Azure Workbook                    → spending dashboard visible in the portal

Prerequisites:

- Install Terraform — download from https://developer.hashicorp.com/terraform/install, extract to C:\terraform\, then:
[Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";C:\terraform", "User")
terraform --version

- Login to Azure
az login
az account set --subscription "Azure subscription 1"
az account show


  FOLDER SETUP: Create a project directory and move into it.
   ( I used the Windows Powershell for this project along with VScode)
  - New-Item -ItemType Directory -Path "$HOME\cost-dashboard-001"
cd "$HOME\cost-dashboard-001"
New-Item -ItemType File main.tf, variables.tf, outputs.tf, terraform.tfvars

Terraform Configuration

Step 1 - Write variables.tf

Defines input variables for the deploying user's name, Azure region, alert email address, and resource tags - the only values that need to change between environments. (Shown in variables.tf folder)

Step 2- Write terraform.tfvars: (Shown in terraform.tfvars folder)
Replace your.email@example.com with the email address where you want to receive cost alerts.

Step 3- Write main.tf (Shown in main.tf folder)
The main Terraform file deploys the Azure infrastructure required for the cost visibility system.
Each resource block is explained before the code so you understand what it does and why it is written the way it is.

It creates:

Resource group
Log Analytics Workspace
Azure Monitor Action Group
Azure Consumption Budget
Logic App workflow container
Subscription diagnostic settings

outputs.tf
The outputs file prints useful values to the terminal after terraform apply completes.

Outputs include:

Resource group name
Log Analytics Workspace ID
Logic App access endpoint
Action Group ID
These values make the portal configuration and post-deployment validation easier.


Step 2- Write terraform.tfvars:

yourname    = "yourname"
location    = "East US"
alert_email = "your.email@example.com"
Update the start_date in main.tf to the first day of the current or a future month:

start_date = "2026-06-01T00:00:00Z"



- Logic App Configuration

- Testing and validation 

- Building the cost dashboard in Azure Workbooks

- Considerations
Cost Management data is not real-time — new usage can take several hours to appear in dashboards and trigger alerts
Very small charges may display as $0.00 if the Workbook visualization rounds values
The Cost Management API returns grouped rows as arrays, which required JSON Path mapping in the Workbook
The Logic App callback URL contains a sensitive sig parameter and should never be committed to source control or shared publicly — if exposed, regenerate the trigger URL immediately and update the Action Group receiver
Azure Resource Manager queries are a reliable alternative when Cost Management is not available through the standard Metrics picker

- Troubleshooting



- Skills Demonstrated
Azure cost monitoring and budget alerting
Azure Monitor Action Group configuration
Logic App webhook integration
Terraform-based Infrastructure as Code
Azure CLI troubleshooting
Azure Workbook dashboard creation
Cost Management API querying
Log Analytics and diagnostic settings
Cloud governance and FinOps fundamentals

- Final Outcome
This project produced a working Azure cost visibility and alerting solution. The system monitors subscription-level spend, sends email alerts when cost thresholds are reached, triggers a Logic App workflow from an Azure Monitor Action Group, and displays cost data through an Azure Workbook dashboard.

The result is a practical FinOps-focused Azure project that demonstrates cloud cost monitoring, automation, reporting, and business-focused cloud governance.

- Teardown


To remove the deployed infrastructure:

terraform destroy
Type yes when prompted. This deletes all Terraform-managed resources including the resource group, budget, Logic App, and Log Analytics Workspace.
