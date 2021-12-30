---
title: 'Get your public IP address from Ripe NCC in Powershell'
date: 2017-06-18T17:54:00.005-04:00
draft: false
#url: /2017/06/get-your-public-ip-address-from-ripe-ncc.html
---

If you are using PowerShell and in the console or the scripts you need to have your public IP address, then Ripe NCC can really help you get this.  

[![](https://www.ripe.net/++resource++ripe.plonetheme.images/RIPE_NCC_logo.png)](https://www.ripe.net/++resource++ripe.plonetheme.images/RIPE_NCC_logo.png)

You can query RIPE stat servers and receive your IP address as JSON and save it in a variable to use it later. To get the JSON data just excecute this:  

    C:\Windows\system32> $a = (Invoke-WebRequest -Uri "https://stat.ripe.net/data/whats-my-ip/data.json" | ConvertFrom-Json)

  
and you should have this output:  
  
```cmd
PS C:\Windows\system32> $a  
  
status            : ok  
server_id         : stat-app15  
status_code       : 200  
version           : 0.1  
cached            : False  
see_also          : {}  
time              : 2017-06-18T21:49:27.221689  
messages          : {}  
data_call_status  : supported  
process_time      : 24  
build_version     : 2017.6.15.213  
query_id          : 000c524e-5470-11e7-8856-00505688b546  
data              : @{ip=123.123.123.123}  
```

If you just need the IP address then filter out only the IP address object like this:  

    $ip_address = (Invoke-WebRequest -Uri "https://stat.ripe.net/data/whats-my-ip/data.json" | ConvertFrom-Json).data.ip

output:

    PS C:\Windows\system32> $ip_address  
    123.123.123.123

