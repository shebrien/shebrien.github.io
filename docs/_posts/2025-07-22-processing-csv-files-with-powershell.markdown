---
layout: post
title:  "Processing CSV files using PowerShell"
date:   2025-07-22 23:46:31 +0300
categories: powershell
---
In this post we'll cover two quality-of-life improvements when processing data, one for big data and the other for time stamps.
You may have gotten used to opening CSV files in Excel to do some manual inspection or extra calculations, but noticed when opening these CSVs you sometimes get unexpected behaviors. For example a line of CSV suddenly breaks down to multiple lines with unaligned columns, or when adding a filter to a column that contains what you know to be dates Excel does not treat it as such. This isn't necessarily a defect in the CSV but more of a side effect when choosing to open CSVs using Excel. In the case of a single row becoming multiple rows of mostly empty columns, it is technically an expected behavior since individual cells in Excel by design have a limit on how much data they hold (exactly 32KiB minus 1 byte) so when a CSV element exceeds that amount then Excel does that behavior. How you resolve this entirely depends on your own needs. If you must preserve this data then you need to find tools other than Excel to perform your CSV processing. Otherwise if you can afford to change this data around then you could use PowerShell to exclude the entire column, the entire row, or just the individual "cells" containing the large data.

I'll be writing a sample script to pinpoint where the problematic cells are within a given CSV file.
{% highlight powershell %}
$ExcelCellLimit = 32kb - 1
$CSVFile = Import-CSV "C:\Path\To\File.csv"
$CSVHeaders = gm -in $CSVFile[0] | ? membertype -eq NoteProperty | select -expand name #Get CSV file headers from a sample row

# Since CSVs can be imagined as 2D arrays, we'll iterate by doing two loops
$currentLine=1
foreach ($i in $CSVFile){
	foreach($j in $CSVHeaders){
		$length = $i.$j.length # This is how to dynamically access object properties in PowerShell, by assigning a property name to a variable then using this notation.
		if($length -gt $ExcelCellLimit){
			# For this sample script we'll simply report our findings
			write-host ("row {0} column {1} has {2} chars" -f ($currentLine,$j,$length)) # This way of making strings with -f is called string formatting
		}
	}
	$currentLine++
}
{% endhighlight %}

Keep in mind that whatever processing method you choose to do in PowerShell, when saving the results back using Export-CSV remember to account for file encodings as this cmdlet by default uses ASCII. With that we can now talk about the other scenario which is the other behavior that involves misunderstood dates.

From my past experiences the way that Excel decides how to auto-process columns as dates depends on what your Windows's date format in Region settings is whenever you launch (or relaunch) the first instance of the Excel application, so if the difference is small enough to be handled by changing Windows settings you can close Excel and do just that. If however the date format is more complicated than that then we can do some data processing using .NET's DateTime data type.

This time our scenario is our CSV file is already open in Excel and everything is fine besides the column containing dates (or a full time stamp that we only need that date part of), and we'd rather fix it quickly instead of interrupting our flow with writing a full CSV processing script. We can do such ad-hoc processing by making use of the clipboard and the Get-Clipboard & Set-clipboard cmdlets (aliases gcb and scb respectively). We'll also try to make it a one-liner with aliases instead of full cmdlet names since it is a task that came up on the fly, but first we must determine the exact way of processing these date strings. We'll optimistic first and take a sample date string from the CSV column and see if .NET DateTime parser can convert it. To avoid making this post too long we'll take the happy route and pick a sample that natively converts. The bad route will mean you'll need to do some string manipulations using regular expressions (I should make a future post about that topic!) then sending the results back via the clipboard.

{% highlight powershell %}
# A sample test of converting a string containing a date, a time, and a time zone to a datetime
[datetime]"2025-07-22T16:52:47+0300"
{% endhighlight %}

With the test being a success we can proceed to doing a one-liner that will convert the timestamps to a short date.

{% highlight powershell %}
# first select the cells containing the dates, press Ctrl-C, then run the below
gcb | %{if($_.length -gt 0){([datetime]$_).ToShortDateString()}else{$_}} | scb
{% endhighlight %}

Keep in mind that clipboard interactions involving Excel and PowerShell can be unreliable, the best case scenario is: When you press Ctrl-C the cells get highlighted as expected, then when you switch to the PowerShell window and run the one-liner you will see the highlighting disappear from the cells, indicating a successful overwrite of clipboard contents with your results, then you go back to Excel and paste as usual. If the highlight doesn't disappear you might need to re-run the one-liner, or paste into an application other than Excel such as a notepad.

That's all for today, and see you next time!
