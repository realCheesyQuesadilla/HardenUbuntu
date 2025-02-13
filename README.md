# HardenUbuntu
## As found on HackForums
#### 1: Update & Upgrade Like It’s a Religion

Old software = vulnerable software, so "PLEASE DADDY HACK ME" is written all over it.

Pop open a terminal and type:

``` sudo apt update && sudo apt upgrade -y ```

Try to stay updated and do this often, automate it if you’re lazy:  
```
sudo apt install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```  
2: Lock Down Your Firewall (UFW)  

Ubuntu ships with "ufw" (uncomplicated firewall) already there, just not enabled so turn it on with:

```
sudo ufw enable
```

You can also check the status of it with input:

```
sudo ufw status verbose
```  
3: Use AppArmor for Process Containment

Ubuntu comes with apparmor, which helps prevent apps from going rogue. To check what's loaded type:

```
sudo aa-status
```  

if you see 'enforce' on a bunch of things, you’re good otherwise, enable it with:

```  
sudo systemctl enable apparmor
sudo systemctl start apparmor
```

Also little side not here - you can create custom profiles for apps that need hardening, but that’s another topic

4: Private DNS + No Logging

By default, I believe ubuntu uses your ISP’s DNS (no bueno papi) so change it to something like Cloudflare’s or a privacy-respecting provider.

Go edit resolv.conf:
```
sudo nano /etc/resolv.conf
```

Replace everything with:

```  
nameserver 1.1.1.1
nameserver 9.9.9.9
```  
Save then exit - then make it immutable so ubuntu doesn’t overwrite it.  

```
sudo chattr +i /etc/resolv.conf
```  
(if you ever need to change it back) then:
```  
sudo chattr -i /etc/resolv.conf
```  
5: Create a Low-Privilege User for Daily Use

Running everything as an admin (root or sudo) is uh, yeah not needed all the time (if you know you know) make a user with fewer privileges:

```  
sudo adduser safeuser
sudo usermod -aG sudo safeuser
```  
Leave root alone.

6: Disable USB Auto-Run

Malware loves USB sticks (pause), so disable auto-mounting by editing this file

```
sudo nano /etc/fstab
```
At the bottom add this line:

```
none /media usbfs noauto,user,exec,noatime 0 0
```


Save it -> Exit -> Reboot

7: Check for Rootkits

First install chkrootkit. Protip - (it’s already in the ubuntu repos)

```  
sudo apt install chkrootkit
sudo chkrootkit
```

You should be able to tell if something looks a little sus - (can dive more into this at a later time of if anyone is specifically curious)

8: Disable Telemetry Reporting

```  
sudo ubuntu-report --disable
```  
Say what you want, those crazy bastards at Canonical are some funny folks. Best to keep them out of the loop data wise. (Just a precaution)

9: Block Crash Reporting

```
sudo systemctl stop apport.service
sudo systemctl disable apport.service
```

10: Disable WebRTC (It Leaks Your Real IP)

```  
> If you use Firefox, go to about:config, search for media.peerconnection.enabled, and set it to false
> If you use Chromium-based browsers, use an extension like WebRTC Leak Prevent
```  
11: Spoof your MAC Address

Unfortunately your real MAC can be used to track you on networks so to mitigate this a bit we're going to install 'macchanger'

```
sudo apt install macchanger
```  
Set it to randomize at every reboot:

```  
sudo nano /etc/network/interfaces
```
Then add this before the iface section:

```
pre-up macchanger -r eth0
```  
Save -> Reboot

12: Disable Unnecessary Services

If you end up seeing something boof, get rid of it with these commands:

First - see what's running

```  
systemctl list-units --type=service --state=running
```  
Then, stop and disable anything that looks like nonsense with:

```
sudo systemctl stop "servicenamehere"
sudo systemctl disable "servicenamehere"
```  
Probably worth disabling bluetooth + printing while you're there:

```
sudo systemctl disable cups
sudo systemctl disable bluetooth
```

13: Defense against Bruteforce Login Attempts

'fail2ban' is your best friend for this one. Install it with

```  
sudo apt install fail2ban
```  
Get it started and enable it

```  
sudo systemctl start fail2ban  
sudo systemctl enable fail2ban
```
Note: this will automatically block too many failed SSH login attempts - but you can configure /etc/fail2ban/jail.local for more aggressive settings if you feel so inclined.

14: Kill Zeitgeist (Activity Tracking)

If you didn't already know - ubuntu keeps a history of files you open… lol right?

Let's fix that permanently with:

```
sudo apt remove zeitgeist-core
rm -rf ~/.local/share/zeitgeist
```  
Unless for some reason you'd like to keep it on, then you can limit what it logs with

```
gnome-privacy-panel
```  
15: Block Network Tracking & Ads

Go to your hosts file and block a couple common trackers. (There are obviously more - this is for example)

```  
sudo nano /etc/hosts
```   
then add these + more
```  
0.0.0.0 google-analytics.com  
0.0.0.0 doubleclick.net  
0.0.0.0 ads.youtube.com
```
Save -> Exit

e you guys enjoyed!
