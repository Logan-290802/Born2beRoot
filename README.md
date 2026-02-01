*This project has been created as part of the 42 curriculum by*

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

> Run **aa-status** to see if the Linux distro already has AppArmor integrated

It confines programmes according to a set of rules that specify what files a given programme is allowed to access.
It does this with profiles loaded into the kernel when the system starts.
The two profile modes are *enforcement* and *complain*.
Enforcement profiles enforce that profile's rules and reports violations into syslog or auditd.
Profiles in complain mode don't enforce any profiles, just log violation attempts.

Profiles are stored /etc/apparmor.d/
You can check Apparmor's status with **sudo apparmor_status** which will return a list of all profiles in complain mode, which processes are defined in enforce and complain.

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

>to turn it on run **sudo ufw enable**

>to check the status **sudo ufw status verbose** or **sudo ufw status**

>to find ports which do not deny incoming **sudo ufw show raw**

>to allow incoming **sudo ufw allow <port>/<optional:protocol>

>to deny incoming **sudo ufw deny <port>/<optional:protocol>

>to delete exiting rule **sudo ufw delete <rule>** e.g sudo delete deny 53

>to allow  entire services **sudo ufw allow ssh**

>to deny entire servies **sudo ufw deny <service name>**

#### What is firewalld
Firewalld is a zone-based firewall. Zone-based firewalls are network securty systems that monitor traffic and take acitons based on a set of defined rules applied against incoming/outgoing packets

Firewalld provides different different levels of security for different connection zones. Generally the rile for a firewall is deny everythign and only allow specific expections to pass. 
#### Why am I using UFW over firewalld

Firewalld is better suited for RHEL or binary compatible distros such as Rocky and is more complex to setup. UFW is a lot more user friendly and easier to set up. Ideal for this project

---

### VirtualBox vs UTM
#### What is VirtualBox
#### What is UTM
#### Similarities and Differences

---

### Design Choises made during the setup
#### Partitioning
##### What is Partitioning
##### Why is Partitioning important?
##### What is LVM
##### What does traditional linux Partitioning look like
##### What is mounting

---

###  Security Protocols
#### Password
##### What is a password policy?
##### What is the password policy?
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

##### How can I view the password policy?
#### Sudo Rules
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

#### User Management
##### What is root user?
##### Why shouldn't you log in as root user
##### Users available on booting.
##### What are user groups?

---

### Services Installed
#### sudo
##### What is sudo
##### Why is sudo important ?

### SSH
#### What is SSH
#### Why do we use
#### What is port forwarding and why do we need to do it
#### How to SSH into a vm form the terminal

### Cron
#### What is Cron?

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
'

---
# Instructions

---

# Step by Step guide to replicate


---

# Resources

[Introduction to Rocky Linux](https://www.geeksforgeeks.org/linux-unix/introduction-to-rocky-linux/)

[Introduction to Debian Linux](https://www.geeksforgeeks.org/linux-unix/introduction-to-debian-linux/)

[What is binary Compatibility and what does it mean for Linux distros](https://securityboulevard.com/2024/08/what-is-binary-compatibility-and-what-does-it-mean-for-linux-distributions/)

[What is AppArmor](https://askubuntu.com/questions/236381/what-is-apparmor)
[What is SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux)
[A beginner's guide to Firewalld](https://www.redhat.com/en/blog/beginners-guide-firewalld)
