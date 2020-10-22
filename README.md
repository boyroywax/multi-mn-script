# Multi masternode setup using ipv6

## Test Env
* Digital Ocean 1GB Server
* Ubuntu 16.04 x64

## Script is hit-or-miss
## Run multi-mn-script
* https://docs.fix.network/english/fix-masternodes/multiple-masternode-setup-on-one-vps
```shell
# Download the script
git clone https://github.com/boyroywax/multi-mn-script && \
cd multi-mn-script
```
Run the install in tmux (better?)
```shell
# Run the script
tmux
./install.sh -p pawcoin -c 2 -n "6"
```
Allow script to run - takes a while and may crash.

Enter the masternode private keys into the conf.  Files located at ```/etc/masternodes```

Start the masternode service script.
```shell
/usr/local/bin/activate_masternodes_pawcoin
```

Check to see if the services are running
```shell
systemctl status pawcoin*
```


---
## Manual install
## Create swapfile
* https://medium.com/@CaveSpectre/multiple-masternodes-one-coin-one-vps-a72832ad5f0e
I'm pretty sure there are other ways of doing this...

```shell
# Create swap file
dd if=/dev/zero of=/swapfile bs=1024 count=6291456 && \
mkswap /swapfile && \
chmod 600 /swapfile && \
swapon /swapfile && \
echo "/swapfile swap swap defaults 0 0" >> /etc/fstab
```
## Add new ipv6 address
* https://www.digitalocean.com/docs/networking/ipv6/how-to/configure-additional-addresses/
```text
## /etc/network/interfaces.d/50-cloud-init.cfg
iface eth0 inet6 static
    address 2604:a880:2:d0::118c:1002/64
```

## Each Masternode needs unique rpcport
Create folder for the configs
```shell
mkdir /etc/masternodes
```
Create a config file for each masternode
```shell
nano pawcoin_n1.conf
```
Example config file:
```text
## /etc/masternodes/pawcoin_n1.conf

################################
# basic settings
################################
txindex=1
logtimestamps=1
listen=1
daemon=1
staking=0
gen=0
maxconnections=256
bind=[2604:a880:2:d0::118c:1002]:8322
port=8322

################################
# masternode specific settings
################################
masternode=1

################################
# createmasternodekey 
################################
masternodeprivkey=CREATEMASTERNODEKEYgoeshere

#############################
# optional indices
#############################
addressindex=1
timestampindex=1
spentindex=1

#############################
# JSONRPC
#############################
server=1
rpcuser=pawcoinrpc
rpcpassword=samplepasssword
rpcallowip=127.0.0.1
rpcport=8323
```

## Copy the blockchain directory
```shell
cd .pawcoin  && \
pawcoin-cli stop && \
sync && \
tar cf ../pawcoin.tar && \
cd .. && \
mkdir .pawcoin2 && \
cd .pawcoin2 && \
tar xf ../pawcoin.tar
```

## Start the nodes
```shell
# Start Masternode
/usr/local/bin/pawcoind -conf=/etc/masternodes/pawcoin_n1.conf -datadir=/root/.pawcoin -daemon
```

## Check the nodes
```shell
/usr/local/bin/pawcoind -conf=/etc/masternodes/pawcoin_n1.conf getinfo
```