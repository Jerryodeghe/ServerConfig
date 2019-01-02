## ServerConfig
## Server Configuration Steps for Ubuntu 16.04.5

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
     __*Change\Add:*__
     - `PasswordAuthentication no`
     - `AllowUsers root`
2. **Install Webmin/Virtualmin** 
   - `wget http://software.virtualmin.com/gpl/scripts/install.sh`
   - `sudo /bin/sh install.sh`
   - Access your site panel at `your-ip-address:10000`
   
3. **Install newer version of PHP:** 
   - `add-apt-repository ppa:ondrej/php && apt-get update`
   - `apt-get install php7.3 php7.3-mysql php7.3-cgi php7.3-cli php7.3-mbstring`
   - Sync up your server and bring virtualmin up to date by going to `System Settings -> Re-Check Config`
4. **Create a Virtualmin Server for your project:** 
You will need a virtualmin ***virtual*** server for your project to get started. This will install your project's Home Directory, DNS, File Manager, Mails, etc. ***Home directory is usually at: /home/projectname/public_html***
With Virtualmin, you can have as many projects as possible. <br />
To do this:<br />
`Click on the Virtualmin tab in your panel and click "Create Virtual Server `
 
5. **DOWNLOAD COMPOSER:** <br />
   Start by: <br />
   - `cd ~`<br />
   Download composer<br />
   - `curl -sS https://getcomposer.org/installer | php`<br />
   *And then move it to become accessible globally*<br />
   - `sudo mv composer.phar /usr/local/bin/composer`<br />
 
6. **GET GIT:** <br />
Chances are you already have git on your development server, however, to make the process of shipping your code from dev to prod servers, you need the two to talk to each other, seamlessly. <br />
Hence, this step.<br />
Start by going to a folder that is not accessible to the public:<br />
   - `cd /var`<br />
   Create and enter into a  directory named `repo`<br />
   - `mkdir repo && cd repo`<br />
   Inside, create another called `site.git` and go in there:<br />
   - `mkdir site.git && cd site.git`<br />
   And there init your git:<br />
   - `git init --bare`<br />
  The `bare` flag means the repository's main job is to receive pushes.<br /><br />
    **Set up git hooks**:<br />
   - `cd /var/repo/site.git/hooks`
   - `sudo nano post-receive`<br />
	**and write:**<br /><br />
     `#!/bin/sh`<br />
    `git --work-tree=/home/project-name/public_html --git-dir=/var/repo/site.git checkout -f`<br /><br />
    The purpose of the hook is to receive pushes into site.git and move them to your working tree, e.g `/home/cool-app/public_html`. Which means, anytime you push to the production server, it gets propagated/checked-out into your products directory.<br /><br />
**Lastly, give execution capacities to the file by:**<br />
   - `sudo chmod +x post-receive`<br />
   And:<br />
   - `exit`<br />

7. **SETUP LOCAL ENVIRONMENT FOR PUSH** 
Once you have your production setup with the hook, ready to receive, its time to set your local server to start sending.<br />
Logout of the remote server and get into your local project directory. And:
   - `git remote add production ssh://root@ip-address/var/repo/site.git`<br />
   You will receive a prompt asking for your password. Give it. Once this is done, you can start pushing locally committed changes to your production server by entering:
   - `git push production master`
 
8. **RUN COMPOSER**  <br />
     You've downloaded Composer, now is the time to install it. In your projects root, run:<br />
    - `composer install --no-dev` <br />
    **Set Permissions:** <br />
    - `sudo chown -R root:www-data /path/to/your/public_html`
    - `sudo find /path/to/your/public_html -type f -exec chmod 664 {} \; `
    - `sudo find /path/to/your/public_html -type d -exec chmod 775 {} \;`
    - `sudo chgrp -R www-data storage bootstrap/cache`
    - `sudo chmod -R ug+rwx storage bootstrap/cache`
    - `sudo chmod -R 777 storage`

9. **INSTALL PHPMYADMIN**  <br />
    - Move into a folder like `/home` and download the file from the source:
    - `wget https://files.phpmyadmin.net/phpMyAdmin/4.8.4/phpMyAdmin-4.8.4-all-languages.zip` <br />
    Note that the latest version might be different. Confirm the version before proceeding. <br />
    Unzip
    - `unzip phpMyAdmin-4.8.4-all-languages.zip` <br />
    Change the name to something easily remembered like: `phpmyadmin`
    - `cp -R phpMyAdmin-4.8.4-all-languages phpmyadmin` <br />
    Its location is at `/home/phpmyadmin`. <br /><br />
    Great, but it serves no useful purpose if it just stays there. <br />
    What you can do is to symlink it to the public folder of your project. And since this is Laravel app, it will be `/home/my-cool-app/public_html/public`. Like this: <br />
    - `sudo ln -s /home/phpmyadmin/ /home/my-cool-project/public_html/public`<br />
    With that, phpmyadmin can now be accessed using: `http://my-cool-app/phpmyadmin`<br />
    Next, copy and amend the sampled config file:<br />
    - `cp -R config.sample.inc.php config.inc.php` <br />
    Add something random to `$cfg['blowfish_secret']` property,like this:<br />
    - `$cfg['blowfish_secret']='sometthing random'`
**Create another user:** <br />
To manage your phpmyadmin account, you need to quickly whip up a database and its user:
   - `sudo mysql -p -u root`
   - `CREATE USER 'pmauser'@'%' IDENTIFIED BY 'password_here';`
   - `GRANT ALL PRIVILEGES ON *.* TO 'pmauser'@'%' WITH GRANT OPTION;` <br />
   That's it. Done. Now login to `http://your-domain/phpmyadmin`with your newly created credentials.
9. **CREATE A USER AND A DB FOR YOUR LARAVEL PROJECT**  <br />
Now that you have your phpmyadmin set up, its time to create a user and a db for your project. Sure its likely you have one already set up locally, but this is your production server.<br />
   Go to virtualmin and create a brand new database user, with localhost host and give the user every available priviledge. 
   - `https://your-ip:10000//mysql/edit_user.cgi?new=1&xnavigation=1` <br />
   Then attach the user to a database of your choice (or even create a new database)
   - `https://your-ip:10000/mysql/list_dbs.cgi?xnavigation=1`
