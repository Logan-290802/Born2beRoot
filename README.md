*This project has been created as part of the 42 curriculum by lveerapa*

# Born2beRoot
42 Born to Be root project is an exercise in system administration

# Description
## Project Description
The Born2beRoot project is aimed at introducing virtualisation and system administration. For this project we need to set up a virtual machine in VirtualBox. We have the options of setting up either Debian or Rocky. I have decided to go with Debian -My reasoning for this is outlined below.
An overview of what we need to do:

- Chose which OS we want to set up
- Find the right location within our local device to store the VM
- Set up the VM in VirtualBox
- Create at least 2 **Encrypted** partitions using LVM
- Set up and SSH service that will be running on the 4242 port in the VM
	-Ensure it isn't possible to connect using SSH as root.
- Configure the OS with UFW and only leave port 4242 open in the VM
- Set up the host name to the following format {username42}
- Implement a strict password policy
- Install and configure sudo following strict predetermined Rules
- Already set up a user with my username alongside root
	- Set this up to belong to user42 and sudo groups
- Create a bash script called monitoring.sh that will display select information on all terminals. This will be done using cron

> All details are outlined in detail below.

---

### Debian Vs Rocky
#### What is Debian
Debian is a free, open-source  operating system known for its stability, security and strong commitment to free software.
Forms the foundation for many major Linux distros and is developed by a large volunteer community
- Provides a robust environment for desktop and servers
- Community-driven, secure and highly reliable
- Base for popular distros like Ubunto, Linux Mint, Kali linux

> Debian OS has a layered structure which organises its components into hierarcical levels: Hardware, Kernel, System Utilites, applications.

This is done to ensure efficient, secure and modular system functioning.
###### Layers
**Layer 1: Hardware**
- Represents the physical components of the computer, such as COU, memory, storage devices and IO hardware.
- Supports multiple architectures allowing it to run on diverse hardware platforms.

**Layer 2: Linux Kernel**
- Linux kernal is at the core of Debian OS and directly interacts with the hardware.
- Manages the essential tasks such as memory allocation, process scheduling, file handling device control and resource management.

**Layer 3: System utiilities and Package management**
- Includes APT packet manager. dpkg (Debian Package), system libraries and GNU utilites.
- Essential commands and tools for system administration, file management, text processing and package operations sit here.

**Layer 4: Destop environment & applications**
-User facing applications, desktop environments (GNOME) & 3rd party software.
- This layer leverages underlying layers to provide services such as web browsing, dev tools etc.

##### Why do we use Debian
- Free and open source philosophy: adheres strictly to free software principles giving users freedom to run, study and modify and distribute the software.
- Advanced package management: Uses apt (advanced package tool) as it;s primary package management system for installing, updating & removing software
- Stability and Reliability: before packages reach the stable branch, there is extensive testing which ensures stability and Reliability
- Universal compatability - Supprots multiple hardware architectures. It can therefore run on everything from embedded devided and PCs to enterprise servers.
- Focus on security
- Community Driven Development - All volunteer organisation debian is developed transparently by its global community


#### What is Rocky
Rocky Linux emerged in response to the CentOS project's shift from a downstream, rebuild of RedHat Enterprise Linux to CentOS stream leaving void for users seeking a stable and free RHEL alternative. Gregory Kurtzer (one of the original co-founders of CentOS) created Rocky Linux project creating to provide a community driven, Enterprise ready linux distro.

It is available in different congfigs.
- Server edition - Standard tailored for server environments
- Workstation edition - designed for personal computers for work. It offers a more complete desktop experience
- Minimal Image- specialised version with most basic components needed for specific setups, Ideal for cutom configs when you only want necessary software
- Cloud image - Built for cloud computing environments, optimised for efficient operation in virtualised cloud based systems

