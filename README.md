# Deploy a Governed Web App to Azure App Service

**By Krishnadeep Chudasama | August 2026**

🔗 **Live app:** https://kchudasama13.azurewebsites.net/

## Overview

End-to-end deployment of a web app on **Azure App Service**, done entirely through **Azure Cloud Shell** (Bash + Azure CLI) — no local environment or portal clicking involved. The goal was to go beyond a basic "hello world" deploy and cover the full lifecycle a real workload needs: provisioning, zero-downtime deployment, governance controls, and monitoring.

![Resource group creation via Azure CLI](/01-resource-group.png)

## What I Built

- Provisioned a resource group and App Service plan via Azure CLI, and deployed the app to a live public URL
- Deployed custom HTML/Node content via ZIP deploy, replacing the default App Service landing page
- Configured **deployment slots** to perform a zero-downtime (blue-green) deployment
- Applied **governance controls** — resource tags and a `CanNotDelete` resource lock — to protect the resource group from accidental deletion
- Set up monitoring using **Azure Advisor**, **Service Health**, and **Azure Monitor** metrics
- Got hands-on with the **PaaS** model — Azure handled the server, runtime, and scaling so I could focus purely on application code

![Live web app running on Azure App Service](/02-live-app.png)

## Tools & Concepts Used

| Category | Details |
|---|---|
| **Tools** | Azure Cloud Shell, Azure CLI, PM2 process manager, ZIP deploy, Kudu, Azure Monitor Metrics |
| **Concepts** | PaaS vs. IaaS, resource group architecture, deployment slots & blue-green deployments, resource locks & tags, Azure monitoring tools, telemetry propagation delay |

## Governance: Tags & Resource Locks

To protect the resource group from accidental deletion, I applied a `CanNotDelete` lock via `az group lock create`. When I then attempted `az group delete`, Azure correctly blocked the operation:

![Resource lock blocking a delete operation](/04-resource-lock-error.png)

```
(ScopeLocked) The scope '.../resourceGroups/az900-webapp-rg' cannot perform
delete operation because following scope(s) are locked. Please remove the
lock and try again.
```

## Zero-Downtime Deployment with Slots

I learned how deployment slots let you stage a new version of an app in an isolated environment (Standard S1 tier) and swap it into production instantly, with no downtime for users — the same blue-green pattern used in production systems.

![Verifying the deployed content after slot swap](/05-slot-swap.png)

**Key takeaways from this section:**
- **PaaS in practice:** deploying directly to App Service showed how Azure abstracts away server setup, scaling, and runtime configuration
- **Resource restrictions:** pricing tiers gate features — deployment slots require Standard S1, and scaling back down to Free F1 requires deleting the slots first

## Monitoring

Explored three complementary Azure monitoring tools to close the loop on management and governance:

- **Azure Advisor** — recommendations across five categories: cost, security, reliability, operational excellence, and performance
- **Azure Service Health** — surfaces platform-level outages that Microsoft, not the customer, is responsible for fixing
- **Azure Monitor Metrics** — configured live metrics (e.g. Average Memory Working Set, HTTP 2xx) for the running `kchudasama13` web app

![Azure Monitor metrics for the web app](/03-monitoring.png)

## Issues Hit & How I Resolved Them

| Issue | Cause | Fix |
|---|---|---|
| Regional quota limit | 0 VM capacity available in East US | Redeployed to Central US |
| Missing Linux runtime stack error | Runtime wasn't explicitly specified | Added an explicit `--runtime` parameter |
| Unsupported Node version string | Used an invalid version identifier | Ran `az webapp list-runtimes` to find the correct supported string |
| `ResourceNotFound` during ZIP deploy | Referenced an incorrect resource name | Verified the actual resource name via `az webapp list` |
| Deployed code not showing (default page persisted) | Static content wasn't being served correctly | Fixed with a custom PM2 startup command for static hosting |
| Deletion blocked by resource lock | `CanNotDelete` lock was still active | Temporarily removed the lock, completed the scale-down, then reapplied it |
| Blocked scale-down (maximum slots exceeded) | Free F1 tier doesn't support deployment slots | Deleted the slot before downgrading the pricing tier |

## Reflections

This project took about **2 days** end-to-end. The most challenging parts were largely around Azure's guardrails — regional capacity limits, runtime version mismatches, and pricing-tier restrictions — rather than the app code itself, which was the point: the goal was to build fluency with Azure's CLI, governance, and monitoring surface, not to build a complex app.

**Next up:** deploying and managing Azure VMs, and exploring more of Azure's resource landscape beyond PaaS.
