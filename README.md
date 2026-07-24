Azure Cost Visibility Dashboard

Video walkthrough: Azure Cost Visibility Dashboard

Project Overview:
A cost tracking and alerting system that gives business owners real-time visibility into their Azure spend, automatically alerts them before bills become a problem, and presents spending in plain language through a dashboard.

Business Problem

Most small businesses move to the cloud because they're expecting to save on costs, then the invoices start arriving full of line items like "Microsoft.Compute/virtualMachines — $340" that nobody in the business could predict, interpret,  or justify to anyone else.

This project remedies that completely. I built a system that:
•	Tracks spend across all Azure services and translates it into categories for business owners.
•	 Fires automatic alerts when spending hits thresholds ($50, $100, $200)
•	Sends email notifications via Logic Apps when an alert triggers
•	Displays a dashboard in Azure Workbooks showing spend by service, Resource, and resource group.

Architecture Flow

The system is event-driven. Azure Cost Management evaluates subscription spend against the monthly budget. When actual costs cross a defined threshold, it fires an event to an Action Group - a reusable notification hub that knows who to contact and how. The Action Group triggers a Logic App, which formats the alert into a readable email and delivers it to the right inbox. Everything that happens between the budget breach and the email landing is automated.
All supporting infrastructure - the Log Analytics workspace, diagnostic settings, and Workbook dashboard - runs alongside this flow to give the full picture of where spend is going and why.

<img width="1920" height="1080" alt="Azure Cost Management Consumption Budget (3)" src="https://github.com/user-attachments/assets/e0cd025a-3fb2-40ba-926e-993a6a55007e" />

Tools & Services Used

IaC	Terraform-(azurerm ~> 3.0)
Cost monitoring- Azure Cost Management Budgets
Alerting-	Azure Monitor Action Groups
Automation-	Azure Logic Apps
Logging-	Azure Log Analytics Workspace
Visualization-	Azure Workbooks + Resource Graph


What Gets Built

rg-cost-dashboard-[yourname]
├── Azure Monitor Action Group        → sends email when alert fires
├── Azure Monitor Alert Rules (x3)    → watch for $50 / $100 / $200 spend
├── Log Analytics Workspace           → stores diagnostic and activity data
├── Logic Apps Workflow               → triggered by alert, formats and sends email
└── Azure Workbook                    → spending dashboard visible in the portal


Prerequistites

Before deploying, install and configure:
  -Terraform
  -Azure CLI
  -An active Azure subscription
  -Sufficient permissions to create budgets, resource groups, monitoring resources, and Logic Apps


Terraform Configuration









Deploy (Terraform)

terraform init
terraform plan
terraform apply
When prompted, enter:

yes


Logic App Configuration (In Portal)

Terraform provisions the Logic App container. The workflow itself is a two-step setup in the portal.

Add the trigger and email action

Go to la-cost-alert-[yourname] in the portal
Open Logic app designer
Add trigger → search "request" → select When an HTTP request is received
Save — copy the HTTP POST URL that generates
Add action → search Gmail or Office 365 Outlook → select Send an email
Fill in To, Subject, and Body (use dynamic content → Body from the HTTP trigger)
Save
Connect it to the Action Group

Go to ag-cost-alerts-[yourname] in the portal → Edit
Under Actions, add a new row:
Action type: Logic App
Name: la-webhook
Selected: your Logic App
Save changes
Once set up, any budget threshold breach triggers the full chain: Cost Management → Action Group → Logic App → inbox.


Building the cost dashboard in azure workbooks

Azure Workbooks is a reporting tool built into the Azure portal. You will build a dashboard that shows spending by service and by resource group.

In the portal, search for Monitor → click Workbooks in the left menu
Click + New
Click + Add → Add query
Set the Data source to Azure Resource Graph
Paste the following query:

resourcecontainers
| where type == "microsoft.resources/subscriptions/resourcegroups"
| project resourceGroup, location

Click Run Query to verify it works, then click Done Editing
Click + Add → Add metric → select your subscription → choose Cost Management as the resource type
Click Save → give the workbook a name like Cost Visibility Dashboard → select your resource group → click Apply

Your workbook is now saved and accessible from the Workbooks section of Azure Monitor any time you open the portal.


Verification Checklist

Resource group rg-cost-dashboard-[yourname] exists in the portal
Budget budget-cost-[yourname] appears in Cost Management → Budgets
Action group ag-cost-alerts-[yourname] exists in Monitor → Action groups
Logic App la-cost-alert-[yourname] exists and shows a green Enabled status
Logic App designer shows an HTTP trigger and a Send email action
Log Analytics workspace law-cost-[yourname] exists
Azure Workbook is saved and visible in Monitor → Workbooks


Troubleshooting

Error
Cause
Resolution
BudgetStartDateInvalid
Start date must be the first of a current or future month
Update start_date in main.tf to the first of the current month
AuthorizationFailed on budget
Your account may need Cost Management Contributor role
Assign it: az role assignment create --role "Cost Management Contributor" --assignee <your-email> --scope /subscriptions/<sub-id>
Logic App email step asks for sign-in
Office 365 connection requires interactive auth
Sign in through the portal designer — this cannot be automated by Terraform
Alert email not received
Budget thresholds require actual spend to cross the limit
Use the portal to manually trigger a test action from the Action Group to verify email delivery



Teardown

"Terraform destroy"
Removes everything Terraform created. The Workbook was created manually in the portal and will need to be deleted there separately.
