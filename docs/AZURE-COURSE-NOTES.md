# ☁️ Microsoft Azure From Zero to Hero – Course Notes

My study notes for **Microsoft Azure From Zero to Hero – The Complete Guide** (Udemy, by Memi Lavi).
The course teaches Azure by building **Readit**, a fictional online bookstore, from several Azure services. This repository contains the **Inventory** part of that system.

---

## 📑 Contents

1. [Cloud Fundamentals](#1-cloud-fundamentals)
2. [Azure Basics](#2-azure-basics)
3. [The Readit Case Study](#3-the-readit-case-study)
4. [Virtual Machines](#4-virtual-machines)
5. [Networking](#5-networking)
6. [App Service](#6-app-service)
7. [Containers & AKS](#7-containers--aks)
8. [Data Services](#8-data-services)
9. [Storage](#9-storage)
10. [Serverless: Functions & Logic Apps](#10-serverless-functions--logic-apps)
11. [Messaging](#11-messaging)
12. [Identity & Security](#12-identity--security)
13. [Monitoring](#13-monitoring)
14. [Cost Management & Best Practices](#14-cost-management--best-practices)
15. [What I Built in This Repo](#15-what-i-built-in-this-repo)

---

## 1. Cloud Fundamentals

| Model    | You manage                           | Azure examples                    |
| -------- | ------------------------------------ | --------------------------------- |
| **IaaS** | OS, runtime, app, data               | Virtual Machines                  |
| **PaaS** | App and data only                    | App Service, Azure SQL            |
| **FaaS** | Just the function code               | Azure Functions                   |
| **SaaS** | Nothing, you only use the software   | Microsoft 365                     |

**Why the cloud:** pay-as-you-go pricing, elasticity (scale up/down on demand), high availability, global reach, and no hardware to manage.

**Deployment models:** public cloud, private cloud, hybrid cloud.

## 2. Azure Basics

- **Account & subscription**: a subscription is the billing boundary. A free account comes with credit for the first 30 days.
- **Azure Portal**: the web UI at `portal.azure.com`.
- **Azure CLI / PowerShell / Cloud Shell**: scripted management.
- **Resource**: any single service instance (VM, database, web app…).
- **Resource Group**: a logical container for resources that share a lifecycle. Deleting the group deletes everything in it.
- **Regions & Availability Zones**: datacenter locations. Zones are physically separate datacenters inside one region.
- **Tags**: key/value metadata for organizing resources and costs.

## 3. The Readit Case Study

Readit is an online bookstore made of several small apps. Each one uses a different Azure service:

| Component      | Purpose                         | Azure service used                  |
| -------------- | ------------------------------- | ----------------------------------- |
| Catalog        | Browse books                    | Virtual Machine / App Service       |
| **Inventory**  | Manage stock (**this repo**)    | **App Service + Azure SQL**         |
| Weekly Deal    | Show a weekly deal              | Azure Functions                     |
| Shopping Cart  | Hold cart items                 | Containers / AKS                    |
| Orders         | Process orders                  | Cosmos DB, messaging, Functions     |
| Images         | Book cover images               | Blob Storage                        |

The architecture is designed step by step: requirements → choose a service → build → secure → monitor.

## 4. Virtual Machines

- IaaS: full control of the OS.
- Choose **image**, **size** (vCPU/RAM), **disk** type (HDD / Standard SSD / Premium SSD).
- Connect with **RDP** (Windows) or **SSH** (Linux).
- **Availability Set** protects against rack failures (fault/update domains). **Availability Zones** protect against datacenter failures.
- **Virtual Machine Scale Sets** add or remove identical VMs automatically.
- Stopping a VM from the **portal** (*deallocated*) stops compute billing. Shutting it down from inside the OS does not.

## 5. Networking

- **Virtual Network (VNet)**: a private network in Azure, divided into **subnets**.
- **Network Security Group (NSG)**: allow/deny rules by port, protocol and source.
- **VNet Peering**: connect two VNets.
- **Load Balancer** (layer 4) vs **Application Gateway** (layer 7, with an optional WAF).
- **Service Endpoints / Private Endpoints**: reach PaaS services (SQL, Storage) over the private network.
- **Bastion**: secure RDP/SSH without public IPs.

## 6. App Service

- PaaS hosting for web apps and APIs (.NET, Java, Node, Python, PHP).
- **App Service Plan** sets the compute tier (Free, Basic, Standard, Premium). Several apps can share one plan.
- **Scale up** = a bigger tier. **Scale out** = more instances (manual or autoscale rules).
- **Deployment slots** (Standard and up) let you deploy to *staging* and then **swap** with no downtime.
- **Configuration**: app settings and connection strings are injected as environment variables and override `appsettings.json`.
- Deploy from Visual Studio, VS Code, ZIP deploy, GitHub Actions, or Azure DevOps.

## 7. Containers & AKS

- **Docker** packages an app with its dependencies into an image.
- **Azure Container Registry (ACR)**: a private image registry.
- **Azure Container Instances (ACI)**: run a single container quickly.
- **Azure Kubernetes Service (AKS)**: managed Kubernetes (pods, deployments, services) for orchestrating many containers.

## 8. Data Services

| Service                | Type                    | When to use                               |
| ---------------------- | ----------------------- | ----------------------------------------- |
| **Azure SQL Database** | Relational (SQL Server) | Structured data, transactions             |
| SQL Managed Instance   | Near-100% SQL Server    | Lift-and-shift of existing SQL Server     |
| **Cosmos DB**          | NoSQL, globally distributed | Flexible schema, low latency, global  |
| MySQL / PostgreSQL     | Managed open-source DBs | Open-source apps                          |

Azure SQL key points:
- Purchasing models: **DTU** or **vCore**. There is also a **Serverless** tier that auto-pauses.
- The **server firewall** blocks everything by default. Allow your client IP and Azure services.
- Built-in backups, point-in-time restore, geo-replication.
- Supports **Microsoft Entra authentication**, which allows passwordless access with Managed Identity.

## 9. Storage

- **Storage Account** services: **Blob**, **File**, **Queue**, **Table**.
- Blob access tiers: **Hot**, **Cool**, **Cold**, **Archive** (cheaper storage, more expensive access).
- Redundancy options: LRS, ZRS, GRS, RA-GRS.
- Secure access with **SAS tokens**, access keys, or Entra ID RBAC.

## 10. Serverless: Functions & Logic Apps

- **Azure Functions**: event-driven code. **Triggers** (HTTP, Timer, Queue, Blob…) and **bindings** (declarative input/output).
- Hosting: **Consumption** (pay per execution), Premium, or Dedicated.
- **Durable Functions** for stateful workflows.
- **Logic Apps**: low-code workflows with hundreds of connectors.

## 11. Messaging

| Service         | Pattern                                | Example                     |
| --------------- | -------------------------------------- | --------------------------- |
| Storage Queue   | Simple queue                           | Background jobs             |
| **Service Bus** | Enterprise queues & topics, ordering   | Order processing            |
| **Event Grid**  | Event routing (reactive)               | "Blob created" → Function   |
| Event Hubs      | High-throughput event streaming        | Telemetry, logs             |

## 12. Identity & Security

- **Microsoft Entra ID (formerly Azure AD)**: the identity provider for users and apps.
  - **App registration** gives an app a Client ID and redirect URIs, so it can sign users in with OpenID Connect.
  - In .NET, `Microsoft.Identity.Web` → `AddMicrosoftIdentityWebAppAuthentication`.
- **RBAC**: assign roles (Owner, Contributor, Reader…) at the subscription, resource group, or resource scope.
- **Managed Identity**: an identity for an Azure resource, so you don't keep credentials in code.
  - *System-assigned* (tied to the resource) or *user-assigned* (a standalone resource).
- **Key Vault**: store secrets, keys, and certificates. Read them from App Service with Key Vault references.
- **Defender for Cloud**: security posture and recommendations.

## 13. Monitoring

- **Azure Monitor**: metrics and logs for all resources.
- **Log Analytics**: query logs with **KQL**.
- **Application Insights**: APM for apps (requests, failures, dependencies, performance).
- **Alerts**: fire on metric/log conditions and notify through **Action Groups**.

## 14. Cost Management & Best Practices

- Use the **Pricing Calculator** before building, and **Cost Management + Budgets** to get alerts.
- Use Free/Basic tiers for learning. Deallocate VMs when you're not using them.
- **Delete resource groups** after each lab.
- Name resources consistently and **tag** them.
- **Well-Architected Framework** pillars: Reliability, Security, Cost Optimization, Operational Excellence, Performance Efficiency.

## 15. What I Built in This Repo

The **Readit Inventory** app ([README](../README.md)):

- ✅ ASP.NET Core Razor Pages app hosted on **Azure App Service**
- ✅ Data in **Azure SQL Database** through EF Core
- ✅ Configuration overridden by App Service settings
- ✅ User sign-in with **Microsoft Entra ID**
- ✅ Passwordless DB connection using **Managed Identity**
- ✅ Upgraded from .NET Core 3.1 to **.NET 10**

### Key takeaways

1. Keep secrets out of source control. Use App Service settings, Key Vault, or better, Managed Identity.
2. PaaS (App Service + Azure SQL) removes most of the infrastructure work compared with VMs.
3. Network rules (SQL firewall, NSGs) are closed by default and have to be opened on purpose.
4. Always clean up resources to avoid unexpected charges.
