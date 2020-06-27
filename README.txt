####******************************************* WELCOME TO IBRACORP **********************************************####
##																													##
## We run a homelab server which offers various services such as media streaming, cloud storage, chat,  and more.	##
## We are most famous for our media streaming product IBRAFLIX (https://ibraflix.com)								##
##																													##
## Our goal is to help bring open source products in the community to more users by making them friendly			##
## to use and helping support an open source future which protects privacy and is collobrative.						##
##																													##
## You can found our repos here: https://github.com/ibracorp?tab=repositories										##
## For help you can join our discord: https://discord.gg/VWAG7rZ													##
##																													##
####**************************************************************************************************************####

## The instructions below are for installing the following on unRAID using Docker:
##	Authelia
##	Website: https://www.authelia.com/
##	Docs: https://www.authelia.com/docs/
##	Git: https://github.com/authelia/authelia
##	DockerHub: https://hub.docker.com/r/authelia/authelia

## It assumes your environment has the following already setup and working:
##	- NGINX Proxy Manager
##	- Domain with the following subdomains (where 'example' is your domain and 'service' is the endpoint you want protected (i.e. monitorr.example.com)
##	Adjust/Create your own CNAMES where required.
##		- example.com
##		- auth.example.com
##		- service.example.com
##	This will not cover how to configure LDAP, Traefik or Let'sEncrypt, however there are plenty of resources on how to do this, including the offical docs of 
##	Authelia

##	REFERENCES ##
##	To make modifying easier we have tried to replace commonly required changes with a placeholder. This allows a quick Find/Replace in something like 
##	Notepad++ (which is highly recommended). All are explain in their respective steps later in this guide:
##	- YOURPASSWORD - Password which you have set, with respect the section your are reading. i.e. MySQL password could be different to your redis password.
##	- YOURSECRET - A secret generated in 128-bit. You can use this site to generate them:
		https://www.allkeysgenerator.com/Random/Security-Encryption-Key-Generator.aspx
##	- YOURDOMAIN - Your own domain name
##	- SERVERIP - Local IP address of your unRAID server the containers run on. i.e 192.168.1.50
##	- CONTAINERPORT - Port the container being proxied is running on in unRAID. i.e. Monitorr could be using 480
## 	- CONTAINERNAME - Name of the container to be proxied. i.e 'monitorr'

************INSTRUCTIONS****************
		
*********** Redis ***********

	## Authelia requires the redis container to work (as referenced in the configuration.yml)
	1. In unRAID, visit the apps tab
	2. Search for and install 'redis'. We are using the bitnami/redis container as it has parameters mapped for a password, which we will need to add into configuration.yml later.
	3. In the template installation screen:
		Network Type: The network you host your containers on so that they can communicate.
		PORT: 6379
		ALLOW_EMPTY_PASSWORD: no
		PASSWORD: YOURPASSWORD
		
*********** MYSQL/MariaDB ***********

	## Authelia requires a MYSQL/MariaDB database container to work (as referenced in the configuration.yml)
	IF YOU DO NOT ALREADY HAVE SQL INSTALLED:
		1. In unRAID, visit the apps tab
		2. Search for and install 'mariadb'. We are using the linuxserver/mariadb container.
		3. In the template installation screen:
			Network Type: The network you host your containers on so that they can communicate.
			PORT: 3306
			MYSQLROOTPASSWORD: YOURPASSWORD
	1. Under Docker tab in unRAID, left-click the mariadb container, select Console
		2. Create our user:
			Enter the following then hit enter:
				mysql -uroot -p
			Enter the password you set in the container settings then type:
				CREATE USER 'authelia' IDENTIFIED by 'YOURPASSWORD'
				## This password will be referenced in configuration.yml
		3. Create our database:
			Enter the following then hit enter:
				CREATE DATABASE IF NOT EXISTS authelia;
		4. Allow privileges to the database:
			Enter the following then hit enter:
				GRANT ALL PRIVILEGES ON authelia.* TO 'authelia' IDENTIFIED BY 'YOURPASSWORD';
				##This is the password you created for the user above.
			Enter the following then hit enter:
				quit
		5. You can now close the terminal window
		
*********** unRAID XML Template ***********

	##	Currently theres is no existing template on the Community Apps store for Authelia. Instead,
	##	we will pull a template from Git, orginally created by (big thanks) lilfade (https://github.com/lilfade).
	##	We we will pull directly from his Git to provide the credit to his work. However, you can also directly access the .xml here if you prefer:
	##	https://github.com/ibracorp/authelia/blob/master/authelia.xml

	1. Open a terminal and run:
			mkdir /boot/config/plugins/community.applications/private/myrepo/ -p 
		this will create a new folder. 
	2. (Optional) To download into the newly created folder:
			cd /boot/config/plugins/community.applications/private/myrepo/
	3. Execute this command to grab the xml file: 
			wget -O - https://gist.githubusercontent.com/lilfade/da12c44580a09c4e85f75489d30bc46b/raw/cf884c68c191e51a99fc49bbe92c7833dd1554cf/authelia.xml

	4. You can now close the terminal window
	5. In unRAID, visit the apps tab 
		Under Categories, select Private Apps. 
		You will see the newly created template for Authelia.

