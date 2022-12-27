# Scripts for System Administration Tasks

## Gathering System Information

Pre-req
Need to install <b>dmidecode</b> Linux tool. which will gather information about CPU information, server, memory, and networking

To get details about the Linux distributions which are located in /etc/ folder.
```
cat /etc/*-release
```

To get Kernel's information
```
uname -a
```

To uptime information , create a server-uptime.sh
```
server_uptime=`uptime | awk '{print $3,$4}'| sed 's/,//'| grep "day"`;
if [[ -z "$server_uptime" ]]; then
       server_uptime=`uptime | awk '{print $3}'| sed 's/,//'`
       echo $server_uptime
else
       :;
fi;
```

To get physical server's information
```
$ sudo dmidecode -s system-manufacture
$ sudo dmidecode -s system-product-name
$ sudo dmidecode -s system-serial-number
```

To get CPU's information
```
sudo dmidecode -t4|awk '/Handle / {print $2}' |sed 's/,//'
```

## Gathering network information and connectivity diagnostics

How to do it...
1. Create test_ipv4.sh script:
```
if ping -q -c 1 -W 1 8.8.8.8 >/dev/null; then
  echo "IPv4 is up"
else
  echo "IPv4 is down"
fi
```

2. test IP connectivity and DNS, create test_ip_dns.sh
```
if ping -q -c 1 -W 1 google.com >/dev/null
then
  echo "The network is up"
else
  echo "The network is down"
fi
```

3. To test web connectivity:
```
case "$(curl -s --max-time 2 -I http://google.com | sed 's/^[^ ]*  *\([0-9]\).*/\1/; 1q')" in
  [23]) echo "HTTP connectivity is up";;
  5) echo "The web proxy won't let us through";;
  *) echo "The network is down or very slow";;
esac
```

How it works...
1. Execute the script as $ bash test_ipv4.sh. Here, we are checking the connection with the 8.8.8.8  IP address. For that, we use the ping command in the if condition. If the condition is true, we will get the statement written and printed on the screen as an if block. If not, the statement in else will be printed.
2. Execute the script as $ bash test_ip_dns.sh. Here, we are testing the connectivity using the hostname. We are also passing the ping command in the if condition and checking if the network is up or not. If the condition is true, we will get the statement written in an if block that's printed on the screen. If not, the statement in else will be printed.
3. Execute the script as $ bash test_web.sh. Here, we are testing the web connectivity. We use the case statement here. We are using the curl tool in this case, which is used to transfer data to and from a server.

## Monitoring directories and files

Getting ready

<b>inotify</b> is a tool in Linux which is used to report when a file system event occurs. Using inotify, you can monitor individual files or directories.

How to do it ...
```
#! /bin/bash
folder=~/Desktop/abc
cdate=$(date +"%Y-%m-%d-%H:%M")
inotifywait -m -q -e create -r --format '%:e %w%f' $folder | while read file
do
    mv ~/Desktop/abc/output.txt ~/Desktop/Old_abc/${cdate}-output.txt
done
```

How it works...
The inotifywait command is mostly used in shell scripting. The main purpose of the inotify tool is to monitor the directories and new files. It also monitors the changes in the files.

## Creating SSH keys for password less remote access

SSH is an open source network protocol and is used to log in to the remote servers to perform some actions. We can use the SSH protocol to transfer files from one computer to another. SSH uses public key cryptography.

How to do it...

1. First, we are going to create a SSH key. The ssh-keygen command is used to create a SSH key. Run the command as follows:
```
$ ssh-keygen
```

Output
```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/student/.ssh/id_rsa): /home/student/keytext
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/student/keytext.
Your public key has been saved in /home/student/keytext.pub.
The key fingerprint is:
SHA256:6wmj6l9EcjufZhvwQ+iKIqEchO1mtEwC/x5rMyoKyeY student@ubuntu
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|.                |
|oo  . o          |
|o.=  + o         |
|.* o  * S        |
|oo* oo * o       |
|=*.. o= X        |
|O. .*+ * =       |
|+E==+o  +        |
+----[SHA256]-----+
```

2. Now, we will copy the SSH public key to the remote host. Run the following command:
```
$ ssh-copy-id remote_hostname
```

3. Now, you can log in to the remote host without a password. Run the ssh command as follows:
```
$ ssh remote_hostname
```

## Creating users and groups systematically

How to do it...
The useradd command is used to create a user. Use a while which will read .csv file and for loop to add each user that's present in .csv file.

Create add_user.sh
```
#!/bin/bash
#set -x
MY_INPUT='/home/mansijoshi/Desktop'
declare -a SURNAME
declare -a NAME
declare -a USERNAME
declare -a DEPARTMENT
declare -a PASSWORD
while IFS=, read -r COL1 COL2 COL3 COL4 COL5 TRASH;
do
    SURNAME+=("$COL1")
    NAME+=("$COL2")
    USERNAME+=("$COL3")
    DEPARTMENT+=("$COL4")
    PASSWORD+=("$COL5")
done <"$MY_INPUT"
for index in "${!USERNAME[@]}"; do
    useradd -g "${DEPARTMENT[$index]}" -d "/home/${USERNAME[$index]}" -s /bin/bash -p "$(echo "${PASSWORD[$index]}" | openssl passwd -1 -stdin)" "${USERNAME[$index]}"
done
```

## Creating a simple NAT & DMT firewall

Getting ready
Ensure <b>iptables</b> installed

How to do it...
Create dmz_iptables.sh 
```
# set the default policy to DROP
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
# to configure the system as a router, enable ip forwarding by
sysctl -w net.ipv4.ip_forward=1
# allow traffic from internal (eth0) to DMZ (eth2)
iptables -t filter -A FORWARD -i eth0 -o eth2 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
iptables -t filter -A FORWARD -i eth2 -o eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
# allow traffic from internet (ens33) to DMZ (eth2)
iptables -t filter -A FORWARD -i ens33 -o eth2 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
iptables -t filter -A FORWARD -i eth2 -o ens33 -m state --state ESTABLISHED,RELATED -j ACCEPT
#redirect incoming web requests at ens33 (200.0.0.1) of FIREWALL to web server at 192.168.20.2
iptables -t nat -A PREROUTING -p tcp -i ens33 -d 200.0.0.1 --dport 80 -j DNAT --to-dest 192.168.20.2 
iptables -t nat -A PREROUTING -p tcp -i ens33 -d 200.0.0.1 --dport 443 -j DNAT --to-dest 192.168.20.2
#redirect incoming mail (SMTP) requests at ens33 (200.0.0.1) of FIREWALL to Mail server at 192.168.20.3
iptables -t nat -A PREROUTING -p tcp -i ens33 -d 200.0.0.1 --dport 25 -j DNAT --to-dest 192.168.20.3
#redirect incoming DNS requests at ens33 (200.0.0.1) of FIREWALL to DNS server at 192.168.20.4
iptables -t nat -A PREROUTING -p udp -i ens33 -d 200.0.0.1 --dport 53 -j DNAT --to-dest 192.168.20.4
iptables -t nat -A PREROUTING -p tcp -i ens33 -d 200.0.0.1 --dport 53 -j DNAT --to-dest 192.168.20.4
```