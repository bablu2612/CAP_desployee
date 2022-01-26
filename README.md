
# Deploy Rails app with capistrano in aws   
### Here 'capistrano-aws' is the app name you can replace this with your app name  

**Step 1.**  create a new rails app with postgresql  
command:  
**$ rails new capistrano-aws -d postgresql**  

**Step 2.**  create amazon instance(refer to amazon notes) and in security group keep "http" and "https" source "Anywhere"  

**Step 3.**  Now in your localmachine create ssh key  
command:   
**$ ssh-keygen -t rsa**  

**Step 4.**  After the key is created just run   
command:  
**$cat ~/.ssh/id_rsa.pub**  
copy the key  

**Step 5.**  open your amazon instance in the terminal. Now in the aws ec2 run  
command:  
**$ sudo nano ~/.ssh/authorized_keys**  
Now delete all the data of the file and paste here the key you have copied in step 4.    
press ctl+x and save the file.  

**Step 6.** To check the ssh authorizaton just run the following command in your local terminal  
command:  
**$ ssh your_aws_ec2_username@aws_ec2_public_ip**  
	 e.g: ssh ubuntu@52.53.213.21  
	 if a warning appear just type yes and hit enter  
	 You should see a message like "Last login: Fri Dec 10 05:58:24 2021 from 24.7.22.30  
	 and you should be in your aws_ec2_instance terminal(not in local terminal)  



	 
## Install curl and required packages
### In your aws_ec2_instance
**Step 7.**  
commands:  
**$ sudo apt install curl
$ sudo apt install zlib1g-dev build-essential libssl-dev libreadline-dev
$ sudo apt install libyaml-dev libsqlite3-dev sqlite3 libxml2-dev
$ sudo apt install libxslt1-dev libcurl4-openssl-dev
$ sudo apt install software-properties-common libffi-dev nodejs**  
           	
## Install and config ruby with rvm
commands:  
	**$ sudo apt update
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
	$ gem install rails**
		       
## Install postgresql and libraries
**$ sudo apt install postgresql postgresql-client-12
$ sudo apt install libpq-dev
$ sudo -u postgres createdb your_app_name()  
$ sudo -u postgres createuser --interactive**  
Enter name of role to add: ubuntu  
Shall the new role be a superuser? (y/n) y  
		
## Install Yarn  
**$ curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -  
$ echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list  
$ sudo apt update  
$ sudo apt install yarn**  
## Install nodejs  
**$ wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash  
$ export NVM_DIR="$HOME/.nvm"  
$ [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  
$ [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion" bash_completion**   
* At this point, the NVM is ready to use. To get a list of all versions available for installation:  
	**$ nvm list-remote **  
* Choose a version and install:  
	**$ nvm install 14.16.0  **  
		
## Ssh key of aws ec2 connection with git  

**Step 8.**  Generate ssh key in aws_ec2_instance    
command:    
**$ ssh-keygen -t rsa**  
 	    
**Step 9.** Show the ssh-key   
command:   
**$ cat ~/.ssh/id_rsa.pub**  
*Copy this key.  
          
**Step 10.** Now open your git repo(which you want to deploy) and go to settings of repo(left side of Insights)  

**Step 11.** Go to "Deploy keys" option and click on "Add deploy key" button  

**Step 12.** Give the title of your choice and in key section just paste the ssh-key of your amazon_ec2 instance which you have copied in Step 9.    
 * Now click on "Add key" button *(Enter Password if asks)  
 	 
**Step 13.** Now in your aws_ec2 instance run the following command  
command:   
**$ ssh -T git@github.com**  
*and type "yes" if asked(without quotes)  
 		  
## Rails app(capistrano-aws) configuration  
     
**Step 14.** In your Gemfile  

1. Add the following gems in group development  
**gem 'capistrano', require:  false    
gem 'capistrano-rvm', require:  false  
gem 'capistrano-rails', require:  false  
gem 'capistrano-bundler', require:  false  
gem 'capistrano3-puma', require:  false**  
* Open terminal in capistrano-aws app and run the following command      		
**$ bundle install  
**$ cap install**   

**Step 15.** Copy the content of the Capfile of this repo and paste it to the Capfile of your rails app  
 
**Step 16.** Copy the content of the deploy.rb(in config folder) of this repo and paste it to the deploy.rb(in config folder) of your rails app  
	  ** Don't forget to change the following in deploy.rb file  
	     1. server 'with your public ip of amazon_ec2_instance'  
	     2. repo_url 'with your repo url'  
	     3. application 'with your app name'   
	     
**Step 17.** Create file 'nginx.conf' in config folder(in your rails app) and paste the nginx.conf(of this repo) content to it.  
	  ** Replace(Press Ctrl+h) **capistrano-aws** with your app name in nginx.conf   

**Step 18.** Config and push code to github   
 		command:-   
 			**$ git remote add origin your_repo_url	 
 			$ git checkout origin master  
 			$ git add .  
 			$ git commit -m "initial commit"  
 			$ git push origin master**  

**Step 19.** Add local secret key base in aws_ec2 instance  
command:-  
## In local  
**$ rake secret**  
*Copy this key  

## In aws_ec2 Instance terminal  
**$ sudo nano /etc/environment**  
*bottom of PATH  write SECRET_KEY_BASE=your copied secret key(from local) **Don't delete anything from this file**  
press ctrl+x and save this file  

**Step 21.** Install nginx in aws_ec2_instance   
	  command:   
	  ## In aws_ec2 Instance terminal  
	  	 **$ sudo apt install nginx  
	  	 $ sudo rm /etc/nginx/sites-enabled/default**  

**Step 22.** Deploy app  
command:   
## In local  
 **$ cap production deploy:initial --trace**  
### If 'uninitialized constant Capstrano::Puma' error occurs just install the capistrano3-puma gem globally by running the following command:  
**$ gem install capistrano3-puma**  
	
**Step 23.** Nginx configuration  
  commands:   
  ## In aws_ec2_instance terminal    
 **$ sudo ln -nfs "/home/ubuntu/apps/capistrano-aws/current/config/nginx.conf" "/etc/nginx/sites-enabled/capistrano"  
 $ sudo service nginx restart**
	  	
# Congratulations you have successfully deployed your rails app  

Now just go to your aws_ec2 public address and you will see there your rails app  

Note:- You can check your nginx logs by the following command:  
		**$ vim apps/capistrano-aws/current/log/production.log**
