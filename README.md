### Action Cable Demo (Remote Deploy)

An adaptation of the [tutorial](https://www.youtube.com/watch?v=n0WUjGkDFS0) *Rails 5: Action Cable Demo* by *David Heinemeier Hansson*,  
[deployed](dhhpgaction.tomgdow.com) remotely with PostgreSQL as database

For a version of this tutorial with local deployment,  
see https://github.com/tomGdow/ActionCable_dhh_tutorial 

### Method 

**1. Local Development**

Developed locally on a virtual machine running Xubuntu 16.04,  
Rails 5.0.0.1 and  Ruby 2.3.1, as follows: 

    okpsql     # script to check if PostgreSQL is installed properly.  OK to skip

&hellip;

    rails new acdhhpg -d postgresql
    cd acdhhpg

> /Gemfile  

uncomment   

    gem 'redis'  

add to development group 

    gem 'pry-rails'

&hellip;  


> config/database.yml

set up as follows:

    default: &default
      adapter: postgresql
      encoding: unicode
      pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

    development:
      <<: *default
      database: acdhhpg_development
      username: <username>
      password: <%= ENV['POSTGRES_PASSWORD'] %>
      host: localhost
      port: 5432

    test:
      <<: *default
      database: acdhhpg_test
      username: <username> 
      password: <%= ENV['POSTGRES_PASSWORD'] %>
      host: localhost
      port: 5432

    production:
      <<: *default
      database: acdhhpg_production
      username: <username>
      password: <%= ENV['POSTGRES_PASSWORD'] %>

&hellip;  

    bundle install
    rake db:create:all

&hellip;  

    rails g controller rooms show

**Follow the method described in:**  

 https://github.com/tomGdow/ActionCable_dhh_tutorial

**from and including this step:**  

> /config/routes.rb
    
    root to: 'rooms#show'

**up to and including this step** 

> app/assets/javascripts/channels/room.coffee

    received: (data) -> 
      $('#messages').append data['message']

**2. Prepare for Remote Deployment**

>  /app/views/layouts/application.html.erb

add the following, before the 'javascript_include_tag'

    <%= action_cable_meta_tag %>
 
> config/environments/development.rb

add the following

    config.action_cable.url = "ws://your-site-url:4000/cable"
    config.action_cable.allowed_request_origins = ['http://your-site-url']

**3 Remote Server**

Transfer all files to remove site, then

    sudo cp -R acdhhpg /var/www/html
    sudo chown -R \$USER:\$USER /var/www/html/acdhhpg/
    
> /etc/apache2/sites-available

    touch acdhhpg.conf  

set up as  follows:

    <VirtualHost *:80>
      
      ServerAdmin webmaster@localhost
      ServerName  <site-url> 
      ServerAlias <site-url-alias>
      DocumentRoot /var/www/html/acdhhpg/public
      PassengerRuby /home/username/.rvm/gems/ruby-2.3.1/wrappers/ruby

      ProxyPass / http://localhost:4000/
      ProxyPassReverse / localhost:4000/
      ProxyPreserveHost on 

      <Proxy *>
        Order deny,allow
        Allow from all
      </Proxy>

      <Directory /var/www/html/acdhhpg/public>
        AllowOverride all 
        Options -MultiViews
        require all granted
      </Directory>

      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined

      </VirtualHost>

&hellip;  

    a2ensite acdhhpg
    sudo service apache2 restart

> /var/www/html/acdhhpg  
 
    bundle install
    rake db:create:all
    rails db:migrate

    rails s -d -p 4000 -b 0.0.0.0

### Deployed Site


http://dhhaction.tomgdow.com

### Notes

[Deployed](http://dhhaction.tomgdow.com) on [Digital Ocean](https://www.digitalocean.com/) with Puma as stand-alone server behind a reverse proxy on an  
 Apache/Phusion Passenger server (see [here](https://www.phusionpassenger.com/library/deploy/standalone/reverse_proxy.html))   

**PostgreSQL**

The method requires that the PostgreSQL username is the 
same  as the current unix/Ununtu login name  

 For example:  

    sudo -u postgres createuser -s <unix_ubuntu_username>
    sudo -u postgres psql
    \password <unix_ubuntu_username>  

**ENV Variable**

Set an ENV variable, here called POSTGRES_PASSWORD, equal to the PostgreSQL password  
For example:

    POSTGRES_PASSWORD='my_postgres_password'
    export POSTGRES_PASSWORD'  

Or, include the following in .bashrc  

    export POSTGRES_PASSWORD='my_postgres_password'
