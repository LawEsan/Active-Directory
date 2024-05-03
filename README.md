# Active Directory Lab | (Oracle VirtualBox) with PowerShell

## Introduction 
I am going to build a Windows environment for a corporate organisation and implement Active Directory.
![AD Diagram]

##  Install VirtualBox
- Go to https://www.oracle.com/virtualization/technologies/vm/downloads/virtualbox-downloads.html
- Download Windows 64-bit Oracle VM VirtualBox Base Packages - 7.0.14
- Download Oracle VM VirtualBox Extension Pack 7.0.14  7.0.14 ExtPack
- The next stage is you must install Windows 10 Disc Image (ISO File) without the Media Creation Tool.

<a data-flickr-embed="true" href="https://www.flickr.com/photos/200533061@N05/53695515776/in/dateposted-public/" title="VirtualBox download"><img src="https://live.staticflickr.com/65535/53695515776_534153fcca_c.jpg" width="800" height="409" alt="VirtualBox download"/></a>

## Download a Windows 10 Disc Image (ISO File) Without the Media Creation Tool
Microsoft's Media Creation Tool is only for Windows. If you access the website from another operating system — like macOS or Linux — you're sent to a page where you can directly download an ISO file instead. To get those direct ISO file downloads on Windows, you'll need to make your web browser pretend you're using another operating system. This requires spoofing your browser's user agent.
- Go to https://www.microsoft.com/en-us/software-download/windows10ISO
- Click the three dots at the top of your Microsoft Edge browser, and then select More Tools → Developer Tools. Alternatively, you can press Ctrl+Shift+I on the keyboard.
- Click the Network icon  → Network Conditions to enable it.
- Under the "User Agent" section, uncheck "Use Browser Default"
- Chrome offers a long list of pre-configured user agents to choose from in a drop-down menu. For this to work, you have to trick Microsoft into thinking you're using a non-Windows operating system. Anything that isn't Windows-based will suffice, so we selected "Safari - Mac."
- Keep the Developer Tools pane open and refresh the ISO file download page. This time, when it loads, you'll see a drop-down menu where you can select the edition of the Windows 10 ISO you want to download.
![Developer Tools]

## Download Windows Server 2019
- Go to https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019
- Download English (United States) ISO downloads > 64-bit edition

## Create DC VM
NOTE: DC stands for Domain Controller. Create a VM using the Windows Server 2019 as the ISO Image.
- Open Oracle VM VirtualBox 
- Click New 
- Name: DC
- Folder: C:\Users\lesan\VirtualBox VMs
- ISO Image: → Other… → Windows Server 2019
- Type: Microsoft Windows 
- Version: Windows 10 (64-bit)
- Click the box “Skip Unattended Installation” → Next
- Base Memory: 2048 MB → Next → Next 
- Finish
- Right click on DC VM → Settings 
- Click on Advanced tab 
- Shared Clipboard → Bidirectional
- Drag’n’Drop’:  → Bidirectional
- Click on Network tab → Click Adapter 1
- Attached to: NAT
- Click Adapter 2 
- Attached to: Internal Network (This so you can obtain a DHCP address from the Domain Controller)
- Click OK

## Power on DC VM
- Power on DC VM by double clicking the icon under tools or clicking the green “Start” arrow. 
- You will see the Windows Server 2019 setup window → Click Next
- Install now
- Click Windows Server 2019 Standard Evaluation (Desktop Experience) → Next
- Custom: Install Windows only (advanced) → Next
- The VM may take some time to install, have patience.
- You should eventually see a Customise settings screen which is the built in Administrator account.
- Username is already set to Administrator. For Password type: Password1  → Reenter password: Password1

## Login to DC VM
- Click the Input tab → Keyboard → Insert Ctrl-Alt-Del
- Login with password “Password1 “
- Once logged in you will you see the following message: Do you want your PC to be discoverable by other pCs and devices on this network? → Click Yes
- For a better user experience meaning to stop the cursor from lagging and to enable the screen to correctly resize itself do the following command: Click on Devices tab → Insert Guest Additions CD image…
- Open File Explorer → CD Drive (D:) VirtualBox Guest Additions
- Click VBoxWindowsAdditions-amd64
- Click “Next” multiple times 
- Click “I want to manually reboot later”
- Finish
- Click on Start menu in the VM → Shut down
- Power on DC VM 
- Click the Input tab → Keyboard → Insert Ctrl-Alt-Del
- Login with password “Password1 “
- The user experience should be improved and the cursor should no longer be lagging.

