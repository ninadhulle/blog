# commands
ls -la  ---list directory permissions
ls -R ---list recursively
ls - altr ---list owners, hidden directories
ls -R | grep ":$" | sed -e 's/:$//' -e 's/[^-][^\/]*\//--/g' -e 's/^/   /' -e 's/-/|/' --- check for director
cp -R <path>/<files> <path>    ---copy 
chown <user>:<group> -R <directory> --- change ownership to user for directory -R recursively
rmdir <directory> ---remove directory if empty
rm -r -f <file(s) or path> --- removes files from current directory -r recursively, -f is for forcibly
service --status-all
service <daemon> status --- daemon is process name

grep -r '<pattern>' <file> ---search in files
grep -R --only-matching "com.cvs.specialty*" | sort --unique

find <directory> -name <filename> --- find files
ps -ef | grep "java" --- ps all process and pipe to grep to search for java
kill -9 <process id from ps ef> --- kill process with name

sudo -su <username> --- change to user with new shell
less <filename> --- open file G go to end of file, U pageup D pagedown
netstat -tulpn ---check open ports and processes
netstat -tulpn | grep <port> ---check if port 9000 is open
tcpdump -i eth0 tcp port <port> ---check tcp port on port
tcpdump -A -s 0 'tcp port 8280 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
sudo tcpdump -n -i ens192 'src host <host> and port <port>'
lsof -i ---display list of open ports
lsof -i :<port> ---display open port details
lscpu ---get info about CPU, processor architecture etc
lshw --- list hardware
hwinfo --- hardware info
top ---check cpu utilization
vmstat --- check vm memory
free --- check ram
iptables -L -n ---check port forwarding /open

id <user> --- get details about groups, uid
groups <user> ---get groups which user belong to
groupadd <user> -g <gid> --- add new group
groupmod -g <NEW_GID> <groupname>
cut -d: -f1 /etc/passwd ---list all the users
cut -d: -f1 /etc/group ---list all the groups
sudo -v ---validate of user has root access
sudo - l ---list out sudo permissions
cat /etc/sudoers
rsync -aqz <source dir> <dest dir> --- sync source and destinations
groupdel <group> ---delete group
userdel <user> --- delete user
findmnt --- check mounted folders
history --- get list of history commands
scp <sourcefolder/file> -i <path of key> -v <user>@<HOST>:<path where to copy> --- scp from source to destination on another server, v is for verbose
passwd -d <user> --- remove passwd for user
scp -v <path>/<file> <user>@<server>:<folder>
tar -zcvf <archive-name>.tar.gz /directory/path
top -p `sudo netstat -tulpn | grep ':8280' | awk '{print $NF}' | cut -d'/' -f 1` -b -n 1 | head -n 8 | tail -n 2 | awk '{print $1,"\011"$2,"\011"$9,"\011"$10}' --- print CPU/Memory/PID/USER
ps -p `sudo netstat -tulpn | grep ':8180' | awk '{print $NF}' | cut -d'/' -f 1` -o %cpu,%mem,pid,user --- print CPU/Memory/PID/USER

mount -t cifs -o username=,password= //<drive>  /mnt/s
sudo mount -t cifs -o username=,password= <drive>  /mnt/s-drive

scp <source folder>/<file> -i <ssh key location and name> -v <user>@ip:<destination directory> --- v is for verbose

cat ~/.ssh/id_rsa.pub | ssh <user>@<HOST_IP> "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys" --- take ssh public key and add it to machines auth keys
echo ' <user> ALL=(ALL)  ALL' >> /etc/sudoers --- add <user> to sudoers list, needs sudo permissions
safe add to etc/suoders.d
jenkins ALL=(ALL) NOPASSWD:ALL

%jenkins ALL=(ALL) NOPASSWD:ALL

Defaults:%jenkins !requiretty

df -h ---disk usage
du -h <directory> ---disage usage by directory
lsof +L1 --- get open large files
du -sh * | sort -hr | head -n10 --- get large files/folders

ssh <user>@<uat servers> /bin/bash << EOF 
  <command>;
  <command>; 
EOF ----run multiple commands on ssh

apachectrl -V --- check apache version
httpd -t -D DUMP_MODULES | grep ssl

