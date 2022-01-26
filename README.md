# Deploy Rails app with capistrano in aws 

### Here 'capistrano-aws' is the app name you can replace this with your app name

Step 1.  create a new rails app with postgresql
	       command:   
                   ** $ rails new capistrano-aws -d postgresql **
	 
Step 2.  create amazon instance(refer to amazon notes) and in security group keep "http" and "https" source "Anywhere"

Step 3.  Now in your localmachine create ssh key
	       command: 
                    ** $ ssh-keygen -t rsa **
	 
Step 4.  After the key is created just run 
	 command: cat ~/.ssh/id_rsa.pub
	 copy the key
	 
Step 5.  open your amazon instance in the terminal. Now in the aws ec2 run
	  command: sudo nano ~/.ssh/authorized_keys
	  Now delete all the data of the file and paste here the key you have copied in step 4.
	 press ctl+x and save the file.
	 
Step 6. To check the ssh authorizaton just run the following command in your local terminal
	 command: ssh your_aws_ec2_username@aws_ec2_public_ip
	 e.g: ssh ubuntu@52.53.213.21
	 if a warning appear just type yes and hit enter
	 You should see a message like "Last login: Fri Dec 10 05:58:24 2021 from 24.7.22.30
	 and you should be in your aws_ec2_instance terminal(not in local terminal)
	 
**********************************  Install curl and required packagess  *****************************************************************
	#In your aws_ec2_instance
Step 7. commands:
	 	$ sudo apt install curl
           	$ sudo apt install zlib1g-dev build-essential libssl-dev libreadline-dev
           	$ sudo apt install libyaml-dev libsqlite3-dev sqlite3 libxml2-dev
           	$ sudo apt install libxslt1-dev libcurl4-openssl-dev
           	$ sudo apt install software-properties-common libffi-dev nodejs
           	
************************************   Install and config ruby with rvm  *****************************************************************
	commands:
	        $ sudo apt update
		$ sudo apt install curl
		$ curl -sSL https://rvm.io/mpapis.asc | gpg --import -
		$ curl -sSL https://rvm.io/pkuczynski.asc | gpg --import -
		$ curl -sSL https://get.rvm.io | bash -s stable
		$ source ~/.bashrc
		$ rvm install 3.0.3
		$ /bin/bash --login
		$ rvm use 3.0.3 --default
		$ ruby -v
		$ gem install bundler
		$ gem install rails
		       
****************************************    Install postgresql and libraries  ***********************************************************
		$ sudo apt install postgresql postgresql-client-12
		$ sudo apt install libpq-dev
		$ sudo -u postgres createdb your_app_name()
		$ sudo -u postgres createuser --interactive
		  Enter name of role to add: ubuntu
		  Shall the new role be a superuser? (y/n) y
		
****************************************    Insatall Yarn  *******************************************************************************
		$ curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
		$ echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
		$ sudo apt update
		$ sudo apt install yarn
		
