What
======
These steps will install ddclient on your Ubuntu server and allow it to update any Cloudflare domains you want with your current dynamic IP address. The computer on which you install this script should be one that remains at the same location as the dynamic IP you'd like to link.

Why
======
Let's say you want to host a VPN server, a website, your files or anything else at home. Unless you have a static IP address, which in some cases you can't get without signing up for a business account, you'll need a way to associate your dynamic IP address with a domain name so your service doesn't get interrupted when your assigned public IP changes. 

Which Technologies
======
This guide focuses on installing [ddclient](https://github.com/wimpunk/ddclient) on Ubuntu (I used 16.04 server) and integrating it with Cloudflare. There are many other installation methods, so check out the ddclient repo for a full list.

How
======
Ssh into your Ubuntu machine and become the root user:
```bash
ssh username@ipaddress.of.machine -p port_you_set_default_is_22
sudo su
```
Next clone/download the ddclient repo from Github. I used the `/tmp` directory, but you can use whatever you want. Move to that directory:
```bash
cd /tmp
```
Clone the repo into the directory you chose:
```bash
git clone https://github.com/ddclient/ddclient.git
```
Next enter the new ddclient directory and copy ddclient to `/usr/sbin`:
```bash
cd /tmp/ddclient
cp ddclient /usr/sbin/
```
Make a couple directories that are missing:
```bash
mkdir /etc/ddclient
mkdir /var/cache/ddclient
```
Once those directories are made, copy the default config to use as your starting place:
```bash
cp sample-etc_ddclient.conf /etc/ddclient/ddclient.conf
```
Open that file for editing:
```bash
vim /etc/ddclient/ddclient.conf
```
Once in Vim, you want to hit `i` to enter insert mode, at which point you'll be able to enter text. The first thing to do is to remove the `#` at the beginning of this line:
```
use=web, web=checkip.dyndns.org/, web-skip='IP Address’
```
Now search for the Cloudflare section by hitting the escape key to exit insert mode and then typing a `/` and then `cloudflare` an hitting enter. That should bring you to the section. Once there, set it up by first removing the `#` from beginning of each line and then changing everything that starts with a `$` below to your personal details:
```
protocol=cloudflare,        \
zone=$domain.tld,            \
login=$your@cloudflare.login,     \
password=$your_api_key             \
ttl=1                       \
$domain.tld,$subdomain1.domain.tld,$subdomain2.domain.tld
```
A couple notes about the lines above: make sure the password line does not end in a comma. You can update as many domains and subdomains as you'd like by placing them on the last line of the config block but remember they will all be set to the same IP.  

Hit escape again to exit insert mode and hit `:` and type `wq` then hit enter to save and exit vim.  

Now you're ready to test your setup with the following command:
```bash
ddclient -daemon=0 -debug -verbose -noquiet
``` 
It may (and probably will) fail and that's just fine; there are a variety of errors that I discuss at the bottom of this readme. Each time you correct an error, run the above command to test whether it corrected that specific error.  

Once everything is working, perform one final test by changing one of your Cloudflare DNS entries of a domain you want associated with your dynamic IP to something that isn't your dynamic ip, then run the test command above and verify that it did, in fact, change it back. Once you see that you know you're good.  
 
 Now you're ready to set ddclient to run at startup and launch it on boot. If you've changed directories, go back to where you initially cloned the ddclient repo, for me that was `/tmp/ddclient` by entering: 
 ```bash
 cd /tmp
 ```
 Copy the Ubuntu launch script to your `/etc/init.d` folder and enable it on boot:
 ```bash
 cp sample-etc_rc.d_init.d_ddclient.ubuntu /etc/init.d/ddclient
 update-rc.d ddclient
 ```
 Finally, start the service manually the first time:
 ```bash
 service ddclient start
 ```
 In order to double check that everything is set up correctly, run `ddclient -query` and make sure that this line is correct: `use=web, web=dnspark address is your.public.ip`. In order to access the query mode, press control+c. After the script has run a couple times you'll also want to check the "mail" that the script is creating because there may be one more issue. Open the mail by running:
 ```bash
 sudo vim /etc/mail/root
 ```
 If you see an error that looks like this:
 ```bash
 WARNING:  file /var/cache/ddclient/ddclient.cache, line 3: Invalid Value for keyword 'ip' = ''
 ```
 you may need to update the version of ddclient. The error has been fixed in more recent versions, but [this post](http://askubuntu.com/questions/64219/why-is-ddclient-giving-me-an-invalid-ip-error-when-trying-to-update-dynamic-dn) explains a bit about what's going on.
 
 
Possible Errors + Their Solutions
======
 
Can't locate Data/Validate/IP.pm in @INC (you may need to install the Data::Validate::IP module)
------
```bash
apt-get install libdata-validate-ip-perl
```  

FATAL: Error loading the Perl module JSON::Any needed for Cloudflare update.
------
```bash
apt-get install libjson-any-perl
```

Can't exec "sendmail": No such file or directory at /usr/sbin/ddclient line 1584.
------
```bash
apt-get install sendmail
```

WARNING: local host name (server) is not qualified;
------
This is an error that sendmail may give you on install. If you want, you can change your hostname to be something like server.domain.com, which would be qualified. This is not necessary, though.  

Error loading the Perl module IO::Socket::SSL needed for SSL connect.
------
```bash
apt-get install libio-socket-ssl-perl
```

Final Thoughts
======
That's it! Your setup should be complete. Please let me know if you encounter any errors that aren't listed below and submit a pull request if you have any corrections, additions or improvements.  

Matt
