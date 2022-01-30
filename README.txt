            _ _   _    _ _   
   _____  _(_) |_| | _(_) |_ 
  / _ \ \/ / | __| |/ / | __|
 |  __/>  <| | |_|   <| | |_ 
  \___/_/\_\_|\__|_|\_\_|\__|
                             
      By Kaizu Shibata...
      kaizushi@infantile.us

Introduction:

	When one has systems physically seperated from Tor by the use of a
	middlebox on a private network, such a system inherits the issue of
	bad exits on the Tor network. Bad exits can manipulate traffic to
	perform 'man-in-the-middle' attacks on software update for the
	isolated systemd involved.

	This guide is about having a transparent proxy running locally on the
	isolated system, which connects it to a SOCKS server running on a
	hidden service. It is trivial these days to obtain a server with no
	questions asked or personal identification required, for a small
	amount of cryptocurrency.

	As such one can setup such a server as a private exit which is far
	more trustworhty and in your control than official Tor exits. This
	package you have downloaded contains configuration files and this
	document which should help you do this yourself. I made this because
	such a task involves several pieces of open source software which need
	to be configured for the job.

	Further, this guide and document ends on providing DNS service over a
	TCP VPN and using that in tandem with Tor's DNS server. This involves
	the software unbound so that Tor still handles requests for onion
	services but the others go to a different DNS server. This becomes
	important when you want to make a mail service that sees MX records as
	Tor's DNS support is limited and only performs A records via UDP. This
	has not been done yet and this guide will only use Tor DNS until I get
	to that.



Issues:
	This guide is published on my Github repo and you should have obtained
	it there. If you have found this text file on its own without those
	files you can get them here...

	https://github.com/kaizushi/exitkit

	If you are having problems with the steps in this guide I recommend
	that you open issues on that Github....

	https://github.com/kaizushi/exitkit/issues

	I recommend that novices spare the developers of software used within
	their problems and bring them to me. This also will help me develop
	this guide. In some cases uses for this guide one will want not want
	to post about these uses in public. Feel free to contact me personally
	with the details in this file and the README.md - one can find my GPG
	public key on my website.

	https://kloshost.online/

