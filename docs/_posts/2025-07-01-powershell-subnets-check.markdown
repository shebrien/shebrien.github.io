---
layout: post
title:  "Checking Which Subnet an IP Belongs to in PowerShell"
date:   2025-07-01 01:00:00 +0300
categories: powershell
---
Today we'll cover an intermediate PowerShell topic.

In an enterprise IPv4 network, planning subnetting is essential for a corporate environment, but it can lead to scenarios for operation teams where confirming which network does an IP belong to becoming a non-trivial task.

Say for example your network team decided to split a 10.0.0.0/8 subnet into 8 subnets with prefix length of 11, meaning you'd get 10.0.0.0/11, then 10.32.0.0/11 , up to the 8th network of 10.224.0.0/11. Now if for example you receive the IP/subnet pair 10.95.123.65/255.224.0.0, how would you know which subnet it belongs to? For a single example you could simply use an online CIDR tool, or even manually do the math if you feel inclined, but what if you receive a list of hundreds of such pairs?

In such a scenario, you either need to find a tool that does this job, or you could write your own PowerShell script! Starting from around PowerShell 7 (which needs manual installation if not already present), you get access to even more .NET classes, including System.Net.IPNetwork and System.Net.IPAddress. We'll be making use of those in the sample script below that'll solve our problem.

{% highlight powershell %}
# In most scenarios you'd load the list of subnets from a file or another source, in this example we mock it instead
$SNs = 0..7 | %{[System.Net.IPNetwork]"10.$(32*$_).0.0/11"}

# Generating a random set of 10 IPs that follow the same pattern as our mock subnets
$ips= 1..10 | %{[IPAddress]"10.$(get-random -Maximum 256).$(get-random -Maximum 256).$(get-random -Maximum 256 -Minimum 1)"}

# Loop through the IPs
foreach ($i in $ips){
	# Get which subnet it belongs to by using the contains method of each subnet
	$sn = $SNs | ? {$_.contains($i)}
	write-host "$i belongs to $sn"
}
{% endhighlight %}

Running the above script will show which subnet does our randomly generated IPs belong to.

Another thing we can do with these classes is do lower level calculations, such as determining the network address and broadcast address of each subnet. Admittedly most networks are subnetted in a way that makes knowing these values trivial (last octect is 0 for network and 255 for broadcast). However, if you do run into a case where these addresses are different (e.g. subnets with less than 256 IPs) you will need to use external tools, OR you could do the binary calculations directly in PowerShell! In the below example you'll find two IPs that at first glance seem to be part of the same subnet since they share the same subnet mask, but because this mask is very small they actually belong to two different subnets.
{% highlight powershell %}
$ip1=[ipaddress]'10.224.55.130'
$ip2=[ipaddress]'10.224.55.87'
$snm=[ipaddress]'255.255.255.128'
write-host "Network address of $ip1 :" ([ipaddress](([uint32]$ip1.Address -band [uint32]$snm.Address))).ToString()
write-host "Broadcast address of $ip1 :" ([ipaddress]([uint32]$ip1.Address -bor -bnot [uint32]$snm.Address)).ToString()

write-host "Network address of $ip2 :" ([ipaddress](([uint32]$ip2.Address -band [uint32]$snm.Address))).ToString()
write-host "Broadcast address of $ip2 :" ([ipaddress]([uint32]$ip2.Address -bor -bnot [uint32]$snm.Address)).ToString()
{% endhighlight %}
