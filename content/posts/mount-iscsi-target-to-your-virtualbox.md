---
title: 'Mount iSCSI target to your Virtualbox VMs'
date: "2018-03-22"
draft: false
tags:
  - virtualbox
  - netapp
  - iscsi
---

Spent some time on this mounting an iSCSI LUN from a NetApp volume.  
In your NetApp SVM enter the new initiator with a default name according to the RFQ:  
  
iqn.2018-03.freebsd.com:freebsd  
  
![netapp_iscsi](/netapp_iscsi.png)


After creating the netapp svm, lun, lif, etc you can mount the new iSCSI volume with the following command.  
  
```powershell
PS C:\\Program Files\\Oracle\\VirtualBox>
.\VBoxManage.exe storageattach freebsd --storagectl "SATA" --port 0 --type hdd --medium iscsi --server 192.168.0.162 --target "iqn.1992-08.com.netapp:sn.944d91542e4811e8b5b800505600c301:vs.4" --tport 3260 --initiator "iqn.2018-03.freebsd.com:freebsd"  
```

Here is an screenshot from the Virtual Media Managet (Ctrl-D) in Virtualbox  

![media_manager](/media_manager.png)

Now you can go on and install your preferred OS.  

![iscsi_install_ok](/iscsi_install_ok.png)
