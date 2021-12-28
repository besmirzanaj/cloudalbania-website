---
title: 'Get your public IP address from Ripe NCC'
date: 2017-06-18T17:54:00.005-04:00
draft: false
url: /2017/06/get-your-public-ip-address-from-ripe-ncc.html
---

If you are using PowerShell and in the console or the scripts you need to have your public IP address then Ripe NCC can really help you get this.  

[![](https://www.ripe.net/++resource++ripe.plonetheme.images/RIPE_NCC_logo.png)](https://www.ripe.net/++resource++ripe.plonetheme.images/RIPE_NCC_logo.png)

  
  
You can query RIPE stat servers and receive your IP address as JSON and save it in a variable to use it later. To get the JSON data just excecute this:  
  
C:\\Windows\\system32> $a = (Invoke-WebRequest -Uri "https://stat.ripe.net/data/whats-my-ip/data.json" | ConvertFrom-Json)   
  
and you should have this output:  
  
PS C:\\Windows\\system32> $a  
  
status           : ok  
server\_id        : stat-app15  
status\_code      : 200  
version          : 0.1  
cached           : False  
see\_also         : {}  
time             : 2017-06-18T21:49:27.221689  
messages         : {}  
data\_call\_status : supported  
process\_time     : 24  
build\_version    : 2017.6.15.213  
query\_id         : 000c524e-5470-11e7-8856-00505688b546  
data             : @{ip=123.123.123.123}  
  
If you just need the IP address then filter out only the IP address object like this:  
$ip\_address = (Invoke-WebRequest -Uri "https://stat.ripe.net/data/whats-my-ip/data.json" | ConvertFrom-Json).data.ip  
  
output:  
PS C:\\Windows\\system32> $ip\_address  
123.123.123.123