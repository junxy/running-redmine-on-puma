# Important notes

This installation guide was created for and tested on **Debian/Ubuntu** operating systems. 

- - -

# Overview

The Redmine installation consists of setting up the following components:

1. Packages / Dependencies
2. Ruby
3. System Users
4. Database
5. Redmine
6. Nginx


# 1. Packages / Dependencies

Install the required packages:

    sudo apt-get install vim curl git-core imagemagick libmagickwand-dev

# 2. Ruby

Remove old 1.8 ruby if present

    sudo apt-get remove ruby1.8

Use rvm Install:

    curl -L get.rvm.io | bash -s stable
    source ~/.bashrc
    source ~/.bash_profile
    source ~/.profile

    rvm install 1.9.3
    
Install the Bundler Gem:

    sudo gem install bundler


# 3. System Users

Create a `redmine` user for Redmine:

    sudo adduser --disabled-login --gecos 'Redmine' redmine


# 4. Database(MySQL)

    # Install the database packages
    sudo apt-get install -y mysql-server mysql-client libmysqlclient-dev

    # Login to MySQL
    mysql -u root -p

    # Create a user for Redmine. (change $password to a real password)
    mysql> CREATE USER 'redmine'@'localhost' IDENTIFIED BY '$password';

    # Create the Redmine production database
    mysql> CREATE DATABASE IF NOT EXISTS `redmine_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;

    # Grant the Redmine user necessary permissions on the table.
    mysql> GRANT SELECT, LOCK TABLES, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON `redmine_production`.* TO 'redmine'@'localhost';

    # Quit the database session
    mysql> \q

    # Try connecting to the new database with the new user
    sudo -u git -H mysql -u redmine -p -D redmine_production

# 5. Redmine

	# We'll install Redmine into home directory of the user "redmine"
    cd /home/redmine
    sudo su redmine

##Use rvm:

    curl -L get.rvm.io | bash -s stable
    source ~/.bashrc
    source ~/.bash_profile
    source ~/.profile

    rvm install 1.9.3
    rvm use 1.9.3 --default 

## Clone the Source

    # Clone Redmine repository
    git clone https://github.com/redmine/redmine.git

    # Go to redmine dir
    cd /home/redmine/redmine

    # Checkout to stable release
    sudo -u redmine -H git checkout 2.3-stable

**Note:**
You can change `2.3-stable` to `master` if you want the *bleeding edge* version, but
do so with caution!

## Configure it

    cd /home/redmine/redmine

    # Make sure Redmine can write to the log/ and tmp/ directories
    sudo chown -R redmine log/
    sudo chown -R redmine tmp/
    sudo chmod -R u+rwX  log/
    sudo chmod -R u+rwX  tmp/

    # Create directories for sockets/pids and make sure Redmine can write to them
    mkdir tmp/pids/
    mkdir tmp/sockets/
    sudo chmod -R u+rwX  tmp/pids/
    sudo chmod -R u+rwX  tmp/sockets/

    # Copy the example Puma config
    curl --output /home/redmine/redmine/config/puma.rb https://raw.github.com/junxy/running-redmine-on-puma/master/config/redmine/puma.rb.example

**Important Note:**


## Configure Redmine DB settings

    # Mysql
    vim config/database.yml

Make sure to update username/password in config/database.yml.

## Install Gems

    cd /home/redmine/redmine

    # For MySQL (note, the option says "without")
    bundle install --without development test postgresql sqlite

## Session store secret generation
    
    rake generate_secret_token

## Database schema objects creation

    RAILS_ENV=production rake db:migrate

Database default data set
    
    RAILS_ENV=production rake redmine:load_default_data

## Install Init Script

Download the init script (will be /etc/init.d/redmine):

	#exit redmine user
	exit

    sudo curl --output /etc/init.d/redmine https://raw.github.com/junxy/running-redmine-on-puma/master/config/init.d/redmine
    sudo chmod +x /etc/init.d/redmine

Make Redmine start on boot:

    sudo update-rc.d redmine defaults 22

## Test the installation

    ruby script/rails server webrick -e production

## Start Your Redmine Instance

    sudo service redmine start
    # or
    sudo /etc/init.d/redmine restart


# 6. Nginx

**Note:**
If you can't or don't want to use Nginx as your web server, have a look at the
[`Advanced Setup Tips`](./installation.md#advanced-setup-tips) section.

## Installation
    sudo apt-get install nginx

## Site Configuration

Download an example site config:

    sudo curl --output /etc/nginx/sites-available/redmine https://raw.github.com/junxy/running-redmine-on-puma/master/config/nginx/redmine
    sudo ln -s /etc/nginx/sites-available/redmine /etc/nginx/sites-enabled/redmine

Make sure to edit the config file to match your setup:

    # **YOUR_SERVER_FQDN** to the fully-qualified
    # domain name of your host serving Redmine. Also, replace
    # the 'listen' line with the following:
    #   listen 80 default_server;         # e.g., listen 192.168.1.1:80;
    sudo vim /etc/nginx/sites-available/redmine

## Restart

    sudo service nginx restart


# Done!

Visit YOUR_SERVER for your first Redmine login. You can use it to log in:
    admin/admin

**Important Note:**
Please go over to your profile page and immediately change the password, so
nobody can access your Redmine by using this login information later on.

**Enjoy!**
