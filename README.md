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
