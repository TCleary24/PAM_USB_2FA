# PAM

# Project Description

Pluggable Authentication Modules (PAM) are a feature in Linux that function as an API that authenticates services and users on a system.  This allows the rules for authentication to be dynamically configurable, allowing for a system administrator to define how users and applications authenticate within the Linux system. <br />
<br />
PAM functions with configuration files located in /etc/pam.d on the Debian release of Linux.  There is also a configuration file that is present, but is ignored in the presence of the PAM directory.  Authentication occurs by passing through a series of stacked modules.  The modules are run from top to bottom and a response is generated as pass or fail depending on the result of the series of rules defined in the modules. <br />
<br />
This project improves system security by establishing two-factor authentication requiring a user to not only know a password, but also have a removable USB device, in order to authenticate.  It leverages code that adjusts the PAM common-auth configuration file by installing a new USB module and reconfiguring the control flag to require a USB in addition to a password. <br />
<br />
The following steps were used to establish the necessary configuration implementing 2FA using a USB.  Note that changes to PAM files should be made with great care as errors could result in a user being completely locked out of a system.

# Prerequisites
VMWare: https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html <br />
Kali Linux base install using Debian OS: https://www.kali.org/get-kali/#kali-bare-metal <br />
Login as SU or root  <br />
Removable USB device available <br />
No prior changes to pam.d files (could cause fatal error during new module compilation below) <br />


# Steps

Install the pam_usb prerequisites by typing the following command: <br />
 <br />
  $ sudo apt install git libxml2-dev libpam0g-dev libudisks2-dev libglib2.0-dev gir1.2-udisks-2.0 python3 python3-gi
  <br />
 ![alt text](https://github.com/TCleary24/PAM_USB_2FA/blob/main/prereq%20installation.png)
  
   <br />
Clone the pam_usb GitHub repository and compile the code to install it with the following commands: <br />
  $ git clone https://github.com/mcdope/pam_usb.git <br />
  $ cd pam_usb/ <br />
  $ make <br />
  $ sudo make install <br />
   <br />
  
   ![alt text](https://github.com/TCleary24/PAM_USB_2FA/blob/main/install%20screen.png) <br />

<br />
Allow services to be restarted without asking to allow for faster installation once the following screen is presented by selecting "Yes". <br />
 <br />

![alt text](https://github.com/TCleary24/PAM_USB_2FA/blob/main/restartservicesmessage.png)

<br />


Add the usb device intended for authentication with the following command, where "USB20FD" is the name of the removable device: <br />
 <br />
  $ sudo pamusb-conf --add-device USB20FD <br />
  
  ![alt text](https://github.com/TCleary24/PAM_USB_2FA/blob/main/add%20device.png)
  <br />
   
Note: The device name can be found on the desktop after the device connection is established. <br />

![alt text](https://github.com/TCleary24/PAM_USB_2FA/blob/main/device%20name%20homescreen.png)<br />

Save the changes to the file <br />
  $ Y <br />
   <br />
Next, define a user for PAM authentication with the following command, where "tim" is the name of a user that exists on the system: <br />
 <br />
  $ sudo pamusb-conf --add-user tim
  
  ![alt text](https://github.com/TCleary24/PAM_USB_2FA/blob/main/add%20user.png)
  
 Save the changes to the file: <br />
  $ Y
  
The next steps involve navigating to the PAM directory and modifying the necessary files.  It is recommended to open another terminal as root so that any mistakes can be corrected without causing complete system lockout. <br />
 
Go to the PAM directory: <br />
  $ sudo cd /etc/pam.d
    
Open the common-auth file: <br />
  $ nano common-auth
  
![alt text](https://github.com/TCleary24/PAM_USB_2FA/blob/main/edit%20pamd.png)
  
 <br />
The configuration modules can be seen in white text where the rules are run from top to bottom.  Each row follows a common syntax with Type, Control Flag, Module, and Module Argument. <br />
<br />
Add the below pam_usb.so module below, setting the type to "auth" and the control flag to "required".  Next, set a module argument for the pam_unix.so module to "try_first_pass": <br />
 <br />
  $ auth  required  pam_usb.so <br />
  $ auth [success=1 default=ignore] pam_unix.so nullok try_first_pass <br />
   <br />
 Initial PAM common-auth file <br />
 
![alt text](https://github.com/TCleary24/PAM_USB_2FA/blob/main/common_auth_initial.png)

  <br />
   <br />
 Final PAM common-auth file <br />
 
![alt text](https://github.com/TCleary24/PAM_USB_2FA/blob/main/common_auth_final.png)
 
 <br />
Exit and save the changes: <br />
  $ Y

# Conclusion
Now, login attempts will fail when the USB is not inserted into the device.  These settings can be further customized to establish authentication requirements by adjusting the control flags.  For example, the pam_usb.so module could be given the "sufficient" control flag that would allow login immediately in the presence of the defined USB, without requiring a password.
