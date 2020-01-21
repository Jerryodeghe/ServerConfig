
## Laravel ~ 5.6 Server Config for Ubuntu 16.04.5

 1. **Setup SSH** 
     - **Remove old keys after server re-installation:** 
	   `ssh-keygen -R your-ip-address`

     -  **Copy the public keys to the server:**
`   cat ~/.ssh/id_rsa.pub | ssh username@remote_host "mkdir -p ~/.ssh && touch 	~/.ssh/authorized_keys && chmod -R go= ~/.ssh && cat >> ~/.ssh/authorized_keys"`

     - **Access the server with: ssh root@ip-address** 
     
     - **Alternatively**
     You can manually install the rsa keys into the server.
     - Generate both the public and private keys with Puyyt Gen.
     - Save both on your PC.
     - Add the public key to the authorized_file on the server by:
     	- `cd /~/.ssh`
	- `nano authorized_keys`
	- Restart:  sudo /etc/init.d/ssh start
	- Exit. And,
	- Add the Private key to your putty client. Make sure to add the 'Auto-login username' in the Data tab, under Connection.
	- Try to login. You should be automatically logged in.
   `ssh root@ip`
     - **Update the sshd config file and prevent the use of password in login**
     - `cd /etc/ssh`
     - `nano sshd_config` <br /><br />
     __*Change\Add:*__
     - `PasswordAuthentication no`
     - `AllowUsers root`
2. **Install Webmin/Virtualmin** 
   - `wget http://software.virtualmin.com/gpl/scripts/install.sh`
   - `sudo /bin/sh install.sh`
   - Access your site panel at `your-ip-address:10000`
   
3. **Install newer version of PHP:** 
   - `add-apt-repository ppa:ondrej/php && apt-get update`
   - `apt-get install php7.3 php7.3-mysql php7.3-cgi php7.3-cli php7.3-mbstring sudo php7.3-xml`
   - Sync up your server and bring virtualmin up to date by going to `System Settings -> Re-Check Config`
   ** - To change another version globally, use:
   	`sudo update-alternatives --config php`
	Then select the version of choice.
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
    - `cp -R phpMyAdmin-4.8.4-all-languages phpmyadmin` (remember to use something different, like pMYadmin, for security reasons)<br />
    Its location is at `/home/phpmyadmin`. <br /><br />
    Great, but it serves no useful purpose if it just stays there. <br />
    What you can do is to symlink it to the public folder of your project. And since this is Laravel app, it will be `/home/my-cool-app/public_html/public`. Like this: <br />
    - `sudo ln -s /home/phpmyadmin/ /home/my-cool-project/public_html/public`<br />
    With that, phpmyadmin can now be accessed using: `http://my-cool-app/phpmyadmin`<br />
    Next, copy and amend the sampled config file:<br />
    - cd into the phpmyadmin directory and:
    - `cp -R config.sample.inc.php config.inc.php` <br />
    Add something random to `$cfg['blowfish_secret']` property,like this:<br />
    - `$cfg['blowfish_secret']='sometthing random'`<br />
**Create another user:** <br />
To manage your phpmyadmin account, you need to quickly whip up a database and its user:
   - `sudo mysql -p -u root`
   - `CREATE USER 'pmauser'@'%' IDENTIFIED BY 'password_here';`
   - `GRANT ALL PRIVILEGES ON *.* TO 'pmauser'@'%' WITH GRANT OPTION;` <br />
   That's it. Done. Now login to `http://your-domain/phpmyadmin`with your newly created credentials.
10. **CREATE A USER AND A DB FOR YOUR LARAVEL PROJECT**  <br />
Now that you have your phpmyadmin set up, its time to create a user and a db for your project. Sure its likely you have one already set up locally, but this is your production server.<br />
===The next section may no longer be necessary==============================<br />
Looks like Virtualmin would have created both the database and its user during installation.
If the links below does not work, check your the "Edit Databases" link on Virtualmin, click THE "Passwords" Tab above it. Click the key icon to reveal the generated password for this database user. The user carries the name of the database. Now, if phpmyadmin does not recognze this, go to its Priviledges, and make sure the user has 'localhost' as the host and that the user has 'All Priviledges'. Give it, if it doesnt already have it.<br />
==============================================<br />
   Go to virtualmin and create a brand new database user, with localhost host and give the user every available priviledge. 
   - `https://your-ip:10000//mysql/edit_user.cgi?new=1&xnavigation=1` <br />
   Then attach the user to a database of your choice (or even create a new database)
   - `https://your-ip:10000/mysql/list_dbs.cgi?xnavigation=1`
11. **EDIT YOUR .ENV FILE**  <br />  
Inside your project root folder, 
   - `ls -A ` <br />
To show a list of all your files including the hidden ones. You would notice that there is no .env present. Yes, its available locally, but Laravel adds it in the `gitignore` file which means it was ignored by git when you were moving your project earlier. This is actually a good thing, as it means you can safely add some sensitive info to your local .env without the fear of getting to production or even available on your remote git client. <br /> To create a production copy of your .env file simply:
 - `cp -R .env.example .env` <br />
 - `nano .env`
And edit as appropriate, e g:
 - `APP_ENV=production` 
 - `APP_DEBUG=false`
 - `DB_DATABASE=yourdb`
 - `DB_PASSWORD=yourpsw`
  You now need to generate an APP:KEY and cache your config:
   - `php artisan key:generate`
   - `php artisan config:cache` <br />
   And lastly, migrate:
   - `php artisan migrate` <br />
  
  12. **Update and Link Your Apache to the /public forlder**  <br />  
  Laravel consumes all its public files including the index file from the /public folder. Lets tell apache to make this folder the main entry point.
  - Go to Webmin. Under Servers, click on 'Apache Webserver'
  -Look for the virtual server you are trying to edit, and click on it.
  - Under the Path table, click on the path that looks like this: 'home/your-project/public_html/' and click on it.
  - Look for "Edit Directives", and click on it.
  - In the file that opens, look for 'DocumentRoot /home/scms/public_html/' and append 'public' to the path... like so:
  - `DocumentRoot /home/scms/public_html/public`
  - Press "Click and Save" and restart your apache like so:
  - `/etc/init.d/apache2 restart`
  
  13. **Add SSL**  <br />  
  	- To add Lets Encrtpt certificates, first make sure that your virtual server has 'Apache SSL website enabled'. Check this by Clicking on 'Edit Virtual Server and clicking expanding "Enabled Features". If SSL is not enabled, do so.
	
	- Now, go to "Server Configuration -> SSL Certificate"
	- Click on the "Lets Encrypt". If you see an error that says something like: "The Let's Encrypt client command letsencrypt or certbot was not found on your system", simply add it like so: <br />
	`apt-get -y install certbot`, 
	- 
## OPTIONAL SETUPS
1. **Add bash aliases for shortcuts**  <br />  
   - Go to ` cd ~` and check if `.aliases` file is already on the server. To check, especially since this is a hidden file, simply `ls -A`
   -`nano .aliases` and add these shortcuts, one per line. You can add as many as possible:
  - `alias par='php artisan route:list'`
  - `alias pam='php artisan migrate'`
  - `alias pamroll='php artisan migrate:rollback'`
  - `alias pamref='php artisan migrate:refresh'`
  - `alias pa='php artisan $*'`
  - `alias mod='php artisan make:model $*'`
  - `alias mig='php artisan make:migration $*'`
  - `alias pat='php artisan tinker'`
  - `alias con='php artisan make:controller $* '`
  - `alias g.='git add .'`
  - `alias gl='git log'`
  - `alias gs='git status'`
  - `alias gc='git commit -m $*'`
  - `alias .='clear'`
  - `alias fresh='php artisan migrate:fresh'`
  - `alias refresh='php artisan migrate:refresh --seed'`
  - `alias restart='service apache2 restart'`
  - `alias dump='composer dump-autoload'`
  -  `alias home='/home/MY-PROJECT/public_html/'`
  - `alias cache='php artisan config:cache'`
  - `alias ccache='php artisan config:clear'`
  - `alias clearlog='truncate -s 0   /home/buzzwords/public_html/storage/logs/laravel.log'`
  - `alias unalias='alias /d $1'`
