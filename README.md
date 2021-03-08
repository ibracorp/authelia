<img src="https://i.imgur.com/hHH2J7w.png" width="800" height="200">

We run a home lab server which offers various services such as media streaming, cloud storage, chat, and more.
We are most famous for our media streaming product IBRAFLIX (https://ibraflix.com)
Our goal is to help bring open source products in the community to more users by making them friendly to use and helping support an open source future which protects privacy and is collaborative.

## == Support Us ==
Membership: http://bit.ly/3bRgdlU <br>

Help support my work by subscribing to our site. It's free with paid options. There are no fees involved and it really helps me give back to you.<br>

Become a free subscriber of our site to:<br>
- Receive the latest YouTube videos first, before going public on YouTube.<br>
- Read our articles which go with our videos and other work we do.<br>
- Emails directly to your inbox with the latest content. No spam, no bs.<br>
- More<br>

Become a paid subscriber of our site to:<br>
- Get exclusive videos only for supporters.<br>
- Ask for direct support with helping install or provide consultancy to you.<br>
- Receive advanced tutorials and articles for your IT needs.<br>
- Help support indie creators (and a father of two) to bring you the best content possible!<br>

We really appreciate your support. <br>

## == IBRACORP ==
IBRACORP - http://bit.ly/IBRACORP
YouTube: https://bit.ly/3q39SJO
GitHub - https://bit.ly/3onzEY3
unRAID Forum - http://bit.ly/2MwDPTV
Discord - http://bit.ly/2YgjcOq

## == Support ==
If you require support or have any questions you can contact us at support@ibracorp.io. <br>

===============================

*Disclaimer: we are in no way offically affiliated with the product below. We simply try to connect their work to a wider audience. We are not responsible or liable for any damages as result of following these instructions and are provided only for reference and assistance.*

----
<img src="https://i.imgur.com/h0frxFb.png" width="500" height="250">

# Table of Contents
1. [Authelia on unRAID](#authelia-on-unraid)
2. [References](#references)
3. [Redis](#redis)
4. [MySQL & MariaDB](#mysqlmariadb)
5. [Authelia](#authelia)
6. [NGINX Proxy Manager](#nginx-proxy-manager-npm)
7. [Protecting Endpoints](#to-protect-an-endpoint-ie-sonarr)
8. [FAQ](#faq)
	1. [Worflow](#workflow)
	2. [Infinite Login Screen](#noinfinite-native-login-screen-on-endpoint)
	3. [Let's Encrypt](#letsencrypt)
	4. [LDAP](#ldap)
	5. [DUO 2FA](#duo-2fa)
	6. [Authelia Interface](#access-the-authelia-interface)
	7. [Startup Order](#startup-order)
	

## Authelia on unRAID
The instructions below are for installing the following on unRAID using Docker:
- Authelia
    -  Website: https://www.authelia.com/
    - Docs: https://www.authelia.com/docs/
    - Git: https://github.com/authelia/authelia
    - Docker Hub: https://hub.docker.com/r/authelia/authelia

We assume your environment has the following already setup and working:
- NGINX Proxy Manager
- Domain with the following subdomains (where 'example' is your domain and 'service' is the endpoint you want protected (i.e. monitorr.example.com)
    - Adjust/Create your own CNAMES where required.
		- example.com
		- auth.example.com
		- service.example.com

This will not cover how to configure ~~LDAP~~ (see bottom), Traefik or Let’s Encrypt, however there are plenty of resources on how to do this, including the official docs of Authelia.

----
##	REFERENCES
To make modifying easier we have tried to replace commonly required changes with a placeholder. This allows a quick Find/Replace in something like Notepad++ (which is highly recommended). 
All are explained in their respective steps later in this guide:
- YOURPASSWORD - Password which you have set, with respect the section you are reading. i.e. MySQL password could be different to your Redis password.
- YOURSECRET - A secret generated in 128-bit. You can use this site to generate them: 
   - https://www.allkeysgenerator.com/Random/Security-Encryption-Key-Generator.aspx
- YOURDOMAIN - Your own domain name
- SERVERIP - Local IP address of your unRAID server the containers run on. i.e. 192.168.1.50
- CONTAINERPORT - Port the container being proxied is running on in unRAID. i.e. Monitorr could be using 480
- CONTAINERNAME - Name of the container to be proxied. i.e. 'monitorr'
- CONTAINERIP - IP address of the container.

---
		
## Redis

 Authelia requires the Redis container to work (as referenced in the configuration.yml)
1. In unRAID, visit the apps tab
2. Search for and install 'redis'. We are using the bitnami/redis container as it has parameters mapped for a password, which we will need to add into configuration.yml later.
3. In the template installation screen:

		Network Type: The network you host your containers on so that they can communicate.
		PORT: 6379
		ALLOW_EMPTY_PASSWORD: no
		PASSWORD: YOURPASSWORD
		
## MYSQL/MariaDB

 Authelia requires a MYSQL/MariaDB database container to work (as referenced in the configuration.yml)
IF YOU DO NOT ALREADY HAVE SQL INSTALLED:
1. In unRAID, visit the apps tab
2. Search for and install 'mariadb'. We are using the linuxserver/mariadb container.
3. In the template installation screen:

			Network Type: The network you host your containers on so that they can communicate.
			PORT: 3306
			MYSQLROOTPASSWORD: YOURPASSWORD
4. Under Docker tab in unRAID, left click the mariadb container, select Console
5. Create our user:
    -   Enter the following then hit enter:

			mysql -uroot -p
	-   Enter the password you set in the container settings then type:

			CREATE USER 'authelia' IDENTIFIED by 'YOURPASSWORD';
    This password will be referenced in configuration.yml
6. Create our database:
	-   Enter the following then hit enter:

			CREATE DATABASE IF NOT EXISTS authelia;
7. Allow privileges to the database:
	-   Enter the following then hit enter:

		    GRANT ALL PRIVILEGES ON authelia.* TO 'authelia' IDENTIFIED BY 'YOURPASSWORD';
		This is the password you created for the user above.
	-   Enter the following then hit enter:

			quit
8. You can now close the terminal window


## Authelia

1. Install Authelia via the Community Apps plugin in unRAID. Original template was created by (big thanks) lilfade (https://github.com/lilfade)
    - The container will stop after first run as the config file is missing and will be created automatically.
    - You should not need to change any settings unless the host port (default: 9091) will clash with any other containers.

2. In your appdata/authelia folder you will find: 

	    configuration.yml
You **MUST** edit this file to suit your domain, gmail (or other smtp) and environment. The sample provided in this repo has been tested and works, however, it is strongly advised to read the official docs on the configuration to ensure it meets your requirements (https://www.authelia.com/docs/configuration/)

3. Configure the file as required. We have placed our confirmed working config in this repo. Remember the placeholders which will need to be changed (listed at the top of this document).
	- For secret keys, you can create a 128-bit encryption to put in from here: https://www.allkeysgenerator.com/Random/Security-Encryption-Key-Generator.aspx
		Remember to keep them different for the different areas which use them.
4. You will notice that LDAP has been commented out for this setup to use file backend instead. LDAP is beyond the scope of this document.
	- In our repo you will find the file named 'users_database.yml'. 
	- Copy this file into your appdata/authelia folder.
	You **MUST** edit this file.
		- Adjust the file to the user you would like to sign in as. For help see here: https://www.authelia.com/docs/configuration/authentication/file.html
		- For password, create one here and then replace the encrypted line with your encrypted line: https://argon2.online/
		- Settings for creating the password on https://argon2.online/ as referenced in the configuration.yml:
	
				Plain input text: your desired password
				Salt: 16
				Parallelism: 8 (or twice your CPU cores)
				Memory Cost: 1024
				Iterations: 1
				Hash length: 32
				Algorithm: Argon2id
			Select Generate Hash
			
	At this point you should start the Authelia container and read the logs. Test that you can reach the webui of Authelia (http://SERVERIP:9091) and can log in or setup 2FA.
	
## NGINX Proxy Manager (NPM)
The templates provided in this repo assume you have created a CNAME subdomain in your DNS for 'auth.example.com' and have a subdomain already working for your endpoint such as 'radarr.example.com'. 
1. Modify the data inside 'Authelia Portal.conf' and 'Protected Endpoint.conf'. If no ports were changed in any of the above config, you should only need to change:
	- 'Authelia Portal.conf':
		- 'SERVERIP' = Local IP address of your unRAID server the containers run on. i.e. 192.168.1.50
	
	- 'Protected Endpoint.conf':
		- 'SERVERIP' = Local IP address of your unRAID server the containers run on. i.e. 192.168.1.50
		- 'CONTAINERNAME' = Name of the container to be proxied. i.e. 'monitorr'
		- 'CONTAINERIP' = IP address of the container.
		- 'YOURDOMAIN' = Your own domain name. 

2. Copy the data and head to your NPM dashboard > Hosts > Proxy Hosts
		 
- ~~**WARNING** - if you use Cloudflare as the DNS for your domain, you must change the setting of the subdomain in Cloudflare to bypass proxy ONLY for this step.~~ Seems to be fixed now.

3. Select Add Proxy Host
    - Details:
        - Domain name: auth.example.com (or whatever CNAME you set in your DNS)
		- Scheme: http
		- Forward Hostname / IP: Local IP address of your unRAID server
		- Port: 9091
		- Turn ON: Cache Assets, Block Common Exploits
	- SSL:
		- Request new SSL certificate
		- Turn ON: Force SSL, HTTP/2 Support, HSTS Enabled (if using, i.e. in Cloudflare)
		- Email address: used to create Let’s Encrypt cert.
		- Select I Agree and Save.
		
~~**REMINDER**: after this is successful, return to Cloudflare and turn the proxy against auth.example.com back ON, or your server IP will be public.~~

4. Test that you can reach the webui of Authelia selecting the new proxy or typing in its address. i.e. 'auth.example.com'
    - **NB:** For some reason in the current version of NPM as of writing this (v2.2.4) the SSL settings turn off after initial creation. Go back into the SSL 
		settings of 'auth.example.com' and turn them back on then save again. 
5. If all the above is working as intended; Edit proxy host 'auth.example.com'
	- Advanced
		- Under Custom Nginx Configuration, paste the config you customised from 'Authelia Portal.conf'
	
6. Save and confirm you can still access the webui via the URL.

## To protect an endpoint (i.e. sonarr)
1. Edit proxy host 'sonarr.example.com'
	- Advanced
		- Under Custom Nginx Configuration, paste the config you customised from 'Protected Endpoint.conf'
2. (Optional) If using services which use API to communicate with eachother such as Radarr, Sonarr or Lidarr, you may also need to add a location for the API in order to disable the authorization else it may fail to connect. Settings below are relevant to Sonarr and it's sister products. Be sure to check the docs of the service you are configuring. 
	- Edit proxy host 'sonarr.example.com'
    		- Custom Locations

	            Location: /api
	            Scheme: http
	            Forward Hostname/IP: SERVERIP/api
	            Forward Port: 8686
	            Select gear icon: auth_request off;
        
	- Confirm you can connect to the API by using, for example, Ombi. TV > Sonarr > Test connection. 		
			
# FAQ

## Workflow
In theory the workflow is:

1. User (listed in the users file, but is not signed in) tries to connect to https://service.domain.com

2. User is redirected to https://auth.domain.com to sign in

3. User is given either single factor or second factor options, depending what is set on the subdomain in the configuration.yml

4. User signs in successfully and is redirected back to origin URL https://service.domain.com


Hope this is of assistance to you. Please provide feedback where required.

## No/infinite native login screen on endpoint
You may find when passing through Authelia successfully that the endpoint (i.e. Sonarr) has no login screen (if you had a login screen enabled).
This is not related to Authelia, but rather NGINX. From personal experience performing the below may fix this.
1. Edit proxy host 'sonarr.example.com'
	- Advanced
		- Under Custom Nginx Configuration, paste the below in **above** any location blocks
			
			proxy_intercept_errors off;

Test again. If no change, try with it on or removed again.

## Let'sEncrypt

If you are using LinuxServer.io LE container you need to add this under the server block for its out-of-the-box Authelia support to work:

	server:
  	path: authelia
	
If you are using the LSIO LE container, there's no need to utilize Authelia as its own subdomain reverse proxy.

## LDAP

If you want to use LDAP as your backend (which is recommended), here's the config we use in the Authelia YAML. Be sure to comment out the File Backend section when using this. 

**NOTE**: This config is based on implementation with FreeIPA as our LDAP server. If using any other server such as OpenLDAP or Active Directory, you will need to adjust the user/group attributes and filters to suit. 
You must also modify the domain settings below to match your environment.

```
# LDAP backend configuration.
  #
  # This backend allows Authelia to be scaled to more
  # than one instance and therefore is recommended for
  # production.
  ldap:
    # The url to the ldap server. Scheme can be ldap:// or ldaps://
    url: ldap://192.168.1.150
    
    # Skip verifying the server certificate (to allow self-signed certificate).
    skip_verify: true
    
    # The base dn for every entries.
    base_dn: dc=contoso,dc=com
    
    # The attribute holding the username of the user. This attribute is used to populate
    # the username in the session information. It was introduced due to #561 to handle case
    # insensitive search queries.
    # For you information, Microsoft Active Directory usually uses 'sAMAccountName' and OpenLDAP
    # usually uses 'uid'
    # Beware that this attribute holds the unique identifiers for the users binding the user and the configuration
    # stored in database. Therefore only single value attributes are allowed and the value
    # must never be changed once attributed to a user otherwise it would break the configuration
    # for that user. Technically, non-unique attributes like 'mail' can also be used but we don't recommend using
    # them, we instead advise to use the attributes mentioned above (sAMAccountName and uid) to follow
    # https://www.ietf.org/rfc/rfc2307.txt.
    username_attribute: uid
    
    # An additional dn to define the scope to all users
    additional_users_dn: cn=users,cn=accounts

    # The users filter used in search queries to find the user profile based on input filled in login form.
    # Various placeholders are available to represent the user input and back reference other options of the configuration:
    # - {input} is a placeholder replaced by what the user inputs in the login form. 
    # - {username_attribute} is a placeholder replaced by what is configured in `username_attribute`.
    # - {mail_attribute} is a placeholder replaced by what is configured in `mail_attribute`.
    # - DON'T USE - {0} is an alias for {input} supported for backward compatibility but it will be deprecated in later versions, so please don't use it.
    #
    # Recommended settings are as follows:
    # - Microsoft Active Directory: (&({username_attribute}={input})(objectCategory=person)(objectClass=user))
    # - OpenLDAP: (&({username_attribute}={input})(objectClass=person))' or '(&({username_attribute}={input})(objectClass=inetOrgPerson))
    #
    # To allow sign in both with username and email, one can use a filter like
    # (&(|({username_attribute}={input})({mail_attribute}={input}))(objectClass=person))
    users_filter: (&({username_attribute}={input})(objectClass=person))

    # An additional dn to define the scope of groups
    additional_groups_dn: cn=groups,cn=accounts
    
    # The groups filter used in search queries to find the groups of the user.
    # - {input} is a placeholder replaced by what the user inputs in the login form.
    # - {username} is a placeholder replace by the username stored in LDAP (based on `username_attribute`).
    # - {dn} is a matcher replaced by the user distinguished name, aka, user DN.
    # - {username_attribute} is a placeholder replaced by what is configured in `username_attribute`.
    # - {mail_attribute} is a placeholder replaced by what is configured in `mail_attribute`.
    # - DON'T USE - {0} is an alias for {input} supported for backward compatibility but it will be deprecated in later versions, so please don't use it.
    # - DON'T USE - {1} is an alias for {username} supported for backward compatibility but it will be deprecated in later version, so please don't use it.
    groups_filter: (&(member=uid={0},cn=users,cn=accounts,dc=contoso,dc=com)(objectclass=groupofnames))

    # The attribute holding the name of the group
    group_name_attribute: cn

    # The attribute holding the mail address of the user. If multiple email addresses are defined for a user, only the first
    # one returned by the LDAP server is used.
    mail_attribute: mail

    # The attribute holding the display name of the user. This will be used to greet an authenticated user.
    display_name_attribute: givenName

    # The username and password of the admin user.
    user: uid=authelia,cn=users,cn=accounts,dc=contoso,dc=com
    # Password can also be set using a secret: https://docs.authelia.com/configuration/secrets.html
    password: YOURPASSWORD
```

## Duo 2FA

These instructions were provided by ThreeFN on our Unraid forum thread (link at top).

>On Duo, you actually need TWO logins.  The first is your admin account that sets-up your hostname/integration_key/secret_key via Partner Auth API.  Now with with you need to go into the config for the Application->PartnerAuthAPI and add a user that is THE SAME NAME as the user you have in file/ldap and then EMAIL THEM which will give you the ability to enroll the phone app to that user.  Then you can enroll that in authelia when you get to that point.


## Access the Authelia Interface

At any time, you can go directly to the Authelia page by typing in your URL set for it. i.e. auth.example.com.

## Startup Order

If using the external database and Redis options (recommended), it's important to note that the startup order of your containers must be configured correctly.
In unRAID:
1. On the Dockers page, select Advanced
2. Click and drag the rows of containers so that all database containers are higher on the list than Authelia
3. Next, beside the Autostart toggle, you can set a delay (in seconds) for the container to wait before starting the next container underneath it.
	- This is useful because it allows certain containers which take a while to start up and may have dependencies to have more time to finish.
