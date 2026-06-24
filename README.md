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

It creates:

Resource group
Log Analytics Workspace
Azure Monitor Action Group
Azure Consumption Budget
Logic App workflow container
Subscription diagnostic settings

Provider and data sources

      The azurerm provider is the Terraform plugin that knows how to talk to Azure. features {} is required       but can be left empty for most configurations. The azurerm_client_config data source reads your             current az login session and gives Terraform your subscription ID and tenant ID, which are needed for       budget and alert scope configuration.

Resource group

      Every Azure resource must live inside a resource group. Think of it as a folder — it holds all the          resources for this project together, controls who has access, and makes cleanup easy (deleting the          resource group deletes everything inside it).

Log Analytics Workspace

      Log Analytics is Azure's central logging and querying service. It stores activity logs, diagnostic          data, and metrics from across your Azure resources in one place. The sku = "PerGB2018" means you pay        only for data ingested — there is no flat monthly fee, and at lab scale the cost is negligible.

      The retention_in_days = 30 setting means log data is automatically deleted after 30 days. This is the       minimum allowed value and keeps costs low for a lab environment.

 Action Group

      An Action Group is Azure Monitor's way of defining what should happen when an alert fires. It is a          reusable list of notification targets — email addresses, SMS numbers, Logic App webhooks, and more.         You define it once and attach it to as many alert rules as you like.

      short_name is required and must be 12 characters or less. It appears in SMS notifications.

      The email_receiver block defines who gets notified. Setting use_common_alert_schema = true means the       email body uses a standardized format that works consistently across all alert types — this is the          recommended setting.

Budget with alert thresholds

      azurerm_consumption_budget_subscription creates an Azure Cost Management budget at the subscription         level. This is what watches your overall Azure spending and fires alerts when you cross thresholds.

      time_grain = "Monthly" resets the budget tracking at the start of each calendar month — which is how       Azure bills, so this makes the most intuitive sense.

      The start_date must be the first day of the current or a future month in RFC3339 format. Update the       year and month to match when you are running this lab.

      amount = 200 sets the total monthly budget at $200. This is the ceiling — the alert thresholds below       fire at percentages of this amount. At 25% ($50), 50% ($100), and 100% ($200).

      Each notification block defines one alert threshold. threshold = 25 means "alert when actual spend          reaches 25% of the budget amount." operator = "GreaterThan" means the alert fires when spending             crosses the threshold going up. The contact_groups list connects each alert to the Action Group you         created above, which is what actually sends the email.

Logic App Workflow

      A Logic App is Azure's low-code automation service. It connects different systems together through          triggers and actions — when something happens (trigger), do something else (action). In this project,       the Logic App receives a webhook call from Azure Monitor when a budget alert fires, formats the alert       data into a readable message, and sends a notification email.

      logic_app_workflow creates the Logic App container. The workflow definition (the actual trigger and         action logic) is managed separately in the portal after deployment — Terraform provisions the               resource, and you configure the steps in the visual designer. This is intentional: workflow logic is        easier to build and test in the visual designer than in Terraform HCL.

Diagnostic settings — send activity logs to Log Analytics

      This resource tells Azure to forward the subscription's activity log into your Log Analytics                workspace. The activity log records every management operation on your subscription: who created or         deleted what, when, and from where. Without this, Log Analytics has no data to query.

      target_resource_id is the subscription itself (not an individual resource), which is why the scope is       the full subscription ID. log_analytics_workspace_id is where the logs get written.


Step 4- Write outputs.tf (Shown in outputs.tf folder)

Outputs print useful values to your terminal after terraform apply completes. They save you from hunting through the portal for information you need for the next steps.

Outputs include:

Resource group name
Log Analytics Workspace ID
Logic App access endpoint
Action Group ID
These values make the portal configuration and post-deployment validation easier.

Step 5- Deploy

Windows (PowerShell) — these commands are identical on both platforms:

terraform init
You should see: Terraform has been successfully initialized.

terraform plan

Review the plan. You should see 6 resources to add: resource group, log analytics workspace, action group, consumption budget, logic app workflow, and diagnostic setting.

terraform apply
Type yes when prompted. Deployment takes approximately 2–3 minutes.

Step 6- Configure the Logic App Workflow in the Portal

Terraform created the Logic App container. Now you will add the trigger and action steps using the visual designer.

1. In the Azure portal, navigate to your resource group rg-cost-dashboard-[yourname]
2. Click on la-cost-alert-[yourname]
3. In the left menu, click Logic app designer
4. Click Add a trigger → search for HTTP → select When a HTTP request is received
5. Copy the HTTP POST URL that appears — this is the webhook URL Azure Monitor will call when a budget alert fires
6. Click + New step → search for Office 365 Outlook → select Send an email (V2)
7. Sign in with your Microsoft account when prompted
8. Fill in the email fields:
	- To: your alert email address
	- Subject: Azure Cost Alert — Budget Threshold Reached
	- Body: Click Add dynamic content and add the Body field from the 	HTTP trigger — this contains the full alert details
9. Click Save

Connect the Logic App to the Action Group:

After saving, you need to add the Logic App as a receiver in the Action Group.
Replace <sub-id> with your subscription ID (from az account show --query id -o tsv) and <logic-app-callback-url> with the URL you copied from the designer.

Step 7 - Build the Cost Dashboard in Azure Workbooks
Azure Workbooks is a reporting tool built into the Azure portal. You will build a dashboard that shows spending by service and by resource group.

10. In the portal, search for Monitor → click Workbooks in the left menu
11. Click + New
12. Click + Add → Add query
13. Set the Data source to Azure Resource Graph
14. Paste the following query:
15. Click Run Query to verify it works, then click Done Editing
16. Click + Add → Add metric → select your subscription → choose Cost Management as the resource type
17. Click Save → give the workbook a name like Cost Visibility Dashboard → select your resource group → click Apply

Your workbook is now saved and accessible from the Workbooks section of Azure Monitor any time you open the portal.

Verification Checklist

   Resource group rg-cost-dashboard-[yourname] exists in the portal
   Budget budget-cost-[yourname] appears in Cost Management → Budgets
   Action group ag-cost-alerts-[yourname] exists in Monitor → Action groups
   Logic App la-cost-alert-[yourname] exists and shows a green Enabled status
   Logic App designer shows an HTTP trigger and a Send email action
   Log Analytics workspace law-cost-[yourname] exists
   Azure Workbook is saved and visible in Monitor → Workbooks




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
