This guide is to set up Portainer and Traefik in Docker

Requirements
-Docker CE
-Docker Compose
-Registered TLD

Options:
The files contained are set up to use a wildcard ssl cert and origin cert/key pair with authenticated TLS pulls.  I do this through Cloudflare for free.  If you use this method, you will have SSL Strict(full) and an A+ SSL rating.
- It is possible to use Lets Encrypt (ACME) to do dynamic SSL in several ways, but this is currently outside the scope of this document (doesnt take much to enable this and the majority of this still applies)
- you would just edit the traefik.toml and the stack config in portainer

#You should already have Docker and Docker-Compose installed
#here's the quick and dirty to install composer (requires curl)
curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
systemctl start docker
systemctl enable docker
systemctl restart docker

############### Install Instructions ##############
Variables Used
$DOCKERDIR
$DOMAINNAME


Step 1.  Add environment variables
    -There are various ways to do this.  I typically just edit the /etc/environment on ubuntu.  You must do this as root.  After you edit this and save, log out of your session and reconnect.  Then check to make sure they stick.
    -Add these variables
        DOMAINNAME=yourdomain.com    #your domain name
        DOCKERDIR=/home/user/docker  #where your container apps config directories live
echo $DOMAINNAME

Step 2.  Create the .htpasswd file, use a generator to generate the encrypted password
 - http://htaccesstools.com/htpasswd-generator/
 - store this file in $DOCKERDIR/shared

step 3.  Create the needed directories
 - $DOCKERDIR/traefik
 - $DOCKERDIR/portainer

step 4. grab the startup script for portainer
 - change to $DOCKERDIR/portainer and run 
wget https://github.com/benderstwin/Portainer-Templates/blob/master/start.sh
#if you want to change the port from the default of 9000, you can edit the file or manually run the docker command 
#docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v ${DOCKERDIR}/portainer/templates.json:/templates.json -v ${DOCKERDIR}/portainer/data:/data portainer/portainer

step 5.  portainer should now be running on port 9000
    - browse to http://hostip:9000
    - set up your admin username and password

###################################  BEGIN TRAEFIK CONFIG ################################

step 6.  create network for traefik
    -In portainer, go to networks and "add network"
    -settings are listed below

Name: traefik_proxy
Driver: bridge
Subnet: 172.100.10.0/24             Gateway: 172.100.10.1
IP range: 172.100.10.0/24           Exclude IPs:

Restrict external: off
Enable manual container attachment: on
Enable access control: on

    - Click "Create the network"

step 5. Create directory for traefik at $DOCKERDIR/traefik

step 6. CRT/KEY
    -  Name your .crt file domain.com.crt (replace with your domain name)
    -  Name your key file domain.com.key (replace with your domain name)
    -  Place both files in $DOCKERDIR/shared

step 7.  Add template to Portainer
    - in portainer, navigate to "stacks" (:9000/#/stacks)
    - click on "add stack"
    - click the option "git repository" and use the settings below

Repository URL: https://github.com/benderstwin/Portainer-Templates
Repository reference: leave blank
Compose path: traefik-compose.yml
Authentication: off
    - add environment variables
DOCKERDIR       /home/youruser/docker
DOMAINNAME      yourdomainname.com

    -Click on "deploy the stack"

step 8.  join traefik to the proxy network
    - in portainer, click on "Containers"
    - click on "traefik"
    - scroll to the bottom, and under "connected networks" select traefik_proxy and click "join network"
    - next to the bridge network, click "leave network"
    - you should now see traefik at http://hostip:8080 inside your network

step 9: external access
    - just forward 443/8080 to your host in your router and set up a DNS record
    - in cloudflare, set up a CNAME record for traefik.yourdomain.com and make sure the little cloud is orange
    