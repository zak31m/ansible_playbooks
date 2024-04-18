# ansible_playbooks


Non-prod status (nunifidcb100)   -- user Joshua Allison 

- Re-enable SELinux and see if application works properly now that the websocket issue was resolved
   *********   cmd - sestatus  **********  (it was enabled already) 
_________________________________________________________________________________________________
- Set vm.overcommit_memory = 1 permanently (it seems to get reset on reboot/patching)

********   cd /etc/sysctl.d 
*******   ls
******  vi 99-dcb.conf (creating a new file) 
*****   vm.overcommit_memory=1 ... wq! 
****** sysctl -p 99-dcb.conf (filename) - it makes it set after it reloads
_______________________________________________________________________________________________
- Right now I’m running the java application as root.  I’ve had app ID a41215r (a41216p in Prod) created to be owner of /opt/osl and all contents/processes, but haven’t chowned yet or started using it.  Last time I tried running the java app as anything other than root I was unable to.


- Grant e70382t access to sudo as a41215r

- Set up ssh/rsa keys for passwordless connections to nunifipas010 (punifipas020 in Prod)
  ***We DO NOT set up SSH/RSA KEYS for APP Teams ****
_____________________________________________________________________________________




___________
Prod steps to accomplish (punifidcb100) THIS IS A PROD SERVER ****

- Disable FIPS (always requires us to create a ticket and get it approved b4hand) 
******** fips-mode-setup --check  (should show its disabled or enabled)

_________________________________________________________________________________________
- set vm.overcommit_memory = 1
********   cd /etc/sysctl.d 
*******   ls
******  vi 99-dcb.conf (creating a new file) 
*****   vm.overcommit_memory=1 ... wq! 
*****  sysctl -p 99-dcb.conf (filename) - it makes it set after it reloads 
__________________________________________________________________________________________
- setsebool -P httpd_can_network_connect 1  (this is the full  command) run it. 
___________________________________________________________________________________________
- Install httpd packages (same as on nunifidcb100)
**** systemctl status httpd (to check if its installed) 
**** dnf install httpd -y (to install on RHEL 8) 
**** systemctl start httpd 
**** systemctl status httpd (should be installed and running) 
____________________________________________________________________________________________
- Create /opt/osl with owner a41216p (and disable noexec on this directory)
*** APPS NEED app specific storage we create seprate a separate volume group, on PROD VGAPP volume GROUP needs to be created and should be its own mounted file system.
 
*** lsblk (should see 4 pads) looks for availble one 
*** vgcreate vgapp /dev/mapper/mpathc
*** vgs (shows 100GB )
*** lvcreate -L 5G -n FS_osl vgapp
*** lvs (FS_osl shows 5GB memory)

now we need to put a FILE system on it ***
***mkfs.xfs /dev/mapper/vgapp-FS_osl***
*** vi /etc/fstab
** go all the way in the bottom ..
** add on *** /dev/mapper/vgapp-FS_osl  /opt/osl xfs   defaults,nondev   0 0 (copy it from above)
** mkdir /opt/osl (to mount it to this file) 
** mount -a 
*** df
___________________________________________________________________________________________
now need to chown it with that app id   "with owner a41216p"
*** chown a41216p: /opt/osl 
*** ls -ld /opt/osl (shows the owner)

*** df -h /opt/osl  (should show **size and mount point) 
____________________________________________________________________________________________
- Grant e70382t access to sudo as a41216p
*** id e70382t (is part of 2 groups "P for Prod ---- D for Dev server" 
 shows (ugr_ux_activeledgers_p_admin)   (ugr_ux_activeledgers_d_admin ALL=(ALL)

visudo -f /etc/sudoers.d/ugr_ux_activeledgers_p_admin
ALL=(ALL) NOPASS%UGR_UX-ActiveLedgers_P_Admin WD: /usr/bin/rootsh

***he will have to go "sudo rootsh" and then he can perform what he wants ![image](https://github.com/zak31m/ansible_playbooks/assets/161354671/93132fa1-11d4-4a45-b009-aa1d7361d744)


