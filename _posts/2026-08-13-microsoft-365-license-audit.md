layout: post
title: "Why Your Company Is Probably Overpaying for Microsoft 365"
date: 2026-08-13
---

Most orgs never audit their Microsoft 365 licensing after initial rollout, and the drift adds up fast. A proper audit usually targets five cost levers: disabled accounts still holding licenses, dormant accounts with no recent sign-in activity, over-provisioned users on plans heavier than their actual usage, unassigned seats still being billed at the tenant level, and suboptimal billing terms under Microsoft's New Commerce Experience (NCE) pricing model.

**Disabled but licensed accounts** are the easiest win. Query them via the admin center by filtering Active Users on Licensed + Block credential = false, or pull the same data with Microsoft Graph PowerShell:

```powershell
$users = Get-MgUser -All -Property Id,DisplayName,UserPrincipalName,AccountEnabled,AssignedLicenses
$users | Where-Object { $_.AccountEnabled -eq $false -and $_.AssignedLicenses.Count -gt 0 }
```

Before pulling the license, convert the mailbox to shared and apply a retention policy — don't just yank access.

**Inactive accounts** need `AuditLog.Read.All` scope and the `SignInActivity` property, which only surfaces cleanly if your tenant has Entra ID P1. Without P1, sign-in timestamps aren't exposed the same way and you're stuck cross-referencing usage reports manually or leaning on a third-party tool that can pull the data out-of-band.

**Over-licensing** is the hardest to automate — Microsoft doesn't expose a unified usage report across every app/service a user touches (Planner, Forms, Viva, Teams webinars, etc. don't show up in the standard usage exports), so downgrading based on email activity alone risks breaking something a user depends on silently. This is the one step where I'd actually loop in someone who's done it before rather than scripting it blind.

**Unassigned seats** are pure billing hygiene — Licenses > Available Licenses in the admin center, or via Graph:

```powershell
$skus = Get-MgSubscribedSku -All
$skus | select SkuPartNumber, @{N="Total";E={$_.PrepaidUnits.Enabled}}, @{N="Assigned";E={$_.ConsumedUnits}}, @{N="Available";E={$_.PrepaidUnits.Enabled - $_.ConsumedUnits}}
```

Watch for false positives here — licenses assigned through third-party apps or consumption-based SKUs (like extra SharePoint storage) can show as "available" when they're not actually free to cut.

**Billing terms** are the most overlooked lever. NCE pricing charges a premium for monthly billing over annual commitment — often 12-16%+ — and you can mix terms within the same SKU pool across your tenant rather than committing everyone to the same cycle.

I pulled most of this from [Gitbit's Microsoft 365 license audit guide](https://www.gitbit.org/docs/microsoft-365-license-audit?utm_campaign=audit-drive&utm_medium=referral&utm_source=malikdeezy), which goes deeper into the exact PowerShell prerequisites and admin center click-paths if you want to run this end-to-end on your own tenant.