##### Why do we use Rocky
- Server Management - ideal for web servers, database servers etc
- Cloud computing - Cloud friendly for both public and private cloud deploments, cost-effective and reliable for virtualised environments.
- Desktop - Highly stable, secure and customisable for a tailored desktop experience
- Education and Research - affordable and open-source, ideal for learning and experimentation
- Personal use - for technical users perfect for stability and versatile for everyday tasks
##### Strengths & Weaknesses of Rocky

###### Strengths
- Binary Compatibility with RHEL who are the leading enterprise level Linux distro.
> A binary is the compiled machine code that a machine understands and is able to execute. The binary of a piece of software compiled for a distro may vary depending on kernal version, libraries system calls etc. Rocky promises RHEL binary compatability which means any software you design and run on Rocky you should be able to do the same on REHL and vice versa.
- Community driven development - It has a community driven development model & therefore a diveres group of developers and users contribute to Rocky.
- Security and Scalability - Inherits security and stability features from RHEL
- Continuation of CentOS which aimed to provide a stable and reliable platform for organisations
- Rocky Linux Plus - Subscription offering additional featuees for enterprises

###### Weaknesses
- Not as established as Debian or CentOS
- Evolving ecosystem - May not be able to keep up with latest advancement in tech
- Limited package availability. Although it benefits from RHEL package ecosystem, there may be instances where certian packages are not readily available
- Fairky young community, doesn't match size and experience found in communities supporting established distros

#### Why Debian over Rocky?

Rocky's founding principles and basis is for enterprises. It lends itself well for users with a technical expertise and want to set up a fully customisable OS that is secure and relialbe.
Debian however is a lot more beginner friendly with flexibility with different architectures, APT and dpkg as package managers facilitating setting up and runing the OS.

---

### APT vs Aptitiude
#### What is APT
- Package manager for Debian based Linux system
- Simpler, faster and command line tool focusing in speed and ease of use.
#### What is Aptitiude
- Package manager for Debian based linux system
- More interactive and advanced text based UI nd superior dependency resolution
#### Why APT
Since we are are not setting up the GUI for debian, it makes more sense to go with APT as package manager as it is desinged around ease of use and command line tool which is primarily what we will be using.


### AppArmor vs SELinux
#### What is AppArmor
AppArmor (Application Armor) is a Linux module that enhanced system security by restricting programs' capabilites via per-programme profiles. A Mandatory Access Control framework is provided to resprict applications eg web browsers, servees etc from accessing files or other capabilites that it doesn't require
AppArmor uses file paths to define rules, making it easier to configure.

Run to see if the Linux distro already has AppArmor integrated

```
aa-status
```
It confines programmes according to a set of rules that specify what files a given programme is allowed to access.
It does this with profiles loaded into the kernel when the system starts.
The two profile modes are *enforcement* and *complain*.
Enforcement profiles enforce that profile's rules and reports violations into syslog or auditd.
Profiles in complain mode don't enforce any profiles, just log violation attempts.

Profiles are stored /etc/apparmor.d/

You can check Apparmor's status with 

```
sudo apparmor_status
```

which will return a list of all profiles in complain mode, which processes are defined in enforce and complain.

It helps defend against attacks by ensuring that even if a programme is compromised, its access to the rest of the system is strictly limited
 
#### What is SELinux
SELinux (Security enhanced Linux) is a security architecture forLinux systems that allows administrators to have more control over who can access the system. Originally developed by the NSA as a series of patches to hte Linux security Modules (LSM).

SELinux define access controls for the applications, processes and files on a system. It uses security policies i.e. a set of rules that tell SELinux what can or can't be accessed. 

When an application aka subject makes a request to access an object e.g a file,SELinux checks with an **Access vector cache (AVC)** where permissoins are cached for subjects and objects. If SElinux can't decide based off cached permissions it sends the request to the security server where it checks for the security contect of the app, process file etc. Context is then applied from the SELinux policy database and permission is either granted or denied.'

> AppArmor is path-based vs SELinux is label based.
> AppArmor is easier to learn and configure than SELinux ideal ffor desktops and simple servers

---

### UFW vs firewalld

