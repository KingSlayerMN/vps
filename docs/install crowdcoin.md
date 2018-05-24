
---

## **Pre-requisites:**
- new VPS Ubuntu 16.04 (DON'T TRY THIS SCRIPT WITH OTHER MASTERNODE COINS INSTALLED or accept unpredictable results)
- Make a note of your **VPS IP Address**
- follow the official setup guide to page 2 (https://crowdcoin.site/guides/QUICK_CROWDCOIN_MASTERNODE_SETUP.pdf)
to get the **masternode private key** and the **TxID** and **TxIndex** before starting this guide.

__Please copy and paste (or type) everything exactly as written behind "$" but not the "$"!__


---

__Login into your VPS with the root privileges.__

### Step 0) 
To update your VPS with the latest software run this command: 
```sh 
$ apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y
```
if prompted for `tools.conf (Y/I/N/O/D/Z) [default=N] ?` just press enter
once finished reboot your VPS and  
**Login with root privileges and continue**
		
### Step 1) 
Download the install script and change into the vps directory with: 
```sh
$ git clone https://github.com/KingSlayerMN/vps.git && cd vps
```
### Step 2) 
This command will start the installation of your masternode   
```sh
$ ./install.sh -p crowd -c 1 -n 4 -s
```
Here is a quick explaination about the parameter
```diff
-p = project to install - we want crowd
-c = number of masternodes to install (yes, you could install multiple MN's if you have multiple IP's)
-n = network 4=IPv4 6=IPv6 (IPv6 is not supported by crowdcoin) 
-s = install sentinel
```
### Step 3) 
Once the install script has finished open the configuration file with
```sh
$ nano /etc/masternodes/crowd_n1.conf
```
add your IP address here `bind=[#NEW_IPv4_ADDRESS_FOR_MASTERNODE_NUMBER:::1]:12875` so it looks like:
```css
bind=44.33.22.11:12875
```
now paste your masternode private keys to `masternodeprivkey=HERE_GOES_YOUR_MASTERNODE_KEY_FOR_MASTERNODE_crowd_1` so it looks like:
```css
masternodeprivkey=93HaYBVUCYjEMeeH1Y4sBGLALQZE1Yc1K64xiqgX37tGBDQL8Xg
```
To ensure your wallet is getting synced we should add some addnodes.  
At https://www.cryptopia.co.nz/CoinInfo, type `crc` in the search field and click on "Connections" of Crowdcoin.  
Select the addnodes shown on the left into the clipboard, paste them to the end of the file.  
Now you can save the file with:
```sh
ctrl+x 
press y 
and enter
```		
### Step 4) 
Now it is time to activate the masternode 
```sh
$ activate_masternodes_crowd
``` 
and wait for the new MN to be fully synced. We can monitor the progress with 
```sh
watch crowdcoin-cli -conf=/etc/masternodes/crowd_n1.conf mnsync status
```
When the wallet is fully synced the output will look like this:
```css
{
  "AssetID": 999,
  "AssetName": "MASTERNODE_SYNC_FINISHED",
  "Attempt": 0,
  "IsBlockchainSynced": true,
  "IsMasternodeListSynced": true,
  "IsWinnersListSynced": true,
  "IsSynced": true,
  "IsFailed": false
}
```
to quit press 
```sh
$ ctrl + c
```
### Step 5) 
Continue with the official setup guide at page 6 "STARTING YOUR MASTERNODE (Windows)" and come back here.  
Just make sure the collateral payment has gotten __15 confirmations__ before you try to start the masternode.  
(you can safely omit the chapter "Test and Troubleshooting" in the official setup guide)  
		
### Step 6) 
Now it is time to activate Sentinel (It is important to have this in one line)
```sh
$ export SENTINEL_CONFIG=/usr/share/sentinel/crowd1_sentinel.conf; /usr/share/sentinelenv/bin/python /usr/share/sentinel/bin/sentinel.py
```
### Step 7) 
If it works without error, you can add that command to crontob to run every minute: 
```sh
$ crontab -e
```
and select 2 (nano) when ask for the editor to use.
Place the string below in one line at the bottom of crontab:
```sh
* * * * * export SENTINEL_CONFIG=/usr/share/sentinel/crowd1_sentinel.conf; /usr/share/sentinelenv/bin/python /usr/share/sentinel/bin/sentinel.py 2>&1 >> /var/log/sentinel/sentinel-cron.log
```
and save crontab with:
```sh
ctrl+x 
press y 
and enter
```		
### Step 8) 
Now we are making the command line interface (crowdcoin-cli) easier accessible by adding the string below to your .bashrc 
```sh
$ echo "alias crc1='crowdcoin-cli -conf=/etc/masternodes/crowd_n1.conf $1'" >> ~/.bashrc
```
and activate it with:
```sh
$ . ~/.bashrc
```

# Done! Enjoy your new masternode

---
### Some basic information about this setup.

Configuration files are stored in: /etc/masternodes  
Data are stored in: /var/lib/masternodes  
This setup will run the daemon as a service with an unprivileged user account  
which does not even can be used to login to your VPS.

---

Because of the alias we have created you can now type the commands as shown below 
```sh
$ crc1 masternode status
$ crc1 getinfo
$ crc1 mnsync status
$ crc1 getblockcount
```

---

Use these commands to start, stop and review the status of the service.

| Command | Action |
| ------ | ------ |
| $ service crowd_n1 start | will start the masternode as a service |
| $ service crowd_n1 stop | will stop the masternode service |
| $ service crowd_n1 status | will show the status and some statistics of the service |


The advantage of running the masternode as service is that it will be monitored  
by systemd and automatically restart if it ever crashes.  
Also in case of a reboot of your VPS the masternode will be started automatically.

---

Only if you wish the make use of watch you need to include the path to the conf-file together with crowdcoin-cli  
The tool `watch` does not use the alias therefore you have to add the path to the config file
```sh
$ watch crowdcoin-cli -conf=/etc/masternodes/crowd_n1.conf mnsync status
```

---
To check the sync status of your masternode
```sh
$ crc1 mnsync status` output
``` 
```css
{
  "AssetID": 999,
  "AssetName": "MASTERNODE_SYNC_FINISHED",
  "Attempt": 0,
  "IsBlockchainSynced": true,
  "IsMasternodeListSynced": true,
  "IsWinnersListSynced": true,
  "IsSynced": true,
  "IsFailed": false
}
```
press ctrl+c to exit

---
To check if the sentinel is running every minute you can run this command:
```sh
$ tail -f /var/log/sentinel/sentinel-cron.log
``` 
press `ctrl + c` to quit
or 
```sh
 cat /var/log/syslog |grep cron
 ```
 Either way you see that cron is executing sentinal every minute.
 
---
Should you wish to get more a verbose output of the sentinel at the command line you can run in one line
```sh
$ export SENTINEL_CONFIG=/usr/share/sentinel/crowd1_sentinel.conf; SENTINEL_DEBUG=1 /usr/share/sentinelenv/bin/python /usr/share/sentinel/bin/sentinel.py
```
Should you have questions you are welcome to post them at our discord 
https://discord.gg/EhTfxGd
