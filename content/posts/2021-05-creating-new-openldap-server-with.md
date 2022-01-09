---
title: 'Creating a new LDAP server with FreeIPA and configure to allow vSphere authentication'
date: "2021-05-28"
draft: false
#url: /2021/05/creating-new-openldap-server-with.html
tags: 
- vmware
- freeipa
- vsphere
- ldap
- ansible
- openldap
---

Was setting up a new FreeIPA sever for my homelab and found out that the default configuration in FreeIPA does not allow you to use VMware vSphere as a client as not being fully [RFC4519](https://datatracker.ietf.org/doc/html/rfc4519) and missing some other LDAP class settings.

Lets go through the steps of setting up a new FreeIPA server. We are going to use the official ansible repositories and collection for this purpose.

For this article we have the following assumptions:

* Ansible host in the same subnet with the server that needs to be set up with FreeIPA.
* ssh connectivity without password (ssh key) to FreeIPA server
* FreeIPA server with CentOS 7 at freeipa.cloudalbania.com with minimum 1 Gb memory and 8Gb disk space
* you already have vCenter up and running

## Preparing the Ansible host and FreeIPA repository

We are going to use the official [ansible repository](https://github.com/freeipa/ansible-freeipa) to install FreeIPA. On a host with ansible 2.9+ issue the following commands to install and setup initial FreeIPA server

Prepare the git repo and the inventory file

```console
$ git clone https://github.com/freeipa/ansible-freeipa.git
$ cd ansible-freeipa
$ echo << EOF > inventory/my-freeipa-server
[ipaserver]
freeipa.cloudalbania.com
  
[ipaserver:vars]
ipaserver_domain=cloudalbania.com
ipaserver_realm=CLOUDALBANIA.COM
ipaadmin_password=<STRONG PASS>
ipadm_password=<STRONG PASS>
EOF
```

Install the ansible collections for freeIPA:

```console
$ ansible-galaxy collection install freeipa.ansible_freeipa -p ./
```

Customize the _ansible.cfg_ file:

```console
$ cat ansible.cfg
[defaults]
host_key_checking = False
deprecation_warnings=False
collections_paths = ./
roles_path = ./roles
nocows=1
```

## Installing FreeIPA

On the same directory of the ansible repo run the following to install the FreeIPA server:

```console
$ ansible-playbook -u root -i inventory/my-freeipa-server playbooks/install-server.yml
```

After 3-4 minutes the server should be up and running

![ansible free ipa install](/ldap1.jpg)

Check the installation on the server with the _ipactl status_ command:

![ipa status](/ldap2.jpg)

Finally login to your server at <https://freeipa.cloudalbania.com> with user `admin@cloudalbania.com` and the password we set in the ansible inventory

![Main login screen](/ldap3.png)

Main screen after login

![Main screen after login](/ldap4.png)

## Configure FreeIPA for RFC4519 and vSphere

The next steps are following this FreeIPA [article](https://www.freeipa.org/page/HowTo/vsphere5_integration) to customize the directory schema for vSphere authentication.

```console
$ echo << EOF > vsphere_usermod.ldif  
dn: cn=users,cn=Schema Compatibility,cn=plugins,cn=config  
changetype: modify  
add: schema-compat-entry-attribute  
schema-compat-entry-attribute: objectclass=inetOrgPerson  
-  
add: schema-compat-entry-attribute  
schema-compat-entry-attribute: sn=%{sn}  
-  
EOF
```

```console
$ echo << EOF > vsphere_groupmod.ldif  
dn: cn=groups,cn=Schema Compatibility,cn=plugins,cn=config  
changetype: modify  
add: schema-compat-entry-attribute  
schema-compat-entry-attribute: objectclass=groupOfUniqueNames  
-  
add: schema-compat-entry-attribute  
schema-compat-entry-attribute: uniqueMember=%mregsub("%{member}","^(.*)accounts(.*)","%1compat%2")  
-  
EOF
```

Now apply with the following

```console
$ ldapmodify -x -D "cn=Directory Manager" -f vsphere_usermod.ldif -W  
```

and this

```console
$ ldapmodify -x -D "cn=Directory Manager" -f vsphere_groupmod.ldif -W 
```

Run following commands as admin to allow the new _sn_ attribute for _compat_ users and _uniqueMember_ for _compat_ groups:

```console
$ ipa permission-mod "System: Read User Compat Tree" --includedattrs sn
$ ipa permission-mod "System: Read Group Compat Tree" --includedattrs uniquemember

```

In case you have and error running the above commands then issue from the console the following command to authenticate first:

```console
$ kinit admin 
```

## Initial configuration for FreeIPA

At this point we need to create at least three resources in FreeIPA:

1. A bind user that will be used to bind to the LDAP server, we are using _bind-user@cloudalbania.com_
2. An end user, in this case _bzanaj@cloudalbania.com_
3. Two LDAP groups that will be used to add our users to _vcsa-admins_ and _vcsa-readonly_.

We are doing this in order to not add individual users permissions and rather manage permissions in our LDAP server.

Users in FreeIPA:

![LDAP users](/ldap_users.png)

LDAP Groups:

![LDAP groups](/ldap_groups.png)

Then add the users to the groups:

![LDAP user groups](/user_groups.png)

## Configure vSphere Authentication for FreeIPA

In the vSphere GUI go in _Admistration -> Single Sign On -> Configuration -> Identity Providers_ and then _Add_.

![vsphere_identity_provider](/vsphere_identity_provider.png)

In the next screen enter the following details as shown in the screenshot below:

![edit_identity_source](/edit_identity_source.png)

_Note: I am not using a certificate to authenticate on the LDAP server as it is out of the scope of this article._

After you save this configuration and there are no errors then you can assign the groups in the _Permissions settings_ in _Access Control_

![vcenter_add_permissions](/vcenter_add_permissions.png)

In the end we should see the following:

![vcenter_final](/vcenter_final.png)
