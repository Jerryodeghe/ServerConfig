## Laravel Server Configuration (LAMP)
## Configuration Steps for Ubuntu 16.04.5

 1. **Setup SSH** 
   - **Remove old keys after server re-installation:** 
	   `ssh-keygen -R your-ip-address`

   -  **Copy the public keys to the server:**
`   cat ~/.ssh/id_rsa.pub | ssh username@remote_host "mkdir -p ~/.ssh && touch 	~/.ssh/authorized_keys && chmod -R go= ~/.ssh && cat >> ~/.ssh/authorized_keys"`

   - **Access the server with:** 
   `ssh root@ip`
   - **Update the sshd config file and prevent the use of password in login**
     - `cd /etc/ssh`
     - `nano sshd_config`
     - __*Change\Add:*__
     - `PasswordAuthentication no`
     - `AllowUsers root`