******************************************  Install nodejs  ******************************************************************************
		$ wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash
		$ export NVM_DIR="$HOME/.nvm"
		$ [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
		$ [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion" bash_completion
	* At this point, the NVM is ready to use. To get a list of all versions available for installation:
		$ nvm list-remote
	* Choose a version and install:
		$ nvm install 14.16.0
		
*************************************** Ssh key of aws ec2 connection with git ***********************************************************

Step 8.  Generate ssh key in aws_ec2_instance
 	    command: ssh-keygen -t rsa
 	    
Step 9.  Show the ssh-key 
 	    command: cat ~/.ssh/id_rsa.pub
          *Copy this key.
          
Step 10. Now open your git repo(which you want to deploy) and go to settings of repo(left side of Insights)

Step 11. Go to "Deploy keys" option and click on "Add deploy key" button

Step 12. Give the title of your choice and in key section just paste the ssh-key of your amazon_ec2 instance which you have 
         copied in Step 9.
 	 * Now click on "Add key" button *(Enter Password if asks)
 	 
Step 13. Now in your aws_ec2 instance run the following command
 	    command: ssh -T git@github.com
 	             *and type "yes" if asked(without quotes)
 		  
***************************************** Rails app(capistrano-aws) configuration *******************************************************
     
Step 14. In your Gemfile

     	     1. Add the following gems in group development :-----
     		  	gem 'capistrano', require:  false
  			gem 'capistrano-rvm', require:  false
  			gem 'capistrano-rails', require:  false
  			gem 'capistrano-bundler', require:  false
  			gem 'capistrano3-puma', require:  false	
  	  ** Open terminal in capistrano-aws app and run the following command		
                       $ bundle install
       		$ cap install
       		
*****************************************************************************************************************************************
Step 15. In Capfile paste the following before line of stars(after deleting all contents of capfile)
        
# Load DSL and set up stages
require "capistrano/setup"

# Include default deployment tasks
require "capistrano/deploy"

require 'capistrano/rails'
require 'capistrano/bundler'
require 'capistrano/rvm'

# Load the SCM plugin appropriate to your project:
#
# require "capistrano/scm/hg"
# install_plugin Capistrano::SCM::Hg
# or
# require "capistrano/scm/svn"
# install_plugin Capistrano::SCM::Svn
# or
require "capistrano/scm/git"
install_plugin Capistrano::SCM::Git

require 'capistrano/puma'
#install_plugin Capistrano::Puma

# Include tasks from other gems included in your Gemfile
#
# For documentation on these, see for example:
#
#   https://github.com/capistrano/rvm
#   https://github.com/capistrano/rbenv
#   https://github.com/capistrano/chruby
#   https://github.com/capistrano/bundler
#   https://github.com/capistrano/rails
#   https://github.com/capistrano/passenger
#
# require "capistrano/rvm"
# require "capistrano/rbenv"
# require "capistrano/chruby"
# require "capistrano/bundler"
# require "capistrano/rails/assets"
# require "capistrano/rails/migrations"
# require "capistrano/passenger"

#install_plugin Capistrano::Puma::Daemon

# Load custom tasks from `lib/capistrano/tasks` if you have any defined
Dir.glob("lib/capistrano/tasks/*.rake").each { |r| import r }

**************************************************************************************************************************************** 
Step 16. In deploy.rb file paste the following before line of stars(after deleting all contents of capfile)  
	
# Change these
server 'your public ip of amazon_ec2_instance', port: 22, roles: [:web, :app, :db], primary: true

set :repo_url,        '<Your git-repo>'
set :application,     'Your project name'

set :rbenv_ruby,      '3.0.2'
# set :rbenv_ruby_dir,  '/home/ubuntu/.rbenv/versions/3.0.2'
set :default_env, { path: "~/.rbenv/shims:~/.rbenv/bin:$PATH" }

# If using Digital Ocean's Ruby on Rails Marketplace framework, your username is 'rails'
set :user,            'ubuntu'
set :puma_threads,    [4, 16]
set :puma_workers,    0

# Don't change these unless you know what you're doing
set :pty,             true
set :use_sudo,        false
set :stage,           :production
set :deploy_via,      :remote_cache
set :deploy_to,       "/home/#{fetch(:user)}/apps/#{fetch(:application)}"
set :puma_bind,       "unix://#{shared_path}/tmp/sockets/#{fetch(:application)}-puma.sock"
set :puma_state,      "#{shared_path}/tmp/pids/puma.state"
set :puma_pid,        "#{shared_path}/tmp/pids/puma.pid"
set :puma_access_log, "#{release_path}/log/puma.access.log"
set :puma_error_log,  "#{release_path}/log/puma.error.log"
set :ssh_options,     { forward_agent: true, user: fetch(:user), keys: %w(~/.ssh/id_rsa.pub) }
set :puma_preload_app, true
set :puma_worker_timeout, nil
set :puma_init_active_record, true  # Change to false when not using ActiveRecord

append :rbenv_map_bins, 'puma', 'pumactl'

## Defaults:
# set :scm,           :git
# set :branch,        :main
# set :format,        :pretty
# set :log_level,     :debug
# set :keep_releases, 5

## Linked Files & Directories (Default None):
# set :linked_files, %w{config/database.yml}
# set :linked_dirs,  %w{bin log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system}

namespace :puma do
  desc 'Create Directories for Puma Pids and Socket'
  task :make_dirs do
    on roles(:app) do
      execute "mkdir #{shared_path}/tmp/sockets -p"
      execute "mkdir #{shared_path}/tmp/pids -p"
    end
  end

  before 'deploy:starting', 'puma:make_dirs'
end

namespace :deploy do
  desc "Make sure local git is in sync with remote."
  task :check_revision do
    on roles(:app) do

      # Update this to your branch name: master, main, etc. Here it's main
      unless `git rev-parse HEAD` == `git rev-parse origin/master`
        puts "WARNING: HEAD is not the same as origin/master"
        puts "Run `git push` to sync changes."
        exit
      end
    end
  end

  desc 'Initial Deploy'
  task :initial do
    on roles(:app) do
      before 'deploy:restart', 'puma:start'
      invoke 'deploy'
    end
  end

  desc 'Restart application'
    task :restart do
      on roles(:app), in: :sequence, wait: 5 do
        invoke 'puma:restart'
      end
  end

  before :starting,     :check_revision
  after  :finishing,    :compile_assets
  after  :finishing,    :cleanup
  # after  :finishing,    :restart
end

# ps aux | grep puma    # Get puma pid
# kill -s SIGUSR2 pid   # Restart puma
# kill -s SIGTERM pid   # Stop puma       		

******************************************************************************************************************************************
Step 16. Create file 'nginx.conf' in config folder(in your rails app) and paste the following in nginx.conf 

upstream puma {
  server unix:///home/ubuntu/apps/project_name/shared/tmp/sockets/project_name-puma.sock;
}

server {
  listen 80 default_server deferred;

  # If you're planning on using SSL (which you should), you can also go ahead and fill out the following server_name variable:
  # server_name example.com;

  # Don't forget to update these, too
  root /home/ubuntu/apps/project_name/current/public;
  access_log /home/ubuntu/apps/project_name/current/log/nginx.access.log;
  error_log /home/ubuntu/apps/project_name/current/log/nginx.error.log info;

  location ^~ /assets/ {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

  try_files $uri/index.html $uri @puma;
  location @puma {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;

    proxy_pass http://puma;
  }

  error_page 500 502 503 504 /500.html;
  client_max_body_size 10M;
  keepalive_timeout 10;
}

******************************************************************************************************************************************

Step 17. Config and push code to github
 		command:- 
 			$ git remote add origin your_repo_url	
 			$ git checkout origin master
 			$ git add .
 			$ git commit -m "initial commit"
 			$ git push origin master

Step 18. Add local secret key base in aws_ec2 instance
	 	command:-
	 	 **In local
	 	 	$ rake secret'
	 	 **Copy this key
	 	 
	 	 **In aws_ec2 Instance terminal
	 	 	$ sudo nano /etc/environment
	 	 *bottom of PATH  write SECRET_KEY_BASE=your copied secret key(from local) **Don't delete anything from this file
	 	  press ctrl+x and save this file

Step 19. Install nginx in aws_ec2_instance
	  command: 
	  	 $ sudo apt install nginx
	  	 $ sudo rm /etc/nginx/sites-enabled/default

Step 20. Deploy app
	 command: 
	 **In local
	         $ cap production deploy:initial --trace
	  **If 'uninitialized constant Capstrano::Puma' error occurs just install the capistrano3-puma gem globally by running the 	    following command:
	  	$ gem install capistrano3-puma
Step 20. Nginx configuration
	  commands: 
	  **In aws_ec2_instance terminal
	  	$ sudo ln -nfs "/home/ubuntu/apps/capistrano-aws/current/config/nginx.conf" "/etc/nginx/sites-enabled/capistrano"
	  	$ sudo service nginx restart
	  	
******************** Congratulations you have successfully deployed your rails app********************************************************

Now just go to your aws_ec2 public address and you will see there your rails app

Note:- you can check your nginx logs by the following command:
		$ vim apps/capistrano-aws/current/log/production.log
