---
layout: post
title:  "Joining lists in PowerShell"
date:   2025-07-12 21:00:15 +0300
categories: powershell
---
This time we'll talk about an advanced topic in PowerShell, and arguably a demonstration of it being the wrong tool for the job but nevertheless it helps in a pinch!

The example scenario is simple enough, given two lists where one contains employee names and IDs, and the other contains IDs and yearly performance evaluations, you are tasked with correlating employee names with their evaluations. 

The trivial solution is to not use PowerShell as this is a simple SQL join question, you just need some time to create appropriate tables, load the data, then perform the join query. But, what if you get such requests frequently and most of the time the data types (and therefore the table column types) are different? You'll waste time preparing different data models to answer repetitive questions.

We could try attempting to solve this using PowerShell. To simplify implementation and to focus on the point we're trying to demonstrate, our scenario will have both lists be 10000 entries long with 1:1 cardinality (each row in Names matches exactly 1 row in Evals) and both lists are ordered by ID. The simplest way is to solve this problem is to mimic join operations by doing basic filtering through pipelines as shown below.

{% highlight powershell %}
# Let's prepare some sample data, instead of coming up with 10000 actual names we'll make random GUIDs.
Write-Host "Generating Data"
Measure-Command {
$Names = 1..10000 | %{[pscustomobject]@{ID=$_;Name=[guid]::NewGuid()}}
$Evals = 1..10000 | %{[pscustomobject]@{ID=$_;Eval=Get-Random -Min 1 -Max 100}}
}
Write-Host "Performing Joins using a pipeline of ForEach-Object (%) and Where-Object (?)"
Measure-Command { $Join = $Names | %{$Current = $_;$Matched = $Evals | ? ID -eq $Current.ID;[pscustomobject]@{Name=$Current.Name;Eval=$Matched.Eval}} }
{% endhighlight %}

For those of you who have previous experience in PowerShell, you might already know that the above method to "join" is the slowest because it purely uses the pipelines feature. All solution approaches in this article are surrounded by Measure-Command to show us how long it took to finish executing the joins. The above sample took 15 minutes to finish "joining" our 10k Names with our 10k Evals on a regular PC.

We can do better. Our next approach is to use the Foreach statement to perform our joins.

{% highlight powershell %}
Write-Host "Performing Joins using Foreach loops"
Measure-Command { $Join2 = Foreach($Current in $Names) {$Matched = Foreach ($i in $Evals) { if ($i.ID -eq $Current.ID){$i}}
[pscustomobject]@{Name=$Current.Name;Eval=$Matched.Eval}} }
{% endhighlight %}

This time the above sample took 4 minutes and 7 seconds, a very noticeable improvement! But stopping here means we haven't talked anything "advanced" as the article promised, so let's go deeper!

If you are a seasoned C# developer, you might have looked at these numbers and thought "I can perform these joins in an instant using LINQ and delegates!" and you'd be correct, which should serve as a good reminder to always consider using the right tool for any given task. Still, since both C# and PowerShell have a very close relation with the .NET ecosystem, is there a way to somehow use LINQ in PowerShell ignoring any trade-offs? The answer dear reader is a resounding yes!

Going into the details of how LINQ and delegates work is beyond the scope of this article so it'll be left as an exercise for the reader, but we'll be using them in the below sample to do what a C# developer would do to implement a join (though admittedly, LINQ is so integrated with C# syntax that you can port the below sample to C# with much fewer lines of codes and statements, and even have better readability).

{% highlight powershell %}
# This custom function should serve as a starting point for doing LINQ in PowerShell, you may need to adapt it to your own specific scenario especially if join attributes are not integers.
Function Do-LINQJoin ($FirstInput,$SecondInput,$FirstKey,$SecondKey){
	$FirstKeyDelegate = iex ("[Func[System.Object,$($FirstKey.gettype().name)]] {`$args[0].$FirstKey }")
	$SecondKeyDelegate = iex ("[Func[System.Object,$($SecondKey.gettype().name)]] {`$args[0].$SecondKey }")
	$InnerJoinDelegate = [Func[System.Object,System.Object,PSCustomObject]] {
		[PSCustomObject]@{
			FirstInput = $args[0]
			SecondInput = $args[1]
		}
	}
	[Linq.Enumerable]::Join($FirstInput,$SecondInput,$FirstKeyDelegate,$SecondKeyDelegate,$InnerJoinDelegate)
}
Write-Host "Performing Joins using .NET LINQ"
Measure-Command { $Join3i = Do-LINQJoin $Names $Evals 'ID' 'ID' }
Write-Host "Generating results for .NET LINQ"
Measure-Command {
	$Join3 = foreach($i in $Join3i){[pscustomobject]@{Name=$i.FirstInput.Name;Eval=$i.SecondInput.Eval}}
}
{% endhighlight %}

In the above sample, performing both the "join" procedure between two lists of 10000 items each and generating final objects from that join took us a total of... less than 1 second! To finalize this article let's compare results for correctness, which should be easy based on how we constructed our sample data.

{% highlight powershell %}
# Take 10 random indices and check all our relevant objects at the same index position so that we can manually check if results were correct or not, after each random index we print a blue line for readability
1..10 | %{$c=get-random -min 0 -max 9999;$Names[$c] | out-host;$Evals[$c]| out-host;$Join[$c]| out-host;$Join2[$c]| out-host;$Join3[$c]| out-host;write-host -back DarkCyan ("="*30)}
{% endhighlight %}

That's all for now, and remember to explore more PowerShell and .NET to expand your toolset, but keep in mind it isn't always the appropriate tool for all scenarios.