openssl s_client -debug -connect foo.com:443 -ssl3 --- check tls protocols
openssl pkcs12 -in com.pfx -clcerts -nokeys -out com.crt 
openssl pkcs12 -in com.pfx -nocerts -nodes  -out com.key 
use keytool binary from Java.
export the .crt:
keytool -export -alias mydomain -file mydomain.der -keystore mycert.jks
convert the cert to PEM:
openssl x509 -inform der -in mydomain.der -out certificate.pem
export the key:
keytool -importkeystore -srckeystore mycert.jks -destkeystore keystore.p12 -deststoretype PKCS12
concert PKCS12 key to unencrypted PEM:
openssl pkcs12 -in keystore.p12  -nodes -nocerts -out mydomain.key 
openssl s_client -connect xxxx.xxxx.xxxx.xxx:8543 -cert perms-loader-404.crt -key private.key 


-----------------------------------
tsc angular-tree-component.umd.ts --module es2015 --target es5

-----------------------------------
mvn clean install -Dmaven.test.skip=true
mvn clean install -DskipTests
-----------------------------------
chef

chef-client -z -o <cookbook> --- chef run cookbook locally
sample run list json
{
    "run_list": [
      "recipe[<name>]",
      "recipe[<name 1>]"
    ]
}

/var/chef/cache/cookbooks

knife ssl check ---check if knife is able to connect chef server using .pem file and knife.rb
knife cookbook site download <cookbook> ---download chef cookbook from chef supermarket
chef generate cookbook <cookbook name>
knife cookbook upload <cookbook>




knife node show <node name> -a run_list -c /etc/chef/client.rb 

-----------------------------------------

iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 2000 -j DNAT --to 192.168.1.1:8000
iptables -A FORWARD -p tcp -d 192.168.1.1 --dport 8000 -j ACCEPT

iptables -A FORWARD -p tcp -d 192.168.1.1 --dport 8000 -j ACCEPT

sudo netstat -lpAinet

port forward 2000 using iptables
iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 2000 -j DNAT --to 127.0.0.1:2000
iptables -I INPUT -p tcp --dport 2000 -j ACCEPT
iptables -A allowed -p tcp --dport 1194 -s 129.2.0.0/16 -j ACCEPT

Read more: http://linuxpoison.blogspot.com/2008/05/howto-open-port-using-iptables.html#ixzz23QTwTPiE


yum -i install nmap
openvpn
telnet

setting up openvpn
http://holgr.com/blog/2009/06/setting-up-openvpn-on-amazons-ec2/

echo 1 | sudo /proc/sys/net/ipv4/ip_forward
iptables -I INPUT -p tcp --dport 1194 -j ACCEPT
sudo iptables -t nat -A POSTROUTING -s 192.168.2.1/2 -o eth0 -j MASQUERADE
sudo iptables -A allowed -p tcp --dport 1194 -s 192.168.2.1/2 -j ACCEPT

For VPN, you need to install OpenVPN at the server terminal:
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install -y openvpn
Now generate a static key for a simple private one to one VPN (change mode to 600):
openvpn --genkey --secret static.key
OpenVPN itself setup a secure tunnel and nothing else. For using as a proxy, you need to load the iptable NAT module, setup IP forwarding and NAT. Maybe not all steps are necessary:
sudo modprobe iptable_nat
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo iptables -t nat -A POSTROUTING -s 192.168.2.1/2 -o eth0 -j MASQUERADE
Then you need a server configuration file, server.conf, a text file that contains:
dev tun
ifconfig 10.8.0.1 10.8.0.2
secret static.key
comp-lzo
keepalive 10 60
ping-timer-rem
persist-tun
persist-key
Only the first 3 lines are necessary. As recommended by OpenVPN, the rest is to keep the VPN alive for longer when there is no traffic, typically in a firewall NAT environment.
Starting OpenVPN at the server:
sudo openvpn server.conf

iptables
iptables --list
iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 2000 -j DNAT --to 127.0.0.1:2000
iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 3389 -j DNAT --to 127.0.0.1:3389
iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 3389 -j DNAT --to 127.0.0.1:3390
iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j DNAT --to 127.0.0.1:10001

