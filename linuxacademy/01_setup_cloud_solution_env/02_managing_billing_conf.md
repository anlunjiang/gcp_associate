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
* Cloud IAM to assign specific Billing Roles
    * CTO - Billing Admin
    * Dev - Billing User - can create billable projects
    * Accounting - Billing Viewer
Lock down access based on their needs 

# Establishing Billing Budgets and Alerts
Under Billing tab you can create a Budget or Alert
* Can set budget amount based on specific amount or the last month's spend
* You can establish notificationsusing Pub/Sub to programmatically receive spend updates on this budget
    * Budget name, threshold, cost, budget, currency etc
    * JSON format

## Billing Exports 
Under Billing you can click on a link called Billing export
* BigQuery export
    * Streamed throughout the day into a dataset
    * More details about the expense, and can use bq to analyse the data in a single dataset
    * can run reports to analyse each service cost across multiple projects 
    * Can optimise spending and cutout services that arent really being used 
* File export
    * Once a day
    * CSV or JSON file stored every day
    * Not as much information
    * Exported into a cloud storage bucket

# Summary
* Billing Accounts : Payment method and Billing roles, attached to one or more projects
* Each project has one billing account, but one billing account can look after many projects 
* Two types of billing accounts:
    * Self-serve
        * Charged at threshold or every 30 days
    * Invoice
        * Billed monthly but will need to talk to google cloud sales 
* Roles
    * Recommend using predefined 
    * Billing admin - Billing owner role
        * Manages payment details, credit card used
        * Link it to projects and manage user roles on the account
    * Billing user
        * Can link and unlink projects to an account
        * to enable users to create billable projects
    * Billing viewer 
        * Limited role - good for finance team
* Budgets and Alerts
    * Track spend on a billing account or on projects associated with that billing account 
    * Alerts allow you to trigger notifications when spending > percentage thresholds
    * Can use Pub/Sub to interact programmatically
* Billing Exports
    * BigQuery Export - runs throughout the day and streams it as it gets it
        * Can export data from multiple projects into the same dataset is a lot easier
    * File exports - JSON or CSV one file per day into a storage bucket 
