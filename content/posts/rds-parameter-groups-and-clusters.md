+++ 
draft = false
date = 2025-07-29T14:12:51-04:00
title = "Headdesking RDS Parameter Groups"
description = "Installing a postgres extension shouldn't be this hard"
slug = "rds-parameter-groups-and-clusters"
authors = ["Peter Vandivier"]
tags = ["rds", "awscli", "aurora"]
categories = []
externalLink = ""
series = []
+++

# Love/Hate Development

I really love the PostgreSQL extensions ecosystem.

I really don't love RDS.

It's not that it's a bad product; I'm just not the target audience. I have [expressed this displeasure](https://bsky.app/profile/petervandivier.bsky.social/post/3lerobebovs23) previously; it's just not a tool for people who like working with database internals.

Recently I went about trying to enable pg_cron on an Aurora PostgreSQL cluster. Fortunately there's [docs](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/PostgreSQL_pg_cron.html)! Unfortunately I got to remember all of the annoying ephemera I perennially encounter in RDS.

# Thing 1: Unclear Hierarchy

An Aurora PostgreSQL cluster has management nodes for both the cluster and the bound instances. Each of these nodes has separate parameter groups (as opposed to [option groups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithOptionGroups.html) - different thing). What's more - option groups come in two classes which don't inter-operate. [DB Parameter Groups](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/USER_WorkingWithParamGroups.html) and [DB Cluster Parameter Groups](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/USER_WorkingWithDBClusterParamGroups.html) have separate API calls with separate inputs and it's one of those things that isn't obvious unless you already know what you're working with. The discoverability is not great.

Additionally - for kicks & giggles I tried enabling pg_cron in the Cluster _but not_ the instance group. I didn't expect it to work but it did. I'm sure there's an elegant inheritance mechanism that can be explained by someone who _already understands it_ but it hammers against that same anvil of murky discoverability.

# Thing 2: `default.` Groups Can Bite Me

AWS does a lot of handling things behind the scenes for you - it's one of the main reasons why people love it. But the `default.` groups created for RDS have "special" rules which - again - are not obvious to the newcomer and have some very annoying "gotchas". Chiefly...

1. You can't edit the default group
2. You can't copy the default group

HT to [Calvin Torra](https://calvintorra.com/blog/aws/unable-to-copy-default-rds-parameter-group-in-aws/) for blogging about this because I thought I was losing my mind when the copy commands kept bombing out. One thing he doesn't cover though is that as a user you are trusting that the "new" parameters match what's in the default group. I didn't trust that to be true so I ended up dumping both groups and diffing the serialized parameters to satisfy my OCD before proceeding.

# Thing 3: `awscli` Is Great Until it's Not

I'm a PowerShell stan. Unfortunately PowerShell has always been a bit of a second-class citizen to it's parent `awscli`. I'll typically end up rubber-ducking the AWSPowerShell docs for a while before giving up and dropping back to the native API in frustration. [This time was no exception](https://discord.com/channels/180528040881815552/447472663187685377/1399477058945417479). Unfortunately, even in the native API, there's some misleading error codes that hide a lot of what I ended up discovering in the past 24 hours. The edit & copy commands return naming violation errors rather than making clear the `default.` objects are untouchable. The same naming violation error also pops when you typo a group name - which... sucks...

Additionally, the parameter values of any given group are not bound to the Group object. This was merely unintuitive until it became annoying that I then had to make multiple additional calls to do the aforementioned diffing. Small typos in modifying my scripting had me quite frustrated by the time I was satisfied.

# Final Thoughts

I still figured it out in an afternoon but it's like pulling teeth. Somehow RDS has made the `CREATE EXTENSION` process complicated. Like all things AWS though - once you _already know_ how to do it, it gets easier; but it's all kinds of troublesome getting there in the first place. 
