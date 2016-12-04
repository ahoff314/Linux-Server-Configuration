# Linux Server Configuration

Public IP: 35.164.165.70
SSH PORT: 2200
URL: http://ec2-35-164-165-70.us-west-2.compute.amazonaws.com/

## 1) Create Development Environment and SSH Into Server
Download Private Key

Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory)

`mv ~/Downloads/udacity_key.rsa ~/.ssh/`

Open your terminal and type in

`chmod 600 ~/.ssh/udacity_key.rsa`

SSH into the server

`ssh -i ~/.ssh/udacity_key.rsa root@<PUBLIC_IP>`