Every person, business, government, etc. uses the web to communicate, exchange currency and data, and generally go through the motions of daily life and operations. However, these connections are not inherently safe, and because of this, we have to put defensive measures in place to keep our location, information, and money protected. In times past, when someone wanted to secure their possessions, they erected gates and fences to keep intruders at a distance. Today, we accomplish these same goals with the use of firewalls.

#### What is UFW
UFW (Uncomplicated Firewall) ia a user friendly command-line interface for managing firewall rules on Linux. It is the default firewall for ubuntu providing an easy way to block or allow network traffic by port, service or IP address. By default disabled.

When you turn UFW on it uses a default set of rules. All 'incoming' is being denied with some exceptions

to turn it on run 
```
sudo ufw enable
```
>to check the status 
```
sudo ufw status verbose or sudo ufw status
```

to find ports which do not deny incoming 
```
sudo ufw show raw
```

to allow incoming 
```
sudo ufw allow <port>/<optional:protocol>
```
to deny incoming
```
sudo ufw deny <port>/<optional:protocol>
```

to delete exiting rule 
```
sudo ufw delete <rule>  e.g sudo delete deny 53
```

to allow entire services 
```
sudo ufw allow ssh
```
to deny entire servies 
```
sudo ufw deny <service name>
```

#### What is firewalld
Firewalld is a zone-based firewall. Zone-based firewalls are network securty systems that monitor traffic and take acitons based on a set of defined rules applied against incoming/outgoing packets

Firewalld provides different different levels of security for different connection zones. Generally the rile for a firewall is deny everythign and only allow specific expections to pass. 
#### Why am I using UFW over firewalld

Firewalld is better suited for RHEL or binary compatible distros such as Rocky and is more complex to setup. UFW is a lot more user friendly and easier to set up. Ideal for this project

---

### VirtualBox vs UTM
#### What is VirtualBox
VirtualBox is a free open source hypervisor that enables users to create and run multiple virtual machines simultaneously inside their curret operating system. It acts as a virtalisation layer allowing users to run different guest OSs on different host OSs eg running a Linux distro on a windows desktop.

#### What is UTM
UTM is a free and opensource virtualisation and emulation software but only for macOS and IOS based systems. It allows users to run windows or Linux distrs on apple silicon and intel based Macs. 

#### Similarities and Differences
They share very similar functionalities, the biggest differneciator is that UTM is soley for macOS systems and since this project is being done on an ubuntu OS we will be using virtualBox

---

### Design Choises made during the setup
##### What is Partitioning
#### Partitioning
Partitioning whe setting up an OS is the process of diving a single physical hard drive or SSD into multiple independent logical sections (partiions)
Each partition acts as a seperate disk allowing for oraganised file storage, improved security or the installation of different OSs. This also allows to manage storage storage efficiently.

##### Why is Partitioning important?
##### What is LVM
The logical volume manager in linux is a device mapping framework that provides a flexible abstact layer between pyhsical storage (disks and partitions) and file systems. It allows admin to compine multiple harddrives into a single pool, resize volume dynamically and create snapshots without downtime. 

Physcial Volue: the underlying storage (e.g the hard drive)

Volume Groups: A pool formed by combining one or more physical volume

Logial volume: Virtual paritions created from the volume gorup spac ewhich are used to hold filesystems

Perks of LVMs include  
Dynamic resizing: VOlumes can be resixed while the machine is running overcoming the limitations onf static partions

Storage abstaction: File systems are not restricted to the size of a single physical disk

Easy management: Allows renaming, replacing and adding drives on the fly. Perfect for servers

##### What does traditional linux Partitioning look like

Traditional Linux partitions are organised using Master Boot Record or GUID Partition table scheme appraring as /dev/sda1.

dev = device
sda = SCSI Disk or SATA Disk \> storage drive type
a = first physical drive found by the system
1 = represents the first primary/ logical partition on the drive

/dev/sda = entire physical raw disk
/dev/sda1 = specific partition in first disk
/dev/sdb = second drive with partitions labelled sdb1, sdb2 ..