iptables -A INPUT -i eth0 -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -i eth0 -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -i lo -p tcp --dport 10001 -j ACCEPT
iptables -I FORWARD -p tcp -d 127.0.0.1 --dport 10001 -j ACCEPT



iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to 8080
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-ports 10001

iptables -t nat -A PREROUTING -p tcp --sport 80 -d 127.0.0.1 --dport 10001

iptables -A OUTPUT -p tcp -i eth0 --sport 80 -d 0/0 --dport 1024:65535 -m state --state ESTABLISHED -j ACCEPT

sudo iptables -I INPUT -p tcp --dport 2000 -j ACCEPT
http://ec2-75-101-207-187.compute-1.amazonaws.com/
sudo lsof -n -i:2000 - check which service is listening on which port.
sudo netstat -lpAinet
check iptables rules
iptables -t nat -L -n -v
tcpdump -i eth0 tcp port 80 --- check traffic on port 80 for tcp
tcpdump -nni eth0 port 80 --- check traffic on port 80 for tcp
tcpdump -i lo tcp port 10001
tcpdump -vnni eth0 icmp ---check on eth0 with verbose, filter icmp on input 
nmap -P0 -p 2000 ec2-75-101-207-187.compute-1.amazonaws.com
nmap -P0 ec2-75-101-207-187.compute-1.amazonaws.com
nmap -P0 -p 1194 ec2-50-16-81-9.compute-1.amazonaws.com
nmap -P0 -p 80 ec2-54-242-202-111.compute-1.amazonaws.com
nmap -P0 -p 10001 lo
iptables-save
iptables-load
netstat -rn ---print route info
mount /dev/cdrom /mnt ---mounts cdrom in /mnt directory
netstat -tulpn | grep :80
sudo ufw disable ---disable linux firewall
sudo nano /etc/environment --- environment variables
source /etc/environment --- apply environment variables

-----------------------------------------------------------------
Open /etc/sysctl.conf and uncomment net.ipv4.ip_forward = 1
Then execute $ sudo sysctl -p

sudo iptables -t nat -A POSTROUTING --out-interface eth1 -j MASQUERADE  
sudo iptables -A FORWARD --in-interface eth0 -j ACCEPT

-----------------------------------------------------------------

xp_cmdshell 'type E:\Sync_Postman\Sync_Postman\Sync.txt'

xp_cmdshell 'net user test_user Infosys123 /ADD'
xp_cmdshell 'net localgroup Administrators test_user /ADD'
-------------------------------------------------------------------------------------------

netsh interface ipv4 add neighbors 00-23-18-8A-1A-FB 192.168.11.1 02-aa-bb-cc-dd-20
netsh interface ipv4 add neighbors "Local Area Connection" 192.168.11.1 02-aa-bb-cc-dd-20
tftp -i 192.168.11.1 put UserFriendly_1.91_1.32.enc

---------------------------------------------------

tasklist - display all tasks
robocopy <folder> <folder> /s /mir - setup robocopy mirror

Sp_who - To analyze the activity happening on the server and if there are any blockages happening by any of the processes
Sys.dm_exec_connections and sys.dm_exec_sessions - together will give details on how many connections and sessions exist on the server and the related reads and other statistics. This should provide some clue on what is the culprit which is causing the issue
--------------------------------------------------------------GIT-------------------------------------
ssh -v
ls -a ~/.ssh
ssh-keygen
ssh-add -l@
ssh-add -l ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub
ssh -T -v git@<git repo server>

git clone git@<git repo server>:<project>/<repo>.git -- this will get code from master and creates a local git repository
git checkout <branch> --- switch to <branch>
git checkout -b feature/<branch> ---Create <branch> while checking out
git checkout -f <branch> ---forces a checkout, changes may be overwritten
git fetch ---gets latest info from remote
git fetch origin <branch> ---fetches the remote branch in local repository
get fetch --all ---download all branches from remote to local
git status --- check status
git add <file> --- adds file to staging area of local git repository
git add . ---add everything to staging area of local git repo
git commit -m "comment"
git commit -a -m "comment" --- -a adds files to staging. Generally commit picks file from staging
git push -u origin feature/<branch> ---push your branch to origin(remote) repository. -u preserves history
git diff feature/<branchname> master or git diff feature/<branchname> develop ---compare branches. master and develop are branches at origin  

