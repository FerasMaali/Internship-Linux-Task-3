# Internship-Linux-Task
This was more or less and exam for what I learned about CentOS Linux

## Part 1: LVM 

Create a volume group, and set 16M as extents. And divided a volume group containing 50 extents on volume group lv, make it as ext4 file system, and mounted automatically under /mnt/data. Please note that this should be implemented on the second disk
```
	vgcreate -s 16M vgdata /dev/sdb1
	lvcreate -n lvdata -l 50 vgdata
	mkfs.ext4 /dev/vgdata/lvdata
	mkdir -p /mnt/data
	blkid | grep lvdata
	echo 'UUID="28a03528-288c-42fa-9627-fb936af48e82"           /mnt/data        ext4       defaults 1 2' >> /etc/fstab
```

## Part 2: users, groups and permissions 

1. Add user: user1, set uid=601 Password: redhat. The user's login shell should be non-interactive. (no ssh access to server)
```
	useradd --shell /sbin/nologin --uid 601 --password redhat user1
```
2. Add user1 to group TrainingGroup.
```
	groupadd TrainingGroup
	usermod -aG TrainingGroup user1
```
3. Add users: user2, user3. The Additional group of the two users is the admin group Password: redhat. user3 with root permissions
```
	useradd user2 --password redhat
	useradd user3 --password redhat
	groupadd admin
	usermod -aG admin user2
	usermod -aG admin user3
	usermod -aG wheel user3
```

## Part 3: SSH 

Generate SSH key and connect to different VM without password.
```
	ssh-keygen
	ssh-copy-id user@host
	ssh user@host
```

## Part 4: permissions

Copy /etc/fstab to /var/tmp name admin, the user1 could read, write and modify it, while user2 can’t do any permission.
```
	cp /etc/fstab /var/tmp/admin
	chown user1:user1 /var/tmp/admin
	chmod 640 /var/tmp/admin

	# We can also do this using ACL
```

## Part 5: permissions

SELinux must be running in the Enforcing mode (permanent even after reboot).
* Set `SELINUX=enforcing` in the file `/etc/selinux/config`

## Part 6: bash script and processes

Write a shell script that will keep running for 10 mins in the background and check the process that it's created and try to kill using commands.
```
	sleep 600
	process_id=`ps aux | grep 'sleep 600' | head -1 | awk '{ print $2 }'`
	kill $process_id
```

## Part 7: Yum Repo

1. Install tmux on your machine
```
	yum install tmux -y
```
2. Install apache server on your machine(httpd) and  Install mysql. 
```
	yum install -y httpd mysql
```
3. Create a local yum repository on your local machine(available publicly) with the zabbix rpms: https://repo.zabbix.com/zabbix/4.4/rhel/7/x86_64/
	* **ALREADY DONE SOMETHING SIMILAR**
4. Disable all other repositories and keep only the new repo       
	* Set enabled option to 0 for all repos in `/etc/yum.repo.d/` directory
5. Install zabbix rpms from the new repo (Download zabbix, zabbix-web,php, zabbix-server, zabbix-agent rpm’s and their dependencies)
	* **ALREADY DONE SOMETHING SIMILAR**

## Part 8: Network management

1. Open port 443 , 80
```
	# We need to disable firewalld before using iptables
	iptables -A INPUT -p tcp –dport 80 -j ACCEPT
	iptables -A INPUT -p tcp –dport 443 -j ACCEPT
```
2. Make the changes permanent
```
	service iptables save
```
3. Block ssh connection for your colleague ip to the VM.
```
	iptables -I INPUT -s 192.168.229.129 -p tcp --dport ssh -j REJECT
```

## Part 9: Cronjob

Create a cronjob that will run at 1:30 AM every day and collect the users logged in and save them in a file
Format : timestamp – users
Note: the cronjob can run a script.
* Use `crontab -e` to enter cron jobs file
* add `30 1 * * * w | tail -n +3 | awk '{ print $1 }' | sort | uniq` to the file

## Part 10: Mariadb  

1. install mariadb from the local repo that was created earlier.
```
	yum install mariadb
	yum install mariadb-server
```
2. open ports in iptables for mariadb.
```
	iptables -A INPUT -p tcp --dport mysql -j ACCEPT
```
3. create database , user(note: handle permissions).
```
	CREATE USER 'user'@'192.168.229.129' IDENTIFIED BY 'password';
	GRANT CREATE, INSERT ON *.* TO 'user'@'192.168.229.129';
```
4. connect to the database created in step 3 using the new user (with password)
	* At first, we connect from the remote host using remote user privilages
	```
		mysql -h 192.168.229.133 -u user -ppassword;
	```
	* Then we do our db stuff
	```
		CREATE DATABASE studentdb;
		USE studentdb;
		
		CREATE TABLE STUDENT ( 
			firstname VARCHAR(30) NOT NULL,
			lastname VARCHAR(30) NOT NULL,
			program_enrolled VARCHAR(30) NOT NULL
		);
		
		INSERT INTO STUDENT (firstname, lastname, program_enrolled)
		VALUES	('f', 'b', 'computer science'),
			('f', 'r', 'computer science'),
			('x', 'b', 'computer science'),
			('a', 'c', 'computer science'),
			
			('q', 't', 'electrical'),
			('w', 'l', 'electrical'),
			('e', 'j', 'electrical'),
			
			('y', 'h', 'mechanical'),
			('u', 'v', 'mechanical'),
			('o', 'x', 'mechanical');
	```

Example schema : 

Create a MariaDB database called studentdb on rhce1 and add ten records each containing “student firstname” (Allen, David, Mary, Dennis, Joseph, Dennis, Ritchie, Robert, David, and Mary), “student lastname” (Brown,Brown, Green, Green, Black, Black, Salt, Salt, Suzuki, and Chen), program enrolled in (3 x mechanical, 3 x electrical,and 4 x computer science), expected graduation year (2 x 2017, 3 x 2018, 5 x 2020), and a student number (110-001 to 110-010) 