In MBR only 4 paritions are allowed sda2 in the example given is known as an extended partition. 
You cannot mount sda2. It has no filesystem and doesn't hold data directly, it
simply contains the logical partions (sda5,sda6...)

##### What is mounting

Mounting is the process of attatching a storge device or filesystem to
a spacific directory in the system's unified directory tree.

In linnuk for any device to be useable it must be hooked onto a point in this
tree

Mount point - The directory where the devide contents will appear

Storage Device: Physical or virtual source e.g. hard drive partition 

Filesystem type : the format of the data on the device.

The basic structure of core directories typically follow the Filesystem heirachy
Standard (FHS) which organises everything into a tree like structure from one
sinlge root

```
/root : top level of the tree. Everything else is nestes inside 
│
├── /boot: Contains Linux kernel and files needed to start the computer
|
├── /etc: "Control room", containing system wide configuration files (network settings, user passowrds)
|
├── /bin: "The toolbox" holding basic commands like ls and cp 
|
├── /sbin: Holds admin-only tools (reboot asn fdisk)
|
├── /usr: "The Library" where most of the installed programmes and their data is kept
|
├── /home: "The private quaters" for users. Each person gets a folder here for their personal files
|
└──/var: Stores variable data that changes often such as system logs and databases
```

To actually have a Linux file system that functions you require:
1. A bootloader: A small programme (GRUB) that tells the computer how to finD and laod the OS
2. The kernel: The brain of the OS that manages hardware like CPU AND RAM
3. An init system: The very first process that starts after the kernel loads, which then starts all other services
4. Shared Libraries: (/lib) Reusable snippets of code that programmes need to run. 
5. A journaling filesystem: A way to track changes so the system can quickly recover if it crashes

---

###  Security Protocols
#### Password
##### What is a password policy?
A passowrd policy is a set of rules and guidelines implemented to ensure users create strong, secure passwords ensuring a high level of protection

##### What is the password policy for this project?
- Has to expire every 30 days
- Minimum number of days before the modification of a password is 2
- The user hads to receive a warning 7 days before the password expires
- Your passwod must be
	- At least 10 characters long,
	- Must contain an Uppercase letter
	- Must contain a lowercase letter
	- Must contain a number
	- No more than 3 consecutive Identical characters
	- Must not include the name of the user
	|The following rule does not app;;y to the root password
	- The password must have at least 7 characters that are not part of the former password

#### Understanding Password policy in Debian filing system & viewing password policies

Thw **password aging** rules we require i.e expiration sit in
/etc/login.defs. To access this we don't need to install anything just navigate to
```
sudo vim /etc/login.defs 
```
and set the password parameters
```
PASS_MAX_DAYS 30 : Maximum days until password expiration
PASS_MIN_DAYS 2 : Minimum days until Password Change
PASS_WARN_AGE 7 :Days until password expiration warning
```

Password Policies however are primarily managed through Pluggable Authentication Modules (PAM). 
The main configuation file for **password complexity** sits in
/etc/pam.d/comom-password

to use the PAM we need to 
```
sudo apt install libpam-pwquality
```
then we can add the password rules we require

```
sudo vim /etc/pam.d/common-password
```
immediately after retry=3, add the following on the same line

```
minlen=10 ucredit=-1 dcredit=-1 lcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```
minlen=10 : Minimum characters a password must contain
ucredit--1 : Password must contain at least one uppercase letter
dcredit=-1 : Password must contain at least digit
lcredit=-1 : Password must contain at least one lower letter
maxrepeat=3 :Password connot have the same charater repeated more than 3 times consecutively
reject_username: Passward cannot contain the username
difok=7 : Password must contain at least seven different characters from the previous password
enforce_for_root: Apply this password policy to the root user

---