## Configure IP address for DC NIC (Internal)
- Within the DC VM go to the Start menu → Settings → Network & Internet → Ethernet → Change adapter options
- Right click Ethernet → Status → Details
- The IPv4 Address is 10.0.2.15 → Close
- Right click Ethernet → Rename → Change Ethernet to _INTERNET_)
- Right click Ethernet 2 → Status → Details
- The IPv4 Address is 172.16.0.1 → Close
- Right click Ethernet 2 → Rename → Change Ethernet 2 to X_Internal_X
- Right click X_Internal_X → Properties → Internet Protocol Version 4 (TCP/IPv4) → Click Use the following IP address:
- IP address: 172.16.0.1
- Subnet mask: 255.255.255.0
- Preferred DNS server: 127.0.0.1 (Note: this is your loopback address meaning when you ping it, you are pinging yourself thereby using yourself as the DNS server)
- Click OK
- Close
![Rename Network Connections]

## Rename PC
- On the DC VM Home Screen, at the bottom, click the magnifying glass → About your PC → Rename this PC → Type “PC”
- Remember, DC stands for Domain Controller.
- Click Next  → Restart now
- Login to DC VM  → Click Input tab → Keyboard → Insert Ctrl-Alt-Del  → enter password “Password1“

## Install Active Directory Domain Services (AD DS)
- In the DC VM open the Server Manager app
- Click Add roles and features → Next → Next
- Click on Windows Server 2019 → Next
- Click box Active Directory Domain Services → Next
- Next → Install
- Once installed, on the top right corner in Server Manager, click  Notifications (the flag icon) → Promote this server to a domain controller
- Add a new forest
- Root domain name: mydomain.com → Next
- Password: Password1 
- Confirm password: Password1
- Click “Next” multiple times until → Install
- The DC VM will restart
- Notice on the login screen the username has changed to MYDOMAIN\Administrator → login with password: Password1

## Create Domain Account
- Now you will create your own dedicated domain account instead of using the built in administrator account 
- Click on Start menu → Windows Administrative Tools → Active Directory Users and Computers
- Right click mydomain.com (FYI this is your domain) → New → Organizational Unit (FYI this is to put your admin account in, think of it like a folder in Active Directory) 
- Name: _ADMINS → OK
- Right click _ADMINS → New → User
- First name: Law
- Last name: Esan
- Username: a-lesan
- Password: Password1
- Click box “Password never expires” → Next → Finish 
- Right click the new account Law Esan → Properties
- Member Of → Add
- Type “domain admins” → Check Names → OK → Apply → OK
- Now you have successfully changed the account to a domain admin account
- To use this new account, first logout by going to the Start menu and clicking Sign out
- Instead of logging into MYDOMAIN\Administrator, click Other user
- Login as the domain admin you created i.e  →
- Username: a-lesan
- Password: Password1

## Install Remote Access Server / Network Address Translation (RAS/NAT)
- In the DC VM open the Server Manager app
- Click Add roles and features → Next → Next
- Click on DC.mydomain.com Windows Server 2019 → Next
- Click box Remote Access → Next
- Click box Routing → Add Features 
- Note: DirectAccess and VPN (RAS) box will automatically become checked. This is fine, please continue.
- Click “Next” multiple times until → Install 
- Once installed click Close 
- In the Server Manager app → Tools → Routing and Remote Access
- Right click DC (local) → configure and Enable Routing and Remote Access → Next
- Click Network address translation (NAT) → Next
- For some reason you are unable to select the option “Use this public interface to connect the internet” therefore click “Cancel”
- Repeat the same steps of clicking Tools → Routing and Remote Access → Right click DC (local) → configure and Enable Routing and Remote Access → Next s → Click Network address translation (NAT) → Next
- Now you are able to click “Use this public interface to connect the internet”
- You should see your two network interfaces _INTERNET_ and X_Internal_X
- Click _INTERNET_ → Next
- Finish
- Now the DC (local) status should be green indicating the RAS/NAT has been configured

## Configure DHCP server on DC (Domain Controller)
This will allow our Windows 10 clients to get an IP address that will let them browse on the internet whilst still being within the private internal network.
- In the DC VM open the Server Manager app
- Click Add roles and features → Next
- Role-based or feature-based installation → Next
- Click on DC.mydomain.com Windows Server 2019 → Next
- Click box DHCP Server → Add Features → Next
- Click box Routing → Add Features 
- Click “Next” multiple times until → Install
- Close
- On the DC VM Home Screen, at the bottom, click the magnifying glass → type DHCP
- Open DHCP app →  Click dc.mydomain.com 
- Notice that both IPv4 and IPv6 are both red meaning they are not working.
- Click IPv4 → New Scope → Next
- Name: 172.16.0.100-200 (FYI this is the Range) → Next
- Start IP address: 172.16.0.100
- End IP address: 172.16.0.200
- Length: 24
- Subnet mask: 255.255.255.0 → Next → Next
- Lease Duration → 8 Days → Next (FYI this is how long the IP address will exist)
- Configure DHCP Options → Click “Yes, I want to configure these options now” → Next
- Router (Default Gateway) → IP address type: 172.16.0.1 → Click Add → Next
- Click “Next” multiple times until → Finish
- Right click dc.mydomain.com → Authorize
- Right click dc.mydomain.com → Refresh
- Notice that both IPv4 and IPv6 are both green meaning they are both working.

## Configure the Server manager to allow internet access from the Domain Controller
- Open Server Manager
- Click “Configure the local server”
- IE Enhanced Security Configuration → On → Change to Off for administrators and users → OK

## Use PowerShell to create Active Directory users
- Open Internet Explorer app 
- Click on this link to download Accounts Script: https://github.com/LawEsan/AD_PS/files/15196730/AD_PS-master.zip → Save file to the Desktop → Minimize all windows so you see a blank Home Screen 
- Once you download the AD_PS-master file onto the desktop of your DC VM, you must right click the file and Extract All, saving the extracted file to the Desktop for easy access. 
- Double click on extracted AD_PS-master file to open the file → Click on names file (this Notepad file contains 1051 random names which is the data set you will use as the employees in your organisation when setting up Active Directory)
- Type your first name and surname at the top of the list of names → Click File →  Save → Close Notepad
- Open PowerShell ISE → Run as administrator
- Click File → Open → AD_PS-master → 1_CREATE_USERS
- This will open the PowerShell script
- Now, if you try to run the script, that is by pressing the green play icon or F5 on the keyboard, PowerShell will show an error message. To solve this run this command: Set-ExecutionPolicy Unrestricted → press enter → Click yes to all 
- To run script manually type cd C:\Users\a-lesan\Desktop\AD-PS-master\AD-PS-master  → press enter
- Type “ls” to list all files in AD-PS-master and all four files should appear
- Click the play button in PowerShell or F5 on the keyboard to run script → Click Run once 
- Once the script is running, you will see PowerShell creating users using the names in the AD_PS-master file
- Minimize PowerShell (you will return to it later)
![Run AD Script]

## Create CLIENT01 VM
- Open Oracle VM VirtualBox 
- Click New
- Name: CLIENT01 
- Folder: C:\Users\lesan\VirtualBox VMs
- ISO Image: → Other… →  Select Windows 10 Disc Image (ISO File)
- Type: Microsoft Windows 
- Version: Windows 10 (64-bit)
- Click the box “Skip Unattended Installation” → Next
- Base Memory: 2048 MB → Next → Next 
- Finish
- Right click on CLIENT01 VM → Settings 
- Click on Advanced tab 
- Shared Clipboard → Bidirectional
- Drag’n’Drop’:  → Bidirectional
- Click on Network tab
- Attached to: Internal Network (This so you can obtain a DHCP address from the Domain Controller to emulate how a corporate network in an organisation operates)
- Click OK

## Install Windows 10 ISO file on CLIENT01 VM
- Power on CLIENT01 VM by double clicking the icon
- Next → Install now → Click “I don’t have a product key
- Select the operating system you want to install → Click Windows 10 Pro → Next
![Windows 10 Pro OS Setup]
- Click the box “I accept the licence terms” → Next
- Custom: Install Windows only (advanced) → Next
- Once it has finished installing, the VM will reboot and ask you to select a region
- Click United States → Yes
- US → Yes → Skip
- Set up for personal use
- Offline account → Limited experience  (if you do not select this, when you will be forced to create a Microsoft account which is unnecessary)
- Name: user
- Password → (No need for a password) so click Next
- Not now
- Choose privacy settings for your device  → toggle all to No  → Accept
- Once logged in you will you see the following message: Do you want your PC to be discoverable by other pCs and devices on this network? → Click Yes

## Ensure Internet is working 
- Switch account user to Domain Account “Law Esan” and login with password: Password1
- Open Command Prompt
- ipconfig → press enter key
- Default Gateway should be set to 172:16.0.1
- ping www.google.com →  to test the connectivity of your infrastructure is working

## Change PC name
- hostname → press enter key (your PC’s name will appear)
- Change your PC name by going right clicking the Start menu → System → Rename this PC (advanced) 
- Click Change
- Computer name: CLIENT01
- Click “Member of Domain:” → type “mydomain.com” → OK
- Username: lesan
- Password: Password1
- Click → OK
- System Properties → Close 
- Click Restart Now

## Power on DC VM and verify CLIENT01 VM has been connected to the domain
- Open Server Manager
- Click DHCP on side menu
- On the Start menu search menu, type “DHCP” and open the app
- Click dc.mydomain.com → IPv4 → Scope → Address Leases
- You should see one IP address lease from the CLIENT01 VM
- Open Active Directory Users and Computers app
- Click Computers
- You should see the CLIENT01 VM that you joined to the domain meaning you can use any of the 1052 accounts created in Active Directory to login.

## CLIENT01 VM
- Go back to the CLIENT01 VM
- Click “Other user”
- Login with username: a-lesan
- Password: Password1
- Open Commpand Prompt
- Type “whoami” → press enter key 
- You should see you are a member of “mydomain” and your username is “lesan”
- You have now created an intranet for your organisation and employees.
