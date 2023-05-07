# Prerequisites
You will need to create two separate droplets for your VPN server. One will host the Certifcate Authority (CA) and the other will host the VPN.
## Initial Server Setup with Ubuntu 22.04
The Region chosen does not matter.
The droplet used for this project ran on Ubuntu 22.04 (LTS) x64 and the plan used was the Regular intel CPU @ $4/month
Add the password (or use SSH) and create the droplet 

After creating the Droplet, access the console to create the users.

Run the following:
- ssh root@your_server_ip (log into your server as the root account)
- adduser [name] (this will be the user for your server)
- usermod -aG sudo [name] (this will give the user administrative privileges)

After creating the users and giving them access, you will have to set up the Firewall

- ufw allow OpenSSH (this will allow OpenSSH to pass through your firewall)
- ufw enable (this will enable your firwall) press y and enter to proceed
- Check the firewall by running the command: ufw status

SSH into the new user account by running the command:
- ssh [name]@[your_server_ip]

This will log you into the user account. If you need to run a command with administrative privileges, type sudo before the command

## How To Set Up and Configure a Certificate Authority On Ubuntu 22.04
YOU WILL NEED TO HAVE MADE ANOTHER SERVER BEFORE STARTING THIS

Create another Droplet using the same configurations as mentioned previously
Login to your CA Server as the non-root sudo user that you created during the initial setup steps and run the following:
- sudo apt update
- sudo apt install easy-rsa
This will install the easy-rsa packages as well as updating any other packages needed

Run the following command to create a directory for your PKI
- mkdir ~/easy-rsa
You will then create a symbolic link pointing to the easy-rsa packages installed earlier
- ln -s /usr/share/easy-rsa/* ~/easy-rsa/

To restrict access to your new PKI directory, ensure that only the owner can access it using the chmod command:
- chmod 700 /home/[name]/easy-rsa

Go into the easy-rsa directory and run the following command to initialize the PKI
- ./easyrsa init-pki

Stay in the easy-rsa directory and run the command:
- nano vars (creates a file named vars)
Paste the following information into the file and change it to fit your information
```set_var EASYRSA_REQ_COUNTRY    "US"
set_var EASYRSA_REQ_PROVINCE   "NewYork"
set_var EASYRSA_REQ_CITY       "New York City"
set_var EASYRSA_REQ_ORG        "DigitalOcean"
set_var EASYRSA_REQ_EMAIL      "admin@example.com"
set_var EASYRSA_REQ_OU         "Community"
set_var EASYRSA_ALGO           "ec"
set_var EASYRSA_DIGEST         "sha512" 
```
Exit out of the file by pressing CTRL-X, Y, then ENTER to confirm

Run the following command to create the public and private key pair for your CA
- ./easyrsa build-ca
Fill out the information regarding the common name for your CN, for simplicity, you can just press ENTER to accept the deafult name
Running this script will also create two very important files for your server
ca.crt is the CA’s public certificate file
ca.key is the private key that the CA uses to sign certificates for servers and clients

Go into your ca.crt file by running the command:
- cat ~/easy-rsa/pki/ca.crt
There will be an output in your terminal similar to the following:
```
-----BEGIN CERTIFICATE-----
MIIDSzCCAjOgAwIBAgIUcR9Crsv3FBEujrPZnZnU4nSb5TMwDQYJKoZIhvcNAQEL
BQAwFjEUMBIGA1UEAwwLRWFzeS1SU0EgQ0EwHhcNMjAwMzE4MDMxNjI2WhcNMzAw
. . .
. . .
-----END CERTIFICATE----- 

```
Copy and paste all of that text (including begin and end certificate with the dashes)

In the other server that you created, run this command to create a filed called ca.crt
- nano /tmp/ca.crt
Paste the contents into that file and save it.

Staying on this server, run the following commands to iport the certificate from the CA server:
- sudo cp /tmp/ca.crt /usr/local/share/ca-certificates/
- sudo update-ca-certificates

# Installing OpenVPN and Easy-RSA