#### User Management
##### What is a user?
*in the beginning* the OG UNIX systems on which the Linux and other unix like
systems are based, they were designed as multi-user systems because PCs didn't
exist yet, which meant each user was connected to a mainframe via a dumb
terminal.
Therefore it was essential for a mechanism to exist whereby you could seperate
and and protect the files of the individual users while enabling simultaneous
use. Furthermore within this system you needed to allow a system administrator
to perform specific tasks within specific users files. Hence the root user.

Every account is automatically assigned a unique id and the system uses these
names rather than usernames to identify users

```
echo $UID
``` 
##### What is root user?
The root username has by default access to all commands and lines on a Linux or
other Unix-like OS. (aka. root account, root user or the superuser)
Root privileges are the powers that the root user has on the system. It is the
most privileged on the systeem and has absolute power. These powers include the
ability to modify the system in any way desired to grant and revoke access
permissions  for other users, including any those that are by default reserved
for root
This has nothing to do with other common uses of "root" in the Linux/Unix based
systems
- root directory: Is the top level directory on a system i.e the directory that other directories including their subdirectories and files reside
- /root: is the root user's home directory. Home directory is the primary repository of a user's files, including that user's configuation files and it's the directory in which a user finds itself in when they log into as system

The UID for root & all other users cna also be seen by looking at
**/etc/passwd**. This is the configuration file for user data. This is visible
to all users by default

##### Why shouldn't you log in as root user
The default permissions in Linux prevent access by ordinary users to critical
	parts of the system, files and directories belonging to other users. As a result, for some userswho aren't accustomed to permission based systems such as linux such as Windows might log in as root as make that their default. This can be extremely risky to the system as it is very easy to damage a UNIX-like system when using it as root. Ordinary work on a system should be done via an ordinary user. 
Because of the principle of UNIX like systems to offer maximum flexibility to
configure the sytem as they see fit, the root user is fully empowered. UNIX-like
systems assume the system administrator knows exactly what they are doing and
that that person will be the only person/people using the root user
In unix-like systems there is no safety net for the root user.

---

### Services Installed
#### sudo
##### What is sudo
Sudo stands for superuser do. This means ot allows the user to have temporary
root user access. *su* however stands for super user, this command allows you to
log in as root user and do whatever you want to there system. Unlike sudo, su is not just
temporary.

##### Sudo Rules
- Authentication using sudo has to be limited to 3 attempts in the event of incorrect password
- A custom message of my choice has to be displayed if an error due to a wrong password occurs using sudo
-Each action  using sduod has to be archived
	- Both Inputs and outputs
	- The log file has to be saved in the /var/log/sudo/folder
	- TTY mode has to be enabled (What is tty?)
- The paths that cna be used by sudo must be restricted to
	- /usr/local/sbin/
	- /usr/local/bin/
	- /usr/sbin/
	- /usr/bin/
	- /sbin
	- /bin
	- /snap/bin/

#### SSH
##### What is SSH
ssh stands for secure shell or secure socket shell. It is a network protocol
that helps users connect to another computer over an unsecured network.

ssh network protocol uses encryption to allow two connected devices (typically
server and client) to communicate securely. It allows users to safely command
and control distant machines. 
##### Why do we use
Secure communication: A client and a server can communicate securely via SSH. It
encrypts all information sent over the network

Authentication SSH offers methods for confirming the legitiacy of the client and
server. TO confirm identities it makes use of cryptographic keys.

Remote access: ssh is primarily used to enavle safe remote access to computeres,
servers and other resources. SSH can safely rin remote commands or access the
servers CLI from a distance
##### What is port forwarding and why do we need to do it
##### How to SSH into a vm form the terminal
```
ssh -p 2222 username@42
```

#### Cron
##### What is Cron?
Cron is a time based job scheduler in Unix-like operating systems, that
automatically runs scripts, commands or software at specified intervals.
 
#####Setting up a cron job 

Minute holds values 0-59
Hour holds values between 0-23
Day of the month values 1-31
Month of the year holds values 1-12 or Jan to Dec (first three letters of the
month)
Day of the week holds 0-6 or Sun-Sat
Command

