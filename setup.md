#General instruction for manual setup
assumes UBOS Arch Linix for ARM7
needs to access archarm respositories [core][extra][community]in pacman.conf

Overview
-add repositories if needed
-Setup new users
-install namecoind
-install PowerDNS
-Install DNSChain
-setup unit files for systemd services


sudo nano /etc/pacman.conf and add

[core]
Server = http://co.us.mirror.archlinuxarm.org/$arch/core

[extra]
Server = http://co.us.mirror.archlinuxarm.org/$arch/extra

[community]
Server = http://co.us.mirror.archlinuxarm.org/$arch/community

Create users
sudo useradd -m -G wheel -s /bin/bash namecoin
sudo useradd -m -G wheel -s /bin/bash dnschain
sudo useradd -m -G wheel -s /bin/bash powerdns

set password for dsnchain user
sudo passwd dnschain

set password for powerdns user
sudo passwd powerdns

set password for namecoin user
sudo passwd namecoin

Logout and log back in as namecoin


Install namecoin

cd ~
download namecoind (armhf version) into /home/namecoin
wget http://bitseed.org/device/namecoin/namecoind
mkdir .namecoin
wget http://bitseed.org/namecoin/namecoin.conf

edit namecoin.conf to change rpcpassword
nano /home/namecoin/.namecoin/namecoin.conf

cd ~
start namecoind as daemon
./namecoind -daemon

Allow about 24 hours to fully sync blockchain

Check if blockchain is synced:
./namecoind getinfo

test namecoin resolution
./namecoind name_show d/okturtles

if you see "error: {"code":-101,"message":"loading block index"}"  then not synced yet.
if synced it will return details
-----------------------------------------------------------------------

Install PowerDNS pdns_recursor

add user to sudoers
sudo nano /etc/sudoers
and add the this line:    powerdns ALL=(ALL) ALL
    ^X to save

sudo pacman -S dnsutils   #installs dig

as powerdns user

sudo pacman -S powerdns-recursor

(or wget http://bitseed.org/namecoin/pdns_recursor)

configure

sudo nano /etc/powerdns/recursor.conf

and add the following:
forward-zones=bit.=127.0.0.1:5333,dns.=127.0.0.1:5333,eth.=127.0.0.1:5333,p2p.=127.0.0.1:5333
export-etc-hosts=off
allow-from=0.0.0.0/0
local-address=0.0.0.0
local-port=53

run as sudo
sudo pdns_recursor

Test
sudo rec_control ping   (should reply with pong)

Test
dig @127.0.0.1 okturtles.org

flags should include "Answer: 1"  and it should show an ip address for okturtles.org 192.184.93.146

----------------------------------------------------------------

Install DNSChain

sudo pacman -S git
sudo pacman -S python2  make  gcc

Install nodejs
sudo pacman -S nodejs
Install npm
sudo pacman -S npm

Install DNSChain and supporting tools
sudo npm install -g coffee-script
sudo npm install -g grunt-cli

sudo npm install -g dnschain

run dnschain is gen key (or gen key manually)
move keys
cp key.pem /home/dnschain/.dnschain/
cp cert.pem /home/dnschain/.dnschain/

run dnschain in background
nohup dnschain >/dev/null 2>&1 &
(stop with: killall node)

Test
dig @127.0.0.1 okturtles.bit

test from another machine
dig @ip.of.dns.server okturtles.bit

----------------------------------------------------------------------
