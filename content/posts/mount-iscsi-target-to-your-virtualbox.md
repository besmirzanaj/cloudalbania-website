---
title: 'Mount iSCSI target to your Virtualbox VMs'
date: 2018-03-22T23:40:00.001-04:00
draft: false
url: /2018/03/mount-iscsi-target-to-your-virtualbox.html
---

Spent some time on this mounting an iSCSI LUN from a NetApp volume.  
In your NetApp SVM enter the new initiator with a default name according to the RFQ:  
  
iqn.2018-03.freebsd.com:freebsd  
  
  

[![](https://1.bp.blogspot.com/-V8dCitEozT4/WrR39arnbeI/AAAAAAAAPiE/uCDFBTe2IuQfvqicjkp5onmCDIQkomymACLcBGAs/s400/2018-03-22%2B23_41_19-Connected%2Bto%2BW2k12%2Bvia%2BRDP.png)](https://1.bp.blogspot.com/-V8dCitEozT4/WrR39arnbeI/AAAAAAAAPiE/uCDFBTe2IuQfvqicjkp5onmCDIQkomymACLcBGAs/s1600/2018-03-22%2B23_41_19-Connected%2Bto%2BW2k12%2Bvia%2BRDP.png)

  
  
After creating the netapp svm, lun, lif, etc you can mount the new iSCSI volume with the following command.  
  
PS C:\\Program Files\\Oracle\\VirtualBox>   
.\\VBoxManage.exe storageattach freebsd --storagectl "SATA" --port 0 --type hdd --medium iscsi --server 192.168.0.162 --target "iqn.1992-08.com.netapp:sn.944d91542e4811e8b5b800505600c301:vs.4" --tport 3260 --initiator "iqn.2018-03.freebsd.com:freebsd"  
  
Here is an screenshot from the Virtual Media Managet (Ctrl-D) in Virtualbox  

  

  

[![](https://1.bp.blogspot.com/-QdPdTZ0jFu0/WrR2SeB6MYI/AAAAAAAAPh0/_F7uLyc21PEUNSn72ymnwPMOco6kM0Z1QCLcBGAs/s400/2018-03-22%2B23_35_02-Connected%2Bto%2BW2k12%2Bvia%2BRDP.png)](https://1.bp.blogspot.com/-QdPdTZ0jFu0/WrR2SeB6MYI/AAAAAAAAPh0/_F7uLyc21PEUNSn72ymnwPMOco6kM0Z1QCLcBGAs/s1600/2018-03-22%2B23_35_02-Connected%2Bto%2BW2k12%2Bvia%2BRDP.png)

  
Now you can go on and install your preferred OS.  
  

[![](https://2.bp.blogspot.com/-cdG_e78TE9s/WrR3FbexIgI/AAAAAAAAPh8/A4fJkygXc8wn9TrTSXikxgPnqYSxWVUsgCLcBGAs/s400/2018-03-22%2B23_38_29-Connected%2Bto%2BW2k12%2Bvia%2BRDP.png)](https://2.bp.blogspot.com/-cdG_e78TE9s/WrR3FbexIgI/AAAAAAAAPh8/A4fJkygXc8wn9TrTSXikxgPnqYSxWVUsgCLcBGAs/s1600/2018-03-22%2B23_38_29-Connected%2Bto%2BW2k12%2Bvia%2BRDP.png)