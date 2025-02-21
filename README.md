<h1>Windows-Forensics</h1>

Computer forensics is an essential field of cyber security that involves gathering evidence of activities performed on computers.

Like in a real world crime scene, artifacts are evidence found at the scene.<br>
These artifacts often reside in locations 'normal' users wont typically venture to.

<h2>Windows registry</h2>
The Windows Registry is a collection of databases that contains the system's configuration data. This configuration data can be about the hardware, the software, or the user's information. It also includes data about the recently used files, programs used, or devices connected to the system. <br>
<br>
The Windows registry consists of Keys and Values. When you open the regedit.exe utility to view the registry, the folders you see are Registry Keys. Registry Values are the data stored in these Registry Keys. A Registry Hive is a group of Keys, subkeys, and values stored in a single file on the disk.<br><br>

<h3>Structure of the Registry:</h3>

The registry on any Windows system contains the following five root keys:

HKEY_CURRENT_USER<br>
HKEY_USERS<br>
HKEY_LOCAL_MACHINE<br>
HKEY_CLASSES_ROOT<br>
HKEY_CURRENT_CONFIG<br><br>
You can view these keys when you open the regedit.exe utility. To open the registry editor, press the Windows key and the R key simultaneously. It will open a run prompt that looks like this: <br>

![lab](https://github.com/user-attachments/assets/013f5b41-6041-4d77-891d-d02eacc8fed2)<br>

In this prompt, type regedit.exe, and you will be greeted with the registry editor window. It will look something like this:

![image](https://github.com/user-attachments/assets/6878505c-3ea2-4454-b6ed-99d93f2d31f9) <br>

Here is how Microsoft defines each of these root keys.<br>

![image](https://github.com/user-attachments/assets/f19ee49e-6f2f-4762-b8ff-b7f4cd5faf3e)<br>

<h3>Accessing registry hives offline</h3><br>
If you are accessing a live system, you will be able to access the registry using regedit.exe. However, if you only have access to a disk image, you must know where the registry hives are located on the disk. The majority of these hives are located in the C:\Windows\System32\Config directory and are:<br><br>

DEFAULT (mounted on HKEY_USERS\DEFAULT)<br>
SAM (mounted on HKEY_LOCAL_MACHINE\SAM)<br>
SECURITY (mounted on HKEY_LOCAL_MACHINE\Security)<br>
SOFTWARE (mounted on HKEY_LOCAL_MACHINE\Software)<br>
SYSTEM (mounted on HKEY_LOCAL_MACHINE\System)<br>

<h3>Hives containing user information:</h3>

Apart from these hives, two other hives containing user information can be found in the User profile directory. For Windows 7 and above, a user’s profile directory is located in C:\Users\<username>\ where the hives are:<br><br>

NTUSER.DAT (mounted on HKEY_CURRENT_USER when a user logs in)<br>
USRCLASS.DAT (mounted on HKEY_CURRENT_USER\Software\CLASSES)<br>
The USRCLASS.DAT hive is located in the directory C:\Users\<username>\AppData\Local\Microsoft\Windows. <br>

![image](https://github.com/user-attachments/assets/15178b86-d0ab-460a-a792-9aa012947af3) <br>

The NTUSER.DAT hive is located in the directory C:\Users\<username>\.<br>

![image](https://github.com/user-attachments/assets/16e3da4b-d5fa-4ebd-afc0-cac132c726fd) <br>

NTUSER.DAT and USRCLASS.DAT are hidden files.

<h3>The Amcache Hive:</h3>

Apart from these files, there is another very important hive called the AmCache hive. This hive is located in C:\Windows\AppCompat\Programs\Amcache.hve. Windows creates this hive to save information on programs that were recently run on the system.

<h3>Transaction Logs and Backups:</h3>

Some other very vital sources of forensic data are the registry transaction logs and backups. The transaction logs can be considered as the journal of the changelog of the registry hive. Windows often uses transaction logs when writing data to registry hives. This means that the transaction logs can often have the latest changes in the registry that haven't made their way to the registry hives themselves. The transaction log for each hive is stored as a .LOG file in the same directory as the hive itself. It has the same name as the registry hive, but the extension is .LOG. Sometimes there can be multiple transaction logs as well. In that case, they will have .LOG1, .LOG2 etc., as their extension. It is prudent to look at the transaction logs as well when performing registry forensics.<br> <br>

Registry backups are the opposite of Transaction logs. These are the backups of the registry hives located in the C:\Windows\System32\Config directory. These hives are copied to the C:\Windows\System32\Config\RegBack directory every ten days. It might be an excellent place to look if you suspect that some registry keys might have been deleted/modified recently.

<h2>Data Acquisition</h2>

When performing forensics, we will either encounter a live system or an image taken of the system. For the sake of accuracy, it is recommended practice to image the system or make a copy of the required data and perform forensics on it. This process is called data acquisition. <br> <br>

Though we can view the registry through the registry editor, the forensically correct method is to acquire a copy of this data and perform analysis on that. However, when we go to copy the registry hives from %WINDIR%\System32\Config, we cannot because it is a restricted file. But don't worry we can use the following tools:<br>
1. KAPE is a live data acquisition and analysis tool which can be used to acquire registry data.
2. Autopsy gives you the option to acquire data from both live systems or from a disk image.
3. FTK Imager is similar to Autopsy and allows you to extract files from a disk image or a live system by mounting the said disk image or drive in FTK Imager.

   <h2>System Information and System Accounts</h2>

   let's find out where to look in the registry to perform our forensic analysis.<br>

   <h3>OS Version:</h3>

If we only have triage data to perform forensics, we can determine the OS version from which this data was pulled through the registry. To find the OS version, we can use the following registry key:<br>

<b>SOFTWARE\Microsoft\Windows NT\CurrentVersion</b><br>

![image](https://github.com/user-attachments/assets/d19271d1-e23f-4bb6-a466-4b2a17651826)

<h3>Current control set:</h3>

The hives containing the machine’s configuration data used for controlling system startup are called Control Sets. Commonly, we will see two Control Sets, ControlSet001 and ControlSet002, in the SYSTEM hive on a machine. In most cases (but not always), ControlSet001 will point to the Control Set that the machine booted with, and ControlSet002 will be the last known good configuration. Their locations will be:<br><br>

<b>SYSTEM\ControlSet001</b><br><br>

<b>SYSTEM\ControlSet002</b><br><br>

Windows creates a volatile Control Set when the machine is live, called the CurrentControlSet (HKLM\SYSTEM\CurrentControlSet). For getting the most accurate system information, this is the hive that we will refer to. We can find out which Control Set is being used as the CurrentControlSet by looking at the following registry value:<br><br>

<b>SYSTEM\Select\Current</b><br><br>

Similarly, the last known good configuration can be found using the following registry value:<br><br>

<b>SYSTEM\Select\LastKnownGood</b><br>

![image](https://github.com/user-attachments/assets/5847c2db-6edb-40e8-9a8e-3444ebcbe423)<br>

<h3>Computer Name:</h3>

We can find the Computer Name from the following location:<br>

<b>SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName </b><br>

![image](https://github.com/user-attachments/assets/9753ff3b-ddd6-4f7c-81fe-1026115dda89)<br>

<h3>Time Zone Information:</h3>
![image](https://github.com/user-attachments/assets/681f4128-f96e-48ee-8c68-8a816614c861)

<br>
Time Zone Information is important because some data in the computer will have their timestamps in UTC/GMT and others in the local time zone. Knowledge of the local time zone helps in establishing a timeline when merging data from all the sources.<br>

<h3>Network Interfaces and Past Networks:</h3>

The following registry key will give a list of network interfaces on the machine we are investigating:<br>

<b>SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces<b>




 ### [YouTube Demonstration](https://youtu.be/7eJexJVCqJo)

<h2>Description</h2>
Project consists of a simple PowerShell script that walks the user through "zeroing out" (wiping) any drives that are connected to the system. The utility allows you to select the target disk and choose the number of passes that are performed. The PowerShell script will configure a diskpart script file based on the user's selections and then launch Diskpart to perform the disk sanitization.
<br />


<h2>Languages and Utilities Used</h2>

- <b>PowerShell</b> 
- <b>Diskpart</b>

<h2>Environments Used </h2>

- <b>Windows 10</b> (21H2)

<h2>Program walk-through:</h2>

<p align="center">
Launch the utility: <br/>
<img src="https://i.imgur.com/62TgaWL.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Select the disk:  <br/>
<img src="https://i.imgur.com/tcTyMUE.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Enter the number of passes: <br/>
<img src="https://i.imgur.com/nCIbXbg.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Confirm your selection:  <br/>
<img src="https://i.imgur.com/cdFHBiU.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Wait for process to complete (may take some time):  <br/>
<img src="https://i.imgur.com/JL945Ga.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Sanitization complete:  <br/>
<img src="https://i.imgur.com/K71yaM2.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Observe the wiped disk:  <br/>
<img src="https://i.imgur.com/AeZkvFQ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