The rules which govern the format of date and time field as follows: 

- When any of the first five fields are set to an asterisk(\*), it stands for all the values of the field. For instance, to execute a command daily, we can put an asterisk(\*) in the week's field.
- One can also use a range of numbers, separated with a hyphen(-) in the time and date field to include more than one contiguous value but not all the values of the field. For example, we can use the 7-10 to run a command from July to October.
- The comma (, ) operator is used to include a list of numbers which may or may not be consecutive. For example, "1, 3, 5" in the weeks' field signifies execution of a command every Monday, Wednesday, and Friday.
- A slash character(/) is included to skip given number of values. For instance,
"\*/4" in the hour's field specifies 'every 4 hours' which is equivalent to 0,
4, 8, 12, 16, 20.

```
* * * * * <command to execute>
| | | | |
| | | | day of the week (0–6) (Sunday to Saturday; 
| | | month (1–12)             7 is also Sunday on some systems)
| | day of the month (1–31)
| hour (0–23)
minute (0–59)
```

---

### Shell Script
#### The systax and functions
#### What each command does, now and why?
- Shell script called *monitoring .sh* developed in bash
- The script will display the following info on all terminals every 10mins
	•The architecture of your operating system and its kernel version.
	• The number of physical processors.
	• The number of virtual processors.
	• The current available RAM on your server and its utilization rate as a percentage.
	• The current available storage on your server and its utilization rate as a percentage.
	• The current utilization rate of your processors as a percentage.
	• The date and time of the last reboot.
	• Whether LVM is active or not.
	• The number of active connections.
	• The number of users using the server.
	• The IPv4 address of your server and its MAC (Media Access Control) address.
	• The number of commands executed with the sudo program.

---
# Linux file structure and purpose


---
# Instructions
## Installing packages
## Installing the essentials before advancing 
## Installing and configuring sudo
Switch to root user 
```
su -
```

install vim
```
apt-get install vim
```

install sudo
```
apt install sudo
```

to verify if sudo was successfully installed 
```
dpkg -1 | grep sudo
```
dpkg (Debian Package) is the low level, fundamental package management system
for debian and it's derivatives.
```
dpkg -l lists all the packages known to the system

| pipe takes this massive list from the dpkg command and feeds it to the next
command

grep sudo filters the list to show only the lines that contian sudo.
```

### Adding a user to Sudo group
```
add <username> sudo
or
usermod -aG sudo <username>
```
usermod is the utility used to modify user account's properties

-a tells the system to add the user to the group without removing them from
their current groups

-G specifies the supplemental groups you want to add the user to

sudo is the group name 

<username> the user you're upgrading

### setting up sudo policies
First we need to create a file in which to store the sudo policy
```
sudo touch /etc/sudoers.d/sudo_config
```

then we need to create a logging directory to store each commands input and
output

```
sudo mkdir /var/log/sudo 
```

now that everything is created we go to the file we created in sudoers.d to add
the policies we need to add. The rules are outlined above.

```
Defaults passwd_tries=3
Defaults badpass_message="Custom message"
Defaults logfile="/var/log/sudo_config"
Defaults log_input, log_output
Defaults iolog_dir="/var/log/sudo"
Defaults requiretty
Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```
requiretty means require the user logging as root to have a physical terminal.
A script for eg trying to run a sudo command would't work.
##Installing and configuring SSH
1. Update the system
```
sudo apt update
```

2. Install the OpenSSH server
```
sudo apt install openssh-server
```

3. Checking it was successfully installed 
```
sudo service ssh status
```

**SSH Configuration file**
As mentionned above the /etc directory in the root is the "control room". That
is where all the system configuations lie

```
sudo vim /etc/ssh/sshd_config
```

As the SSH service must be listening on port 4242 we need to uncomment and
change the port setting.

The other requirement is that we cannot connect using ssh as root.

We must also change the port settings on the SSH client config files.
```
sudo vim /etc/ssh/ssh_config
```

now we restart the ssh service to apply these settings