**Edit bashrc**
  - `nano ~/.bashrc` <br />
	add as the first line: <br />
 - `. ~/.aliases` 
Exit nano and:
- `source .aliases` <br />
to recompile and add the aliases to your server. <br />
To see this in action, type `pam` while in your project directory and see `php artisan migrate` run.

2. **INSTALL REDIS**  <br />  
Get up to date:
   - `sudo apt-get update` 
   - `sudo apt-get install build-essential tcl` <br />
 Download to a temporary location:
   - `cd /tmp`
   - `curl -O http://download.redis.io/redis-stable.tar.gz tar xzvf redis-stable.tar.gz`
  - "If it complains that the url cant be resolved, ignore. It has actually been downloaded. Just do this to untar/unzip":
  	- `tar xzvf redis-stable.tar.gz`
   - `cd redis-stable`
   - `make`
   - `make test`
   - `sudo make install`
   - `sudo mkdir /etc/redis`
   - `sudo cp /tmp/redis-stable/redis.conf /etc/redis` <br />
	**Change few settings:**
	`sudo nano /etc/redis/redis.conf` <br />
	Uncomment `supervised` and change it to :
  - `supervised systemd` <br />
  Also uncomment `dir` and change it to:
  - `dir /var/lib/redis` <br />
	Save and close
**Create a systemd unit file:**
  - `sudo nano /etc/systemd/system/redis.service` <br />
  And add the following: <br />

    [Unit] <br />
    Description=Redis In-Memory Data Store<br />
    After=network.target<br /><br />
    [Service]<br />
    User=redis<br />
    Group=redis<br />
    ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf<br />
    ExecStop=/usr/local/bin/redis-cli shutdown<br />
    Restart=always<br /><br />
    
    [Install]<br />
    WantedBy=multi-user.target<br />

**Save and close** <br />
Next, create the dir specified above:
   - `sudo mkdir /var/lib/redis` <br />
Create a redis user:
   - `sudo adduser --system --group --no-create-home redis`
   - `sudo chown redis:redis /var/lib/redis`
   - `sudo chmod 770 /var/lib/redis` <br />
 Start redis service:
   - `sudo systemctl start redis`
   - `sudo systemctl status redis` <br />
	To test again:
   - `sudo systemctl restart redis` <br />
	Enable to start at boot:
	- `sudo systemctl enable redis`
3. **INSTALL PHPREDISADMIN**  <br />  
To get an admin panel for your redis instances, you need a script just like `phpmyadmin`. Its aptly named `phpredisadmin` <br />
   - Create an empty directory outside your projects home, eg: `/home/redis` just like you did for `phpmyadmin` <br />
Again, symlink this link into your projects public directory:
  - `sudo ln -s /home/redis/ /home/project-name/public_html/public` <br />
Then go to your project directory (where your vendor folder is, e.g /home/buzz/public_html)
And run: <br />
`composer create-project -s dev erik-dubbelboer/php-redis-admin /home/redis` <br />
**SET UP SECURITY** <br />
    - Go to:<br />
`cd /home/redis/includes` <br />
and copy config.sample.inc.php into config.inc.php <br />
`cp -R config.sample.inc.php config.inc.php` <br />
    Open it and uncomment the 'login' array, then add/set the password, username.
    - `nano config.inc.php` <br />
    To get a form based login, change the `cookie_auth` value to:
    - `cookie_auth=> true`<br />
   **Thats it! Access your panel at: your-ip/redis**
4. **Use Faker in production**  <br />
I have found that by default, the `composer install --no-dev ` command will ignore all developmental packages like phpunit, faker, etc. But if for whatever reason you need to use the faker package in production, then you need to:<br />
    - Move `"fzaninotto/faker": "^your-version-digit",` away from the `require-dev` array to the `require` array.
    - Push to the server and run `composer update --no-dev` on the server.
    - If it asks for any php installations, as required by even your development packages, give it. The reason is, even though you use 	the `--no-dev` flag, composer would still require that all php extensions required by all the packages are duly installed, again, even though you will not be needing them in production. This could be ignored when you ran `composer install` but not when you ran `composer update`. Weird.
    