*********** Authelia ***********

	1. Install Authelia using the new template.
	##	The container will stop after first run as the config file is missing and will be created automatically.
	##	You should not need to change any settings unless the host port (default: 9091) will clash with any other containers.

	2. In your appdata/authelia folder you will find: 
		configuration.yml
	## You MUST edit this file to suit your domain, gmail (or other smtp) and environment. The sample provided in this repo has been tested and works, however, it is strongly advised ## the read the official docs on the configuration to ensure it meets your requirements (https://www.authelia.com/docs/configuration/)
	3. Configure the file as required. We have placed our confirmed working config in this repo. Remember the placeholders which will need to be changed 
		(listed at the top of this document)
		For secret keys, you can create a 128-bit encryption to put in from here: https://www.allkeysgenerator.com/Random/Security-Encryption-Key-Generator.aspx
		Remeber to keep them different for the different areas which use them.
	4. You will notice that LDAP has been commented out for this setup to use file backend instead. LDAP is beyond the scope of this document.
		- In our repo you will find the file named 'users_database.yml'. 
	5. Copy this file into your appdata/authelia folder.
		## You MUST edit this file
		Adjust the file to the user you would like to sign in as. For help see here: https://www.authelia.com/docs/configuration/authentication/file.html
		For password, create one here and then replace the encrypted line with your encrypted line: https://argon2.online/
			Settings for creating the password on https://argon2.online/ as referenced in the configuration.yml:
				Plain input text: your desired password
				Salt: 16
				Parallelism: 8 (or twice your CPU cores)
				Memory Cost: 1024
				Iterations: 1
				Hash length: 32
				Algorithm: Argon2id
			Select Generate Hash
			
	At this point you should start the container and read the logs. Test that you can reach the webui of Authelia (http://SERVERIP:9091) and can log in or setup 2FA.
	
*********** NGINX Proxy Manager (NPM) ***********

	##	The templates provided in this repo assume you have created a CNAME subdomain in your DNS for 'auth.example.com' and have a subdomain already working 
		for your endpoint such as 'radarr.example.com'. 
	1. Modify the data inside 'Authelia Portal.conf' and 'Protected Endpoint.conf'
		If no ports were changed in any of the above config, you should only need to change 
			'Authelia Portal.conf':
				- 'SERVERIP' = Local IP address of your unRAID server the containers run on. i.e 192.168.1.50
			'Protected Endpoint.conf':
				- 'SERVERIP' = Local IP address of your unRAID server the containers run on. i.e 192.168.1.50
				- 'CONTAINERPORT' = Port the container being proxied is running on in unRAID. i.e. Monitorr could be using 480
				- 'CONTAINERNAME' = Name of the container to be proxied. i.e 'monitorr'
	2. Copy the data and head to your NPM dashboard > Hosts > Proxy Hosts
		## WARNING - if you use Cloudflare as the DNS for your domain, you must change the setting of the subdomain in cloudflare to bypass proxy ONLY for this step.
	3. Select Add Proxy Host
		Details:
			Domain name: auth.example.com (or whatever CNAME you set in your DNS)
			Scheme: http
			Forward Hostname / IP: Local IP address of your unRAID server
			Port: 9091
			Turn ON: Cache Assets, Block Common Exploits
		SSL:
			Request new SSL certificate
			Turn ON: Force SSL, HTTP/2 Support, HSTS Enabled (if using, i.e. on in Cloudflare)
			Email address: used to create Let'sEncrypt cert.
			Select I Agree and Save.
		## REMEMBER, after this is successful, to return to Cloudflare and turn the proxy against auth.example.com back ON, or your serverip will be public.
	4. Test that you can reach the webui of Authelia selecting the new proxy or typing in it's address. i.e. 'auth.example.com'
		## NB: For some reason in the current version of NPM as of writing this (v2.2.4) the SSL settings turn off after initial creation. Go back into the SSL 
		settings of 'auth.example.com' and turn them back on then save again. 
	5. If all the above is working as intended;
		Edit proxy host 'auth.example.com'
			Advanced
				Under Custom Nginx Configuration, paste the config you customised from 'Authelia Portal.conf'
			Save
			Confirm you can still access the webui via the URL.
	6. To protect an endpoint (i.e. monitorr)
		Edit proxy host 'monitorr.example.com'
			Advanced
				Under Custom Nginx Configuration, paste the config you customised from 'Protected Endpoint.conf'
			
			
*********** TESTING ***********
In theory the workflow is:

	1. User (listed in the users file, but is not signed in) tries to connect to https://service.domain.com

	2. User is redirected to https://auth.domain.com to sign in

	3. User is given either single factor or second factor options, depending what is set on the subdomain in the configuration.yml

	4. User signs in successfully and is redirected back to origin URL https://private.domain.com


Hope this is of assistance to you. Please provide feedback where required.



















