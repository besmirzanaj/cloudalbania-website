---
title: 'Set up a Powershell dev environment'
date: 2017-10-06T09:39:00.000-04:00
draft: false
url: /2017/10/set-up-powershell-dev-environment.html
tags: 
- tools
- script
- powershell
- git
---

For us sysadmins it is important to create scripts on the fly and make things work. By working in an ad-hoc way, our scripts will get lost, not cured and especially not searchable.  
  

### Initial tools setup

To better organize our code some personal best practices follow:  
Environment: Windows  
Install and download [TortoiseGit](https://tortoisegit.org/download/) and of course [GIT](https://git-scm.com/download/win). While installing GIT just select the defaults for you.  

[![](https://2.bp.blogspot.com/-ebiWQmZTV-w/WdeEWq_bYJI/AAAAAAAAN3U/8SjkYFS_BXwk8RRZP8ZGtnQ3Qxblw0QgACLcBGAs/s320/Capture.PNG)](https://2.bp.blogspot.com/-ebiWQmZTV-w/WdeEWq_bYJI/AAAAAAAAN3U/8SjkYFS_BXwk8RRZP8ZGtnQ3Qxblw0QgACLcBGAs/s1600/Capture.PNG)

  

### Saving code

For every single script it is a better idea to integrate or put all its relative files in a separate folder. Then put all these folders in a single one named: scripts.  
  
After installing TortoiseGIT it is a good idea to go with the First Start wizard.  
Select the defaults and in the "configure user information" screen input your name and a valid email address.  

[![](https://4.bp.blogspot.com/-eVTSnlNKvjc/WdeEfln90OI/AAAAAAAAN3Y/HX8QrduSUU0A3zKReafaeavKtx7dzPnuQCLcBGAs/s320/3.PNG)](https://4.bp.blogspot.com/-eVTSnlNKvjc/WdeEfln90OI/AAAAAAAAN3Y/HX8QrduSUU0A3zKReafaeavKtx7dzPnuQCLcBGAs/s1600/3.PNG)

  
  
The next thing to do is to integrate all your scripts in a GIT repo. With the help of TortoiseGIT this can be easily achieved with only the right click of the mouse  

[![](https://4.bp.blogspot.com/-Q0kz6gUc1RU/WdeFVKt2MLI/AAAAAAAAN3g/2j3tS8bxgoQhhB7Xa55ah3dH7uLFLdTIQCLcBGAs/s320/2017-10-06%2B09_28_59-Greenshot.png)](https://4.bp.blogspot.com/-Q0kz6gUc1RU/WdeFVKt2MLI/AAAAAAAAN3g/2j3tS8bxgoQhhB7Xa55ah3dH7uLFLdTIQCLcBGAs/s1600/2017-10-06%2B09_28_59-Greenshot.png)

  
And you are done with the repository creation.  

[![](https://2.bp.blogspot.com/-SbqdXUmFtfo/WdeFolMrn4I/AAAAAAAAN3k/41nRK0qzLxg09uoP8SBMARKPih5nuYxWQCLcBGAs/s320/2017-10-06%2B09_30_58-TortoiseGit.png)](https://2.bp.blogspot.com/-SbqdXUmFtfo/WdeFolMrn4I/AAAAAAAAN3k/41nRK0qzLxg09uoP8SBMARKPih5nuYxWQCLcBGAs/s1600/2017-10-06%2B09_30_58-TortoiseGit.png)

  
After each working sessions it is a good idea to commit your work. With TortoiseGIT again this can be easily done by a mouse right-click.  

[![](https://4.bp.blogspot.com/-HdgLzDuXauc/WdeGq41Os8I/AAAAAAAAN3w/HZA3o1zlX0g3QKWrKye8mQ5aw5j_XDWHwCLcBGAs/s320/2017-10-06%2B09_33_00-Blogger_%2BCloud%2BAlbania%2B-%2BEdit%2Bpost.png)](https://4.bp.blogspot.com/-HdgLzDuXauc/WdeGq41Os8I/AAAAAAAAN3w/HZA3o1zlX0g3QKWrKye8mQ5aw5j_XDWHwCLcBGAs/s1600/2017-10-06%2B09_33_00-Blogger_%2BCloud%2BAlbania%2B-%2BEdit%2Bpost.png)

  
Then fill out the commit screen with an appropriate message and then click commit  

[![](https://3.bp.blogspot.com/-6YriNVUL-S4/WdeGxeBQncI/AAAAAAAAN30/v9NN6_jNbAgVJzD8NjGBRRs3AQqfmvgLACLcBGAs/s320/2017-10-06%2B09_33_44-C__scripts%2B-%2BCommit%2B-%2BTortoiseGit.png)](https://3.bp.blogspot.com/-6YriNVUL-S4/WdeGxeBQncI/AAAAAAAAN30/v9NN6_jNbAgVJzD8NjGBRRs3AQqfmvgLACLcBGAs/s1600/2017-10-06%2B09_33_44-C__scripts%2B-%2BCommit%2B-%2BTortoiseGit.png)

  
The commit confirmation screen will show up.  
  

[![](https://2.bp.blogspot.com/-cCFdcIHSleY/WdeG-zuV_wI/AAAAAAAAN34/SUulSeL5Uycc3MQUfb32TLVqfXKoOTopwCLcBGAs/s320/2017-10-06%2B09_34_49-C__scripts%2B-%2BCommit%2B-%2BTortoiseGit.png)](https://2.bp.blogspot.com/-cCFdcIHSleY/WdeG-zuV_wI/AAAAAAAAN34/SUulSeL5Uycc3MQUfb32TLVqfXKoOTopwCLcBGAs/s1600/2017-10-06%2B09_34_49-C__scripts%2B-%2BCommit%2B-%2BTortoiseGit.png)

  
Do the same steps (commit) after each coding session to save your work and your coding history.  
  

### Pushing code remotely

After working locally, you might consider storing your code in a server repo. A good one is at [GitLab](https://gitlab.com/users/sign_in) which offers private repos for free.