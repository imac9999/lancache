LAN-Cache v1.3-unbound
==============

Based off work of https://gitlab.com/frag-o-matic/lan-cache
So credits go to Bruno Gysels and MultiPlay.co.uk for the base they made!

Come visit us @ www.cu-lan.be | www.gunsnbits.de | www.discoverpc.net/ | https://lanfest.intel.com/netwar

Facebook @ https://www.facebook.com/groups/434599530274923/?notif_id=1509019697563943

Twitter @ https://twitter.com/search?q=%40LAN_CACHE&src=typd

OS: Debian 9.4 amd64 (Stretch)

## Short Changelog
* 03-14-2018 - Minor changes in Readme.md to provide a more hassle free installation.
* 6-30-2017 saambd
    * Added missing } on line 56 of Microsoft conf    
* 8-03-2017 bn_
    * Added 2 Steam URLs to Unbound.Conf as posted in Multiplay Github
* 10-17-2017 bn_    
    * Added Sony DNS to Unbound.Conf as posted by Muffeee in issues
    * Added Glyph Support (Still needs testing)
* 10-25-2017 Nagilum99 & Nexusofdoom
    * Merged the pull request from nagilum99 to correct / standardize the layout (Issue #65)
    * Changed Steamconfig as problem and solution posted by Nexusofdoom (Issue #61)
* 10-28-2017 nexusofdoom
    * Added 2 Battlenet URLs to Unbound.Conf
    * Added New Origins URLs to Unbound.Conf
    * Did some magic in lancache-origin.conf to make it work :-)
* 12-4-2017 nexusofdoom
    * Added Warframe and ESO/Elder Scrolls Online to Config.

## Important!
If you already have an installation of nginx installed via apt-get install nginx, it is necessary that you remove it, the configuration files and all recommended packagaes via:
apt-get purge nginx
apt-get autoremove

Otherwise nginx may not start with lancache start script, instead run as wrong user and load /etc/nginx/nginx.conf.
It results in not proxying and leaving entries in /var/log/nginx/...

### Manual installation

If you want to install it manually, please follow the instructions below:

	0) Configure proper network interface in your /etc/network/interfaces file, go for static IP address, take notes about all IPs you'll assign, as you need to refer to them during this installation by A LOT!

    	1) Install the required utilities
	   apt-get install curl git unbound build-essential libpcre3 zlib1g-dev libreadline-dev libncurses5-dev libssl-dev httpry libudns0 libudns-dev libev4 libev-dev devscripts automake libtool autoconf autotools-dev cdbs debhelper dh-autoreconf dpkg-dev gettext pkg-config fakeroot libpcre3-dev -y

	2) Clone the git repo
	   git clone -b master http://github.com/imac9999/lancache

	3) Install nginx
	   curl http://nginx.org/download/nginx-1.13.4.tar.gz | tar zx
	   cd ngnix-1.13.4
	   ./configure --with-http_ssl_module --with-http_slice_module --with-file-aio --with-threads
	   make
	   make install

	4) Add the virtual interfaces (used for caching in nginx) to /etc/network/interfaces


	5) Create the user lancache
		adduser --system --no-create-home lancache
		addgroup --system lancache
		usermod -aG lancache lancache
	
	6) Just create the folders:
		mkdir -p /srv/lancache/data/blizzard/
		mkdir -p /srv/lancache/data/microsoft/
		mkdir -p /srv/lancache/data/installs/
		mkdir -p /srv/lancache/data/other/
		mkdir -p /srv/lancache/data/tmp/
		mkdir -p /srv/lancache/data/hirez/
		mkdir -p /srv/lancache/data/origin/
		mkdir -p /srv/lancache/data/riot/
		mkdir -p /srv/lancache/data/gog/
		mkdir -p /srv/lancache/data/sony/
		mkdir -p /srv/lancache/data/steam/
		mkdir -p /srv/lancache/data/wargaming
		mkdir -p /srv/lancache/data/arenanetworks
		mkdir -p /srv/lancache/data/uplay
		mkdir -p /srv/lancache/data/glyph
		mkdir -p /srv/lancache/data/zenimax
		mkdir -p /srv/lancache/data/digitalextremes
		mkdir -p /srv/lancache/data/pearlabyss
		mkdir -p /srv/lancache/logs/Errors
		mkdir -p /srv/lancache/logs/Keys
		mkdir -p /srv/lancache/logs/Access

	6.1) chown the folder:
		sudo chown -R lancache:lancache /srv/lancache

	7) Copy the conf folder and contents (where you originally git cloned it to in step 4) to /usr/local/nginx/conf/
		sudo cp -R ~/lancache/conf /usr/local/nginx/
    		
		7.1) Replace the proxy_bind variable with your primary IP address (not one of the virtual ones)

	8) Copy the Lancache file from ~/lancache/init.d/ to /etc/init.d/ by:
		sudo cp -R ~/lancache/conf/lancache /etc/init.d/

	9) Make it an executable:
		sudo chmod +x /etc/init.d/lancache

	10) Put it in the standard Boot:
		sudo update-rc.d lancache defaults

	11) Copy ~/lancache/limits.conf to /etc/security/

   	12) Edit ~/lancache/hosts to your needs, placing all your virtual IP's next to the appropriate caching service
		12.1) copy ~/lancache/hosts into your /etc dir
			cp ~/lancache/hosts /etc/

	13) Disable IPv6
	    sudo echo "net.ipv6.conf.all.disable_ipv6=1" >/etc/sysctl.d/disable-ipv6.conf
        sudo sysctl -p /etc/sysctl.d/disable-ipv6.conf

	14) Install sniproxy for passing through HTTPS traffic (cannot be cached)
		14.1) git clone https://github.com/dlundquist/sniproxy
		14.2) sudo curl https://raw.githubusercontent.com/OpenSourceLAN/origin-docker/master/sniproxy/sniproxy.conf -o /etc/sniproxy.conf
		19.3) cd sniproxy
		19.4) ./autogen.sh && ./configure && make check && make install
			# If there are problems during test procedures, you can try to skip the checks by leaving out "&&make check" 
		19.5) Start sniproxy with /usr/local/sbin/sniproxy -c /etc/sniproxy.conf

	15) Copy the unbound configuration from ~/lancache/unbound/unbound.conf to /etc/unbound/unbound.conf
	15.1) Replace the interfaces: section with the normal ip (not the virtual ones)
    	15.2) Replace all "A records" with the appropriate IPs (the virtual IPs for the appropriate caching service)

## Traffic Monitoring on CLI

	A) Monitor through nload
	   sudo apt-get install nload -y
	   sudo nload -U G -u M -i 102400 -o 102400 (shows bandwith in MByte/s)
	   or
	   sudo nload -U G -u m -i 1024000 -o 1024000 (shows bandwith in Mbit/s, scales graph to 1 Gbit/s)
	   
	B) Monitor network usage through iftop
	   sudo apt-get install iftop -y
	   sudo iftop -i eth0
	   (Instead of eth0 just use your physical interface)

## Optionals
### DHCPd

If your lancache server is a DHCP client, please add public resolvers (not the ones who you use for your LAN party guests) to dhclient.conf
to avoid resolving issues for the cache server. This can be achieved like this:
	- Add to /etc/dhclient.conf: prepend domain-name-servers 8.8.8.8, 8.8.4.4;

Alternatively you can use one of those mechanism to avoid resolv.conf being overwritten by DHCP or other services:
https://www.cyberciti.biz/faq/dhclient-etcresolvconf-hooks/