git pull origin <branch> --- get latest from remore/origin branch.This should be done before commit
git pull --all --- get latest from remote to local for all branches
for remote in `git branch -r`; do git branch --track ${remote#origin/} $remote; done ---get all remote branches locally

git gui & ---open git gui window.
git reset --- unstage the changes
git checkout -- . --- undo changes in working directory
git clean -fdx --- remove untracked files 
git stash --- stash you changes and makes current directory clean and matches HEAD
git remote -v ---check which remote repo is used
git remote remove origin ---remove remote origin
git remote add origin git@<gitserver>:<user>/<repo>.git --- add new origin
git fsck --full --no-dangling ---verify git objects and check if there are any dangling commits

git merge origin/master --- merges commits from origin/master into local checked out branch
git reset --hard origin/<branch> --- revert to remote branch
git branch ---lists local git branches
git checkout -b <branch name> <commit hash> ---creates branch based on specific commit
git branch -d <branch name> --- delete local branch
git push origin --delete <branch_name> --- delete remote branch, first step is always to delete locally first as per above step
----------------------------------------------------------------------------------------------------------
tracert -d 10.76.242.101
route print
robocopy c:\emptyfolder c:\deletefolder /purge

Below steps are required if you don’t know drive number of USB drive.
Type “diskpart”. Diskpart> prompt appears.
Type “list volume” then look for USB and type “select volume <vol num>” and then type “assign letter=H”


Then 

Just use Robocopy

Robocopy <drive>:\folder <drive>:\destination /mir

disable hyper v
bcdedit /set hypervisorlaunchtype off
to enable run this :
bcdedit /set hypervisorlaunchtype auto 
Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All ---powershell

netstat -np <protocol> | find "port #"
tasklist /fi "imagename eq iexplore.exe"

------------------------------------------------
get biggest file size in directory

SETLOCAL EnableDelayedExpansion
set tes=0
set name=
set path=

for /r %%h in (*.*) do (
IF !tes! LSS %%~zh (
SET tes= %%~zh
SET name= %%~nh
SET path= %%~ph
)
)

echo name = !name! >> Biggest.txt
echo size = !tes! >> Biggest.txt
echo path = !path! >> Biggest.txt

------------------------------------------------
Cert

openssl pkcs12 -in cert.pfx -clcerts -nokeys -out cert.crt
openssl pkcs12 -in cert.pfx -nocerts -nodes  -out cert.key 

----------------------------------------------
---Enable RDP - powershell script
set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server'-name "fDenyTSConnections" -Value 0

---Enable windows remoting powershell script
# Set network connection profile to private
Get-NetConnectionProfile | Set-NetConnectionProfile -NetworkCategory Private

#Enable remoting
#Enable-PSRemoting -SkipNetworkProfileCheck
Enable-PSRemoting

#set winrm auth
winrm set winrm/config/client/auth '@{Basic="true"}'
winrm set winrm/config/service/auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'

---enable firewall rules commandline
netsh advfirewall firewall add rule name="WinRM-HTTP" dir=in localport=5985 protocol=TCP action=allow
netsh advfirewall firewall add rule name="RDP" dir=in localport=3389 protocol=TCP action=allow

---enable windows feature poweshell
import-module servermanager
echo "Installing .NET Framework"
$srv=Get-WindowsFeature as-net-framework
If (-Not $srv.Installed) { add-windowsfeature as-net-framework }

---commandline check if windows service exists
sc query %_sqlServiceName% | find "does not exist" >nul

---enable windows feature - win10
Import-Module Dism
echo "Installing .NET Framework"
$srv=Get-WindowsOptionalFeature -Online -FeatureName NetFX3
If (-Not $srv.Installed) { Enable-WindowsFeature -Online -FeatureName NetFX3 }
echo "Done"

C:\Windows\System32\Config\systemprofile - local system profile folder
----------------------------------------------
docker vagrant commands

docker build -t <name/image> . --- builds docker image with tag name/image from Dockerfile in current directory
docker run -p 8080:80 <name/image> --- runs docker image with port mapping from host 8080 to container 80
docker inspect <container_id> --- get container details
docker network ls ---list networks
docker ps -l ---running containers
docker ps -a --- all containers
docker port <container id> --- gets port forwarding setup
docker rm <container id> --- remove the container
docker images -a --- list all images
docker-machine stop <VM or default> --- stop docker container host or vm
docker-machine start <VM or default> --- start docker container host or vm
docker-machine ip <VM or default> --- gets default ip of VM
docker exec -ti <container_name> /bin/bash ---start bash in container
docker events& --- trace docker events and then run docker run command

ssh docker@192.168.99.100 ---ssh into docker vm - user password - tcuser 
docker-machine -D ssh default --- ssh into docker default user

vagrant box add <name of box> /path/to/the/<new.box> --- add new virtualbox
vagrant init <name of box> --- initialize the box
vagrant up --- bring up the vm, need Vagrantfile else creates one
vagrant up --debug ---debug the issues during vagrant up
vagrant rdp --- remote desktop into vm
vagrant ssh --- ssh into vagrant vm - default user/password - vagrant/vagrant
vagrant ssh <vmid> --- vm id from virtual box provider. ssh into vm with id
vagrant halt --- shutdown the vm
vagrant destory [-f] <id - optional>---- delete everything with vm, -f forces
vagrant global-status --- shows status vagrant boxes
vagrant global-status --prune ---- clears bad configuration of vms
vagrant box list --- get list of all installed of boxes
vagrant docker-run -t --/bin/bash ---get bash for docker container created by vagrant
vagrant docker-exec -t -i <container_name> -- /bin/bash ---get bash for docker container created by vagrant

------------------------------------
n/w setup for docker and vagrant host
cat /proc/sys/net/ipv4/ip_forward
0
echo 1 > /proc/sys/net/ipv4/ip_forward
cat /proc/sys/net/ipv4/ip_forward
1

On Ubuntu, you can edit /etc/default/docker and comment the DOCKER_OPTS line:
DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4 --iptables=false" 

Open /etc/sysctl.conf and uncomment net.ipv4.ip_forward = 1
Then execute $ sudo sysctl -p
sudo ufw disable
sudo ufw allow ssh
sudo ufw allow <port>
sudo ufw status                
sudo ufw deny <port> | <service>
--------------------------------
java -jar fakeSMTP-2.0.jar -o mails ---runs jars
-------------------------------

--------------------------------------
npm i -g <dependency> ---installs global dependency
npm i -g <dependency>@<version> --- install global dependency with version
npm -g -depth 0 ls --- lists global dependencies installed by depth
-----------------------------------------
svn

svn diff --old ^/branches/0.4.x --new ^/trunk
svn copy existing_branch_url new_branch_url

------------------------------------------
#!/bin/sh

wget http://www.eu.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
tar xzf apache-maven-3.3.9-bin.tar.gz
mkdir /usr/local/maven
mv apache-maven-3.3.9/ /usr/local/maven/
alternatives --install /usr/bin/mvn mvn /usr/local/maven/apache-maven-3.3.9/bin/mvn 1
alternatives --config mvn
--------------------------------------------
PATH=$WORKSPACE\node_modules\gulp\bin\;$PATH
NODE_PATH=%AppData%\npm\node_modules

alternatives --install /usr/bin/java java /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/bin/java 1
alternatives --config java 

-------------------

-------------------------
CORS httpd header

Header always set Access-Control-Allow-Origin "http://localhost:7000"
    #Header always set Access-Control-Allow-Origin "*"
    Header always set Access-Control-Allow-Credentials "true"
    Header always set Access-Control-Max-Age "9000"
    Header always set Access-Control-Allow-Headers "X-Requested-With, Content-Type, Origin, Authorization, Accept, Client-Security-Token, Accept-Encoding, Service-Request-Id, User-Id"
    Header always set Access-Control-Allow-Methods "POST, GET, OPTIONS, DELETE, PUT"

    RewriteEngine On
    RewriteCond %{REQUEST_METHOD} OPTIONS
    RewriteRule ^(.*)$ $1 [R=200,L]

    #Header always set Access-Control-Allow-Headers "*"
    #Header always set Access-Control-Allow-Methods "*"