```
sudo service ssh restart

sudo sevice ssh status
```
After this we should see towards the bottom port 4242


## Installing & configuring UFW

To install UFW 
```
sudo apt install ufw
```

To activate ufw
```
sudo ufw enable
```

now we need to configure it to allow port 4242 to accept connections:
```
sudo ufw allow 4242
```

like the ssh we can do this to activate ufw
```
sudo service ufw restart
```

to see if everything is working as we want we run
```
sudo service ufw status

sudo ufw status
```
this will show us
-	if ufw is active
-	if we have allowed port 4242 to accept connections


## Setting up the strong password policy
As we covered there are 2 files which control the password policy. These are
found in 

```
/etc/login.defs  (for time constraints)
and
/etc/pam.d/common-password  (for password complexity)
```
we'll start with adding the time constraints

first we navigate to 

```
sudo vim /etc/login.defs
```

here we'll change:
- PASS_MAX_DAYS 99999 to PASS_MAX_DAYS 30
- PASS_MIN_DAYS 0 to PASS_MIN_DAYS 2

Now that thats done write the changes and we'll move to pam.d

for pam.d to work we need to install the libpam-pwqualty package like so
```
sudo apt install libpam-pwquality
```

now we need to navigate to the pam.d directory and open common password.

```
sudo vim /etc/pam.d/common-password
```

here we need to look for the password quality settings.

## How to ssh into the vm via terminal

The port forwarding has already been set up
1. Get the virtual machine running 
2. Open your terminal 
3. Run the following code
```
ssh <username>@localhost -p 2121 (or the host port you chose)
```

## Setting the Script info up

A script is a sequence of commands stored in a file that when executed, will
perform the specified commands.

###Architecture
Architecture refers to the design and organisation of a computer's components

To display the architecture of the OS you can use hte uname command with -a (a
means all).
```
uname -a
```
###Physical processors
The physical processors refer to the hardware of the CPU. To display the number of
physical processors, we use the following commanf

```
grep "physical id" /proc/cpuinfo | sort -u | wc -1
```

grep "physical id" /proc/cpuinfo searches in /proc/cpuinfo for the lines
containing "physical id"

sort -u sorts the output and removes the duplicates.

wc -l counts the number of lines in the output which corresponds to the number
of physical processors


###Virtual Cores
The virtual core is referred to as a logical processor in the /proc/cpuinfo
file. To find it we use the following command
```
grep -c "processor" /proc/cpuinfo
```
this command searches for lines contianing processor in cpuinfo and counts the
number of matches. THis ensures we accurately count the number of virtual cores
by counting the logical processors


###RAM

###Disk Memory

###CPU usage pecentage

###Last reboot

###LVM activation

###TCP

###Number of Users

###IP address

### Number of commands executed with sudo

###Total result of the script



**Note .d are directory for a folder that holds modular configuation files for
a specific service.

Note /sgoinfre/lveerapa/lveerapa42/lveerapa42.vdi**

---

