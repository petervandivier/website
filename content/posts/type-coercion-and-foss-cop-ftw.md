+++ 
draft = false
date = 2025-09-26T11:41:06-05:00
title = "Intuitive Type Coercion and FOSS COP Supremacy"
description = ""
slug = "type-coercion-and-foss-cop-ftw"
authors = ["Peter Vandivier"]
tags = ["powershell","open-source","community-of-practice"]
categories = []
externalLink = ""
series = []
+++

> [PeterVandivier — 9/24/25, 12:31 PM](https://discord.com/channels/180528040881815552/447476117629304853/1420447481560432722)
> 
> _why does_ `@(1,2)[$null]` _fail where_ `@(1,2)[[int]$null]` _succeeds?_

A PowerShell behavioural quick caught my attention debugging an little for-loop and prompted me to ask the above in PowerShell discord.

Usually, `$null` coerces intuitively in PowerShell... 

```pwsh
> "Hello, $null." # empty string
# Hello, .

> 1 + $null # zero
# 1
```

...and provides a helpful message if it doesn't...

```pwsh
> [timespan]$null                        
InvalidArgument: Cannot convert null to type "System.TimeSpan".
```

> _Why then_ ([I asked](https://discord.com/channels/180528040881815552/447476117629304853/1420450888589377692)...) _is explicit typing needed in the index operator?_

This is a throwaway question. I don't _need_ to know the answer. It's one of the ten thousand language quirks you pick up on using the same tools for over a decade. You type these things into Twitter or Reddit or Quora or IRC as much in the hope of getting an answer as to just complain about it. 

![denvercoder9 knows what I'm talking about](https://imgs.xkcd.com/comics/wisdom_of_the_ancients.png "what did they see?")

...but something changed in the past decade... [PowerShell went open source](https://azure.microsoft.com/en-us/blog/powershell-is-open-sourced-and-is-available-on-linux/). _You can look at the code now_.

Now to be clear - FOSS software isn't a documentation panacea. You still need people who know the code base and have a working knowledge of it to parse your question. Why only very recently I asked [someone to write me a grep statement](https://dba.stackexchange.com/a/347778/68127) to get an answer out of a FOSS codebase I had _already been digging in_.

I go to the PowerShell discord precisely _because_ it's chockablock full of people who know the language and the code base and where to look. This is what it means to be in a [community of practice](https://en.wikipedia.org/wiki/Community_of_practice) and it's deeply facilitated by FOSS principles.

I love deeply that I not only _got an answer in minutes_... but that it was given with _receipts_. That is - a link to the actual code comments in source.

Some times we do get to have nice things.

----

...oh yea... the reason for the language thing is "By Design" 🤪...

> [seeminglyscience — 9/24/25, 1:07 PM](https://discord.com/channels/180528040881815552/447476117629304853/1420456619317268612)
> 
> _it's assumed to be significantly more likely to be a design time error rather than the author genuinely wanting_ `0`<br>
> _basically the get index binder itself explicitly throws when you pass null as an index for an array_
> 
> https://github.com/PowerShell/PowerShell/blob/d8b1cc55332079d2be94cc266891c85e57d88c55/src/System.Management.Automation/engine/runtime/Binding/Binders.cs#L4028-L4038 _relevant code with comments_

```cs
// A null index is not allowed unless the index is one of the indices used while slicing, in which case we'll attempt
// the usual conversions from null to whatever the value being indexed supports.
// This is oddly inconsistent e.g.:
//     $a[$null] # error
//     $a[$null,$null] # no error, result is an empty array
// The rationale: V1/V2 did it, and when people are slicing, it's better to return some of the results than none.
if (indexes.Length == 1 && indexes[0].Value == null && _allowSlicing)
{
    return (errorSuggestion ??
            target.ThrowRuntimeError(indexes, BindingRestrictions.Empty, "NullArrayIndex", ParserStrings.NullArrayIndex)).WriteToDebugLog(this);
}
```

...which... come on... I love that you get to read the authors' thoughts on the matter too...

<br><br><br><br><br><br>

_so cool_
