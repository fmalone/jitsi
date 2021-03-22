#!/bin/bash

# Register user
#prosodyctl register user your_domain password

# Function for error messages
errorecho() { cat <<< "$@" 1>&2; }

# Check for root
if [ "$(id -u)" != "0" ]
then
	errorecho "ERROR: This script has to be run as root!"
	exit 1
fi

BLUE='\033[1;34m'
NC='\033[0m'

# Get user input
printf "${BLUE}Enter your domain name(example.com): ${NC}"
read Domain

printf "${BLUE}Updating...\n${NC}"
apt update && apt upgrade -y
printf "${BLUE}Done.${NC}\n\n"

printf "${BLUE}Changing hostname...\n${NC}"
hostnamectl set-hostname $Domain
echo "127.0.0.1 $Domain" >> /etc/hosts
printf "${BLUE}Done.${NC}\n\n"

printf "${BLUE}Setting up firewall...\n${NC}"
apt install ufw -y
yes | ufw enable
ufw allow ssh
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 4443/tcp
ufw allow 10000/udp
ufw status
printf "${BLUE}Done.${NC}\n\n"

printf "${BLUE}Adding jitsi gpg key...\n${NC}"
apt install gnupg -y
wget https://download.jitsi.org/jitsi-key.gpg.key
apt-key add jitsi-key.gpg.key
rm jitsi-key.gpg.key
printf "${BLUE}Done.${NC}\n\n"

printf "${BLUE}Adding jitsi repository...\n${NC}"
echo "deb https://download.jitsi.org stable/" >> /etc/apt/sources.list.d/jitsi-stable.list
printf "${BLUE}Done.${NC}\n\n"

printf "${BLUE}Installing jitsi-meet...\n${NC}"
apt update
apt install jitsi-meet -y
printf "${BLUE}Done.${NC}\n\n"

printf "${BLUE}Generating cert...\n${NC}"
apt install software-properties-common -y
yes | add-apt-repository ppa:certbot/certbot
apt install certbot -y
/usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
ufw delete allow 80/tcp
printf "${BLUE}Done.${NC}\n\n"

printf "${BLUE}Configure Jitsi to only allow registered users to create conferences (y/n)? ${NC}"
read answer
if [ "$answer" != "${answer#[Yy]}" ] ;then
	printf "${BLUE}Configuring jitsi...\n${NC}"
	sed -i "s/authentication = \"anonymous\"/authentication = \"internal_plain\"/g" /etc/prosody/conf.avail/$Domain.cfg.lua
	echo "VirtualHost \"guest.$Domain\"" >> /etc/prosody/conf.avail/$Domain.cfg.lua
	echo "    authentication = \"anonymous\"" >> /etc/prosody/conf.avail/$Domain.cfg.lua
	echo "    c2s_require_encryption = false" >> /etc/prosody/conf.avail/$Domain.cfg.lua
	sed -i "s/\/\/ anonymousdomain: 'guest.example.com',/anonymousdomain: 'guest.$Domain',/g" /etc/jitsi/meet/$Domain-config.js
	echo "org.jitsi.jicofo.auth.URL=XMPP:$Domain" >> /etc/jitsi/jicofo/sip-communicator.properties
	printf "${BLUE}Done.${NC}\n\n"
fi

# Restart Jitsi Meet processes
printf "${BLUE}Restarting jitsi services...\n${NC}"
systemctl restart prosody.service
systemctl restart jicofo.service
systemctl restart jitsi-videobridge2.service
printf "${BLUE}Done.${NC}\n\n"

printf "${BLUE}DONE.${NC}\n\n"
