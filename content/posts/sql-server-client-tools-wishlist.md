+++ 
draft = true
date = 2024-09-20T10:01:43-04:00
title = "SSMS (and other client tools) Wishlist"
description = "List of client-side features I wish I had (or had more of) for interacting with SQL Server"
slug = "sql-server-client-tools-wishlist"
authors = ["Peter Vandivier"]
tags = ["sql-server-management-studio","ssms","sql-server","powershell","dbatools","azure-data-studio"]
categories = []
externalLink = ""
series = []
+++

Recently, [Erin Stellato has been asking](https://www.linkedin.com/feed/update/urn:li:activity:7240382832949215233/) for SSMS feedback on LinkedIn. This has me thinking about features for SSMS and other clients I'd like to have. This is gonna be a scratchpad doc for half-formed thoughts on that matter.

## SSMS

### Query Store - Multiple Dashboard Instances

When using Query Store dashboards - often I'll want to have multiple concurrent views of the same dashboard. E.g. - I'll want to browse different `query_id`s in the "Tracked Queries" dashboard at the same time; or I'll want to have the "Regressed Queries" dashboard open with both the default lookback (1 hour) as well as a longer lookback; or I may simply want to visualize the CPU Time & Duration regression critera side-by-side. 

In any case, you can only instantiate each dashboard once per Object Explorer session. This means I need to connect to the server by another connection string or open another running instance of SSMS to get the side-by-side view I want.

## PowerShell/dbatools

### Integrated "Recent Connections"

Much like SSMS, I'd like an integrated solution for dbatools to access recent/frequent connections. 

## VS Code/Azure Data Studio

TBH I need to force myself to use this more before I comment on what I think it needs.
