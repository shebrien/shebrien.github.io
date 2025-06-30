---
layout: post
title:  "Let me sell you PowerShell!"
date:   2025-06-21 00:12:21 +0300
categories: powershell standalone
---
Just like how Excel is to data analysis, so is PowerShell to system administration: a ubiquitous tool with limitless potential, for better or worse!

A scripting language at its core, PowerShell gives sysadmins the ability to inquire, control, and manage systems in ways that sometimes rival readymade programs if used by an experienced sysadmin. Need to double check live disk usage of multiple systems? Or list all VMs that raised a specific error during last night's incident? Maybe even get the mapping between drive letters and WWNs of physical disks zoned to your Windows servers? PowerShell has you covered.

While learning the syntax is important, its real value comes from both the plethora of capabilities that it gets from Modules (custom libraries made by Microsoft or third parties that expose new functions) and its deep integration with .NET classes. Both of these make for an expansive tool chain that can address many of the urgent needs that may come during your work.

Another feature worth mentioning is pipelining, which is taking the output of one command and making it the input of the next one, effectively chaining commands into a pipeline to reach a desired result. While this concept does exist in other scripting tools such as bash, the main difference is that these other tools output pure text thus making further processing limited to direct text manipulation, whereas in PowerShell the output tends to be actual "objects" that give way more options when it comes to manipulating or displaying the results.

All of these PowerShell benefits do come at a cost though. If you think how there exists some Excel files that are handling very complex scenarios to the point where the file is essentially filling the role of a database minus the actual benefits of using a fully fledged database engine, then in a similar vein you can very much end up knowing enough PowerShell to use it in ways that may end up "working" but it won't scale as the workloads grow with your business. An example of PowerShell gone wrong would be attempting to scan gigabytes of log files containing millions of lines to find occurrences of a specific text using by building a pipeline of commands (get file contents -> match specific pattern -> output result to file). Pipelining in this case will be very slow compared to other approaches that perform better I/O, and while it's true you could force PowerShell to use those alternative approaches if have the know-how and get good results, it still holds that after a certain complexity threshold you're better off using compiled programming languages.

PowerShell truly shines in automating any mundane system admin work such user onboarding/offboarding, simplifying a multiple step procedure into a single command that does the steps, and finding data that would otherwise take a long time if a GUI tool was used.

Learning PowerShell (and scripting languages in general) will ensure you stand out in IT.