Software involved:

	You will be using the following software some of which is not packaged
	on many Linux distributions.

	- Debian Linux is what this guide is written for and while other
	  distros might be similar my advice may not work with them.
	- Tor is obviously required for this guide and your probably already
	  know all about it.
	- Git is used to fetch software source code so that you can build it
	  on your systems. 
	- libevent's development package is required to build nylon and
	  redsocks2 on your system.
	- nylon (https://github.com/ngocbd/nylon) is a SOCKS proxy server
	  required on the system used as the private exit on the clearnet.
	- redsocks2 (https://github.com/zcotape/redsocks2) is the transparent
	  proxy software which gets traffic from a firewall redirect and
	  converts it into SOCKS requests that are sent to the hidden service
	  of your nylon proxy.
	- iptables is used to configure the firewall. While this has become
	  obsolete modern kernels usually have an iptables wrapper which
	  converts the rules into netfilter tables for you.
	- unbound is used when you want to enhance DNS support on your system
	  beyond what is supported within Tor itself.

Know Your Customer:

	In discourse among us Dark Web hackers we have the term KYC which
	stands for 'Know Your Customer' and discuss services which 'don't KYC'
	which means they do not request identification and other details.

	Obtaining the clearnet server to fulfill this guide in a way that it
	can not be traced back can be tricky. A lot of providers which don't
	ask questions and such still block Tor Exits and some VPN providers.
	While I provide this guide for free you could contact me to purchase
	access to such a system that you would be able to use to set up your
	own similar system using the next steps in this guide.

	You should also ensure your cryptocurrency itself does not trace back
	to you. There are a variety of Bitcoin mixers and exchanges on the
	Dark web. I recommend obtaining XMR somehow and using that to pay or
	converting it to BTC. I find the Electrum wallet software usually
	works nicely behind Tor.

	CAUTION: If you fuck up this part you could make things incredibly bad
	for yourself. Only do anything in this guide of you have a habit of
	personal discipline.

Securing access clearnet side:

	Now that you somehow have obtained a server, which does not trace back
	to you, and is yours to control you should set it up for its basic
	job. In this step we install Tor and configure it so that SSH is
	accessible on it via a hidden service. You are probably accessing the
	server via its IP address over Tor exits. While SSH is encrypted you
	have to verify you are really talking to your server's SSH process by
	confirming the SSH fingerprint.

	When you first connect to a system over SSH it shows its fingerprint
	and asks if you which to save it to your system. It is a very
	important step. If you used a reliable VPN to obtain the server in the
	first place you might be able to rely on the fingerprint you get that
	that way. Good providers will show this fingerprint on the control
	panel for your clearnet server.

	A good way is to confirm it from the actual system using its console
	(screen and mouse) which most providers have. Log in to the system and
	just run this simple command...

	$ ssh localhost

	This will show the fingerprint for your server which you can confirm
	with the one you are getting behind Tor. You don't even need to log
	in, once you see the fingerprint you can just hit CTRL-C and be done
	with it.

	Now that you see that identifying your server is pretty important and
	that a lot can go wrong here, you can make an assurance against such
	problems by using the power of hidden services. If you put SSH access
	on a hidden service the SSH fingerprint will become redundant and you
	will be able to rely on the big random key that a hidden service
	address is to know it is still your server. Also, the hidden service
	will be a bit faster than using the clearnet IP over Tor exits.

	As root on your new server you will want to install Tor...

	# apt install tor

	This will install Tor and start it on your server which gets things
	ready. You then just need to edit the configuration file to have a
	hidden service for SSH.

	Open this file in a text editor: /etc/tor/torrc

	Add these lines within the file, ignore the identation in this guide
	you are reading.

	HiddenServiceDir /var/lib/tor/ssh-service
	HiddenServicePort 22 localhost:22

	This forwards the port 22 (which is assigned to SSH) to your server
	from a new hidden service. You then just need to reload Tor so it
	works...

	# systemctl reload tor

	The command below will show you the address of the hidden service...

	# cat /var/lib/tor/ssh-service/hostname

	You should see a single line ending in .onion which is the hidden
	service for SSH. Keep this address to yourself as nobody should need
	it. The hidden service for your nylon proxy will be something else. If
	it said the file or directory didn't exist instead of giving you your
	onion you might have made a mistake in your Tor configuration.

	You should setup some credentials on your system of your own and
	remove those provided by your host. You should surely change the root
	password...

	# passwd

	You should also setup a user on the system and log in using this and
	switch to root. Replace 'username' here with your desired username...

	# useradd -m -G users username

	Note: useradd is a completely different command from adduser!

	Set a password for this user...

	# passwd username

Logging in securely the first time...

	Now that you have the hidden service setup for SSH you can log into it
	to peform the rest of the steps. To connect to a hidden service using
	SSH you must have Tor and the 'torsocks' utility running on your
	system. It is as simple as follows...

	$ torsocks ssh username@xxxxxxxxxxxxxxxxxxxxxxxxxxxx.onion

	This will log you into your server using the credentials you setup
	before using the 'useradd' and 'passwd' commands. You can make this
	better still by using private-public keys with SSH. This involves
	locally generating SSH keys and uploading them to your clearnet
	server. First do this on your client system (not the clearnet VPS)...

	$ ssh-keygen

	That will ask some questions and write a SSH keypair to the '.ssh'
	directory in your home directory. You can then upload it to your new
	clearnet server. This will ask for the password and then put the key
	on the server. This is the command...

	$ ssh-copy-id username@xxxxxxxxxxxxxxxxxxxxxxxxxxxx.onion

	Once that is done you can then connect to SSH more securely than
	before. You can also copy these keys so that you can use these keys to
	log in as root. On your clearnet server do the following...

	# mkdir .ssh
	# cp /home/username/.ssh/authorized_keys .ssh/

	Now you should be able to log in as root immediately like so...

	$ torsocks ssh root@xxxxxxxxxxxxxxxxxxxxxxxxxxxx.onion

	You can of course become root with 'su' and you should actually type a
	little more than that. By putting a dash on the end of the command
	'su' will setup the environment for root...

	# su -

	You might notice that there is some pretty heavy lag while working on
	your server over its hidden service. I recommend that you run a text
	editor on your desktop system where you type things and then paste
	them into the terminal for your hidden server. It makes things so
	much easier!

	Anyway, if you followed this advice you can get started installing
	software.

Installing the dependencies...

	Your server will need some software so that you can then build and
	install some more software. You will need to install Git which will be
	used to grab source code for nylon and redsocks2. You will need some
	tools to build and compile the software obtained via Tor. You will
	need the libevent-dev libraries required by nylon and redsocks2.

	These dependencies are the same of both your isolated system which
	will be running redsocks2, as well as the clearnet system with the
	nylon SOCKS proxy server. So you need to run the following command on
	both of these systems...

	# apt install git build-essential libevent-dev

	Once this has completed on both systems you will have everything you
	need to install nylon and redsocks on these systems.

Copying configuration files in base64...

	This guide comes with a package with other files, configuration files
	used to configure the software mentioned within. Copying these files
	using tools like scp and such is possible. However I like to use the
	tool 'base64' to convert files into something I can copy paste between
	systems in my terminal as I do the other steps involved below.

	You can convert a file into base64 as follows, in this instance we are
	telling the shell to get 'redsocks.conf' and turn it into base64
	output...

	base64 < redsocks.conf

	This will display a big block of base64 output which you can then
	select and copy on your system. You can then run the command below on
	the remote system to decode this block of output back into the
	configuration file you wish to upload, ending by pressing CTRL-D at
	the bottom.

	base64 -d > /etc/redsocks/redsocks.conf

	The CTRL-D has to be at the start of a line, if your paste does not
	end with a newline CTRL-D will appear to do nothing. In that case just
	press ENTER and then CTRL-D. CTRL-D sends the ascii code for 'EOF'
	which means 'end of file.' This tells base64 to exit and you are
	finished. If you notice a mistake you can send CTRL-C which will stop
	base64 without doing the conversion.

	This can of course be adapted by changing what file is written in and
	out of base64, and the method above can be used for pretty much any
	file.

Getting the nylon proxy server running...

	This software goes on the clearnet VPS that is going to become your
	private exit from the Tor network. First you obtain the source code
	using Git...

	# git clone https://github.com/ngocbd/nylon

	You then need to change in its directory...

	# cd nylon

	Once that is done you should just need to type the command 'make' to
	build the nylon proxy software. This command reads the Makefile and
	follows its instructions to build this program. Once that is finished
	you install it as follows...

	# make install

	This will install nylon on your system which should be ready for you
	to use. Nylon does not have a configuration file and this does not
	install a systemd service. In the package where you found this guide
	you will want to upload the nylon.service file to systemd on your
	clearnet server. Using the base64 method mentioned above...

	# base64 -d > /lib/systemd/system/nylon.service

	Once you have done this you should be able to start nylon and get
	things running...

	# systemctl start nylon

	If there is no output from the above command then nylon has started
	but it might output some kind of error. If that happens you will just
	have to find some help. I recommend contacting me directly about this
	so I can improve this guide.

	Now that you nylon can run on your system you will want to make it so
	that it starts when your server boots...

	# systemctl enable nylon

	Now we configure the hidden service for the nylon socks proxy you are
	using within Tor. This is similar to the advice for SSH above. You
	once again add a couple of lines to your Tor configuration.

	HiddenServiceDir /var/lib/tor/nylon
	HiddenServicePort 1080 localhost:1080

	This creates a hidden service which sends traffic to the nylon service
	you have running on your system. You will want to reload Tor with
	systemd so that it loads the new configuration and creates the hidden
	service and gets it running...

	# systemctl reload tor

	Once that is done you will want to get the address for this new hidden
	service, and make a note of it as later it will be used in the
	redsocks configuration on your isolated system.

	# cat /var/lib/tor/nylon/hostname

	You should test the new proxy using the curl command on the system you
	are working from. This involves torsocks much like when you connected
	to your clearnet server over SSH. You do this...

	$ torsocks curl	--socks5 xxxxxxxxxxxxxxxxxxxxxx.onion:1080 google.com

	(The above command was adapted to be on two lines because of the
	formatting of this text file. You can remove the '\' character and put
	it on one line if you wish.)

	That test should take some time and then output six lines of HTML from
	Google, which redirect people to the HTTPS site for Google using a
	HTML redirect. 

	If this all worked you are ready to configure the isolated system
	which will be using the Tor exit.

Getting redsocks2 running on the isolated system...

	Much like before this starts with you getting redsocks2 from Git but
	in this case we will also disable a feature we do not use. Much like
	before you run this to grab its source code...

	# git clone https://github.com/zcotape/redsocks2

	You then change to the directory that created...

	# cd redsocks2

	Yet this time we apply a patch which disables support for SSL
	inspection, which will not be needed...

	# git apply redsocks2/disable-ss.patch

	You then build it in the typical way...

	# make

	Once this has completed we have to install redsocks2 but its Makefile
	does not have a target called 'install' and so we cannot do the usual
	thing. However the step just completed as created an executable
	binary file called 'redsocks2' and we just have to copy it into your
	system...

	# cp -p redsocks2 /usr/local/sbin/redsocks2

	This has copied the file while preserving its permissions and put it
	where it needs to be for the systemd service I have supplied in the
	file redsocks2.service, which as before can be supplied to the system
	using base64 as mentioned above...

	# base64 -d > /lib/systemd/system/redsocks.service

	Unlike nylon redsocks2 also has a configuration file and a directory
	where it belongs. First you will want to create the directory...

	# mkdir /etc/redsocks

	The configuration file which goes in this directory will have
	sensitive information. The onion address for the hidden service of the
	nylon proxy must be kept private as it its only authentication is that
	big random onion address. Change the permissions...

	# chmod a-rwx,u+rwx,g+rx /etc/redsocks

	Now you want to upload the configuration file I have supplied in
	redsocks.conf, using the base64 method...

	# base64 -d > /etc/redsocks/redsocks.conf

	This configuration requires some modification and it is here you will
	use the onion address of the nylon proxy. You want to replace the
	letters HIDDEN_SERVICE with this address. This can be done from the
	command line using a tool called sed...

	# sed -i 's/HIDDEN_SERVICE/xxxxxxxxxxxxxxxxxxx.onion/g' redsocks.conf

	Now you can start redsocks2 on your system...

	# systemctl start redsocks2

	If things have been done correctly that should have no output but if
	something is wrong it will show some kind of error. In that case you
	should seek support by contacting me. If it worked you will want to
	enable redsocks2 on boot...

	# systemctl enable redsocks2

	There is some firewall configuration requried which I have provided a
	script for, which should only be ran to set things up. Once you have
	ran it you will have to save your firewall rules and have them loaded
	on boot. You should make it executable run the script...

	# chmod +x iptables-redsocks.sh
	# ./iptables-redsocks.sh

	You might get an error about a chain already existing. This can be
	safely ignored.

	You then want to save your firewall rules...

	# iptables-save > /etc/iptables.up.rules

	Getting the rules to load on boot involves modifying your network
	interface configuration in /etc/network/interfaces to load those
	iptables rules. This page of the Debian Wiki is relevant...
	
	https://wiki.debian.org/iptables

	An interface should have a configuration which could look like this...

	iface eth0 inet static
		address 192.168.0.2/24
		gateway 192.168.0.1
		network 192.168.0.0

	You will want to change it so it looks like this...

	iface enp1s0 inet static
		address 192.168.0.2/24
		gateway 192.168.0.1
		network 192.168.0.0
		post-up '/usr/sbin/iptables-restore < /etc/iptables.up.rules'

	This part with the firewall can get a little complicated and different
	on different systems. Yet the basic essentials of doing it are to save
	the iptables rules my script creates and have them loaded on boot.

	Once this is done you should be able to test it all as working with
	the following command...

	# curl api.ipify.org; echo

	That command will take some time but should output the IP address for
	your clearnet VPS that is private exit. If it outputs some other IP
	address it that of a Tor exit and something has gone wrong.

Congratulations...

	You now have your system using a private Tor exit to do things. If
	this guide has been helpful please tell others about it. You can send
	me donations of cryptocurency!

	Bitcoin donations...
	1FN1Tb6f8WE8uk6Wj5kBy6ryzohhrqfvui

	Monero donations...
	83eVdewjYYzQMqcwFVtbKKNjJnowHVXRJA7aLXB6Tov3iAuRATHRWSNaxUWs9Ez2uN6at3vvdJ6JmF2kDNrCTFBdEGRG1Z9

	Thanks!
