# Managing Billing Configuration

[[TOC]]

# Billing accounts

All the resources that are used inside of a project have their own pricing e.g.
* App Engine: Network traffic, compute instances etc
* Billing accounts are just a way to pay for all the resources that we use (for a specific project)
* You can create multiple billing accounts for different projects (dev and prod projects)

## When will I get billed
Two types of billing accounts:

|Self-Serve                                      | Invoiced                                               |
|------------------------------------------------|--------------------------------------------------------|
| Billed at Threshold                            | Billed Monthly                                         |
| Monthly (Every 30 Days)                        | Check/Wire Transfer                                    |
| Threshold or Monthly (whichever happens first) | Need to contact Google Cloud Sales team to set that up |

Default is self service

## How do I manage access to billing info
Cloud IAM to assign specific Billing Roles
    * CTO - Billing Admin
    * Dev - Billing User - can create billable projects
    * Accounting - Billing Viewer
Lock down access based on their needs 

# Establishing Billing Budgets and Alerts
Under Billing tab you can create a Budget or Alert