## Step by Step guide to replicate
Step 1: Download VM iso from the officical debian website [here](https://www.debian.org/download.en.html)

Step 2: If you don't have virtualbox download it [here](https://www.virtualbox.org/) else start virtual box

Step 3: In virtual Box, navigate to the toolbar on top, click on machine then
click on new.

Step 4: Configure the settings which will be used to create the VM
    - The file location of the .iso image you got from Debian Website
    - Folder where this will be setup. At 42 it will be in the goinfre folder in
      the root directory
    - At the bottom select the skip unattended installation

Step 5 : Select the total Ram memory to reserve for the VM, in the Hardware
dropdown. I have gone 4096MB and 4 processors 

Step 6: Navigate to Hard Disk and select the following
    - Create a Virtual Hard Disk now
    - Select the Hard disk location and Size. The size I have gone for is 30GB

Step 7: Click Finish

Step 8: Virtual box will bring you back to the homepage, click start now.

Note: VMs doesn't support mouses, there <tab> moves, <space> selects, <enter>
activates. THis is mentionned at the bottom

Step 9: Upon starting select the version without graphical interface. This
should be the second option "install"

Step 10: Select language, mine is to english

Step 11: Select next, and virtual box will set up the vm.

Step 12: You will be prompted to set th hostname: For this exercise, we need it
to set it to username42.

Step 13: leave domain blank

Step 14: Set the root password, chose a strong password and make not of it 

Step 15: Set up the non root user credentials. YOu can use the same password as
before if you want

Step 16: Select Manual partioning

Step 17: you will see the general description of your memory partitions and
mount it. If there are no partitions you need ot create your own

Step 18: Accept the confirmation message

Step 19: select the new free space to configure it.

Step 20: Create a new partition

Step 21: View the subject.pdf to review example partition structure

Step 22: I've set my first one to 500m

Step 23: select primary. Note there can only be 4 primary per hard drive or
3 primary and one extended.
Etended is desinged to break the 4 primary partition limitation on a single
physical disk. Even if an HDD contains one extended, that extended can be
subdivided into multiple logical partitions.


Step 24: Select beginning as we want the new partitions to be created at the
beinging of the available space

Step 25: Modify the mountoint to /boot then select done settign up partition

Step 26: now we are going to create a logical partition using the rest of the
available space.

Step 27: Select the free space, create a new partition 
, set size to MAX, set partition type to logical, then select do not mount it
then select done setting paritions

Step 28: Select the configure encrypted volumes section and select yes to the
confirmation message

Step 29: Proceed to create logical volumes and select those you want to encrypt.
here it is sda5.

Step 30, select done and finish and accept final confirmation mestage


Configuring the logical volume manager
step 31:Click on configure logical volume manager and accept the confirmation
message.

Step 32: Select create volume group and give the group a name mine is LVMGroup

Step33: Select the destination for where this volume gorup will be created

Step 34: Create a logical volume, insert the name and how much you'd like to
allocate to it.

Repeat this process for each logical volume

Then click finish

Step 35: Now mount all your partitions. Select the ext4 jounralling system and
mount the partions to these directories


When finished double check everything is properly mounted then proceed to
finish and write the changes to the disk.

Step 36: Select no for additional packages

Step 37: Select your country

Configuring debian installation
Step 38: select deb.debian.org then leave HTTP proxy blank

Step 39:opt out of the populairty contest

Step 40: Deselect all software options

Step 41: Chose the destination for the bootloader installation /dev/sda

Step 42: reboot

---

# Resources

[Introduction to Rocky Linux](https://www.geeksforgeeks.org/linux-unix/introduction-to-rocky-linux/)
[Introduction to Debian Linux](https://www.geeksforgeeks.org/linux-unix/introduction-to-debian-linux/)
[What is binary Compatibility and what does it mean for Linux distros](https://securityboulevard.com/2024/08/what-is-binary-compatibility-and-what-does-it-mean-for-linux-distributions/)
[What is AppArmor](https://askubuntu.com/questions/236381/what-is-apparmor)
[What is SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux)
[A beginner's guide to Firewalld](https://www.redhat.com/en/blog/beginners-guide-firewalld)
[UTM | Virtual Machines for Macs](https://mac.getutm.app/)
[Oracle Virtual Box](https://www.virtualbox.org/)
[/etc/pam.d/common-passowrd example](https://community.learnlinux.tv/t/etc-pam-d-common-password-example/2785/2https://community.learnlinux.tv/t/etc-pam-d-common-password-example/2785/2)
[root Definition](https://www.linfo.org/root.html)
[Unix/Linux Privilege Management: Should You Sudo? Here’s What It Does and Why It’s Not Enough](https://www.beyondtrust.com/blog/entry/unix-linux-privileged-management-should-you-sudo)
[Introduction to SSH](https://medium.com/@aqeelabbas3972/introduction-to-ssh-secure-shell-0d07e18d3149)
[cron](https://en.wikipedia.org/wiki/Cron)
