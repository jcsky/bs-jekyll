---
layout: post
title: "deploy on ubuntu (no rvm)"
description: ""
category:
tags: []
---
{% include JB/setup %}

# Deploy Rails on Ubuntu
原文<https://ihower.hackpad.com/Deploy-Rails-on-Ubuntu-j8zYjjMpHth#:h=設定-Deploy-使用者>

## 更新系統套件
sudo apt-get update
sudo apt-get upgrade -y
sudo dpkg-reconfigure tzdata
		（選Time zone=>Asia=>Taipei）

## 安裝編譯套件 
sudo apt-get install -y build-essential git-core bison openssl libreadline6-dev curl zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-0 libsqlite3-dev sqlite3  autoconf libc6-dev libpcre3-dev curl libcurl4-nss-dev libxml2-dev libxslt-dev imagemagick nodejs libffi-dev
		（沒錯，就是copy上面一整串去執行）

## 安裝 Ruby

### 1.下載源碼編譯
    wget http://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.2.tar.gz
    tar xvfz ruby-2.2.2.tar.gz
    cd ruby-2.2.2
    ./configure
    (在centos下安裝失敗時，改用 ./configure --prefix=/usr 指定路徑)
    make
    sudo make install


### 2.brightbox(比較快)

使用 https://www.brightbox.com/docs/ruby/ubuntu/ 已經編譯好的套件

		sudo apt-get install software-properties-common
		sudo apt-add-repository ppa:brightbox/ruby-ng
		sudo apt-get update
		sudo apt-get install ruby2.2 ruby2.2-dev ruby-switch
		 


## 安裝 MySQL
sudo apt-get install mysql-common mysql-client libmysqlclient-dev mysql-server
		過程中會出現視窗提示你輸入 root 密碼
sudo gem install mysql2 --no-ri --no-rdoc
$ mysql -u root -p
		（上面指令輸入後會進入 mysql 的 console ） 
CREATE DATABASE rails_exercise CHARACTER SET utf8;
		＊＊shopping_exercise請換成自己的專案名稱並且與下面 dababase.yml 內的database名稱一樣
		上面指令為create一個資料庫，你可以把rails_exercise改成你想要的資料庫名稱
		執行完，輸入 exit 離開 mysql console，然後繼續以下步驟
		若是之後用相同主機建立一個新專案，要再 create database

## 安裝 PostgreSQL
		要用 MySQL 的話，就不需要裝 PostgreSQL 了
sudo apt-get install postgresql libpq-dev postgresql-contrib
		上述在 Ubuntu 12.04 只能裝到 PostgreSQL 9.3，如果要裝 9.4 請參考 http://www.postgresql.org/download/linux/ubuntu/ 
		apt-get install postgresql-9.4 postgresql-client-9.4 libpq-dev
sudo gem install pg --no-ri --no-rdoc

修改帳號 postgres 的密碼 sudo -u postgres psql 然後打 \password 
建資料庫 sudo -u postgres createdb your_database_name

		papt-get install postgresql-9.4 postgresql-client-9.4 libpq-dev
		 

## 安裝 Passenger 和 Nginx
sudo gem install bundler passenger --no-ri --no-rdoc
sudo passenger-install-nginx-module
		安裝Nginx透過Passenger去安裝，目的是讓你可以跑ruby
		按enter繼續
		選 ruby
		接著會要你選1. download或2. customize=>請選 1
		用預設目錄 /opt/nginx

設定 Nginx 啟動腳本

wget -O init-deb.sh http://www.linode.com/docs/assets/1139-init-deb.sh
sudo mv init-deb.sh /etc/init.d/nginx
sudo chmod +x /etc/init.d/nginx
sudo /usr/sbin/update-rc.d -f nginx defaults 

 Nginx 用法

	•	sudo service nginx start 可以啟動看到 Welcome to nginx!
	•	sudo service nginx stop 
	•	sudo service nginx restart 

		瀏覽器打開 http://106.187.52.234應該可以看到 It works

## 設定 Deploy 使用者
		root帳號權限很大，我們不希望每個人都需要用到root權限，所以會開個deploy帳號
		會放在home/deploy
sudo adduser --disabled-password deploy
		讓deploy無法用密碼登入
sudo su deploy
		su就是切換身分
		sudo su root可以切換成root身分，要打密碼
ssh-keygen -t rsa
複製本機的 ~/.ssh/id_rsa.pub 到 /home/deploy/.ssh/authorized_keys
		注意上面這個步驟其實有一些額外的步驟要做
		1. Jimmy的的作法是新開一個console視窗，輸入cat ~/.ssh/id_rsa.pub，會出現一串文字，待會再來copy
		2. 在原來的ssh主機 console下輸入vi /home/deploy/.ssh/authorized_keys，進入vi去編輯該檔，把上一個步驟的視窗內的文字copy貼上到vi內，然後 :wq 離開

chmod 644 /home/deploy/.ssh/authorized_keys
chown deploy:deploy /home/deploy/.ssh/authorized_keys
		這樣本機就可以直接  ssh deploy@106.187.52.234 登入無須密碼

		Linode ssh雲端主機設定到此暫時完成，接下來是本地code部分去設定

## 設定 Capistrano 腳本

範例 code: <https://github.com/ihower/rails-exercise-ac5/blob/master/config/deploy.rb>

(本機) Gemfile 加入 
		gem 'capistrano-rails', :group => :development
		gem 'capistrano-passenger', :group => :development

		資料庫用 MySQL 記得加上 gem "mysql2" 

(本機) 輸入 bundle 和 cap install 
(本機) 編輯 Capfile 在中段加入  
		require 'capistrano/rails'
		require 'capistrano/passenger'
(本機) 編輯 config/deploy.rb 
	◦	開頭加上一行 \`ssh-add\` # 注意是鍵盤左上角的「 \`」不是單引號「 '」，need this to make key-forwarding work，參考 https://ihower.tw/blog/archives/7837
	◦	set :application, 'rails-exercise'
	◦	set :repo_url, 'git@github.com:ihower/rails-exercise.git'
	◦	set :deploy_to, '/home/deploy/rails-exercise'
	◦	set :keep_releases, 5
	◦	打開 set :linked_files, fetch(:linked_files, []).push('config/database.yml' 'config/secrets.yml')
		有 facebook.yml 或 email.yml 的話也要加進來
	◦	打開註解 set :linked_dirs, fetch(:linked_dirs, []).push('log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'vendor/bundle', 'public/system')
		paperclip的圖檔是存在public/system內，若沒加上此設定，每伺佈署新程式時，圖檔就會被清掉，所以要加上。
		另外一種上傳圖檔的gem carrierwave，圖檔是放在 'public/uploads'
	◦	加上 set :passenger_restart_with_touch, true
		在 current 專案目錄下 touch tmp/restart.txt 也會重開 rails
(本機) 編輯 config/deploy/production.rb  把 example.com 換成 server IP
	•	server '139.162.21.176', user: 'deploy', roles: %w{app db web}, my_property: :my_value
		上面範例是iHower的，要換成你自己的ssh主機位置
(本機) cap production deploy:check   過程中出現少了以下兩個檔案的錯誤，請補上後再試一次
		(遠端) 設定 shared/config/`x`database.yml
		(遠端) 設定 shared/config/secrets.yml 
		格式參考 config/secrets.yml，在本機用 rake secret 可以隨機產生一個新的 key。
		 
		production 用的 database.yml, 和 secrets.yml 資訊只存在於伺服器上，不要 commit 進 repo.
		上面的設定其實需要夠詳細的的步驟：
		ssh deploy@106.187.52.234（以你的ssh主機位置為主）
		cd social-exercise（進入你的專案目錄，以你的專案名稱為主）
		cd shared
		cd config
		打vim database.yml便會新增並且編輯，把database.yml的內容copy上去，直接copy會有一些字跑掉，要注意修改一下，ESC，:wq存檔離開
		secrets.yml同樣做一次
		若有其他 settings.yml 一樣要做，例如 email.yml、facebook.yml
		 
**database.yml 範例**

		production:
		  adapter: mysql2
		  encoding: utf8
		  database: your_project_name
		  host: localhost
		  username: root
		  password: your_password
  
**secrets.yml 範例 ( rake secret 產生一個新的 key)：**
  
		production:
		  secret_key_base: xxxxxxx........

(本機) cap production deploy  (之後部署就用這個指令，接下來設定 Nginx)

## 一些錯誤問題

cap production deploy 執行中間出現很多 (failed) 是怎麼回事? 例如：
		 [39417d64] Finished in 0.105 seconds with exit status 1 (failed).
		 .....
		[854031f0] Finished in 0.099 seconds with exit status 1 (failed).
		.....
		 [3f250525] Finished in 0.109 seconds with exit status 1 (failed).
		 ...
		 [6e3a177b] Finished in 0.110 seconds with exit status 0 (successful).
		 
不用擔心!! 這些是因為有些 CLI 指令的回傳值不是0 (照慣例非0代表有錯誤，不過有些 CLI 指令沒有照這個慣例)，所以 Capistrano 使用的 NetSSH 輸出成 (failed)。如果真的有問題，Capistrano 會丟出例外然後中斷執行，你就看不到最後的 successful 了。

錯誤: cap production deploy 碰到 deploy:assets:backup_manifest fails 錯誤: capistrano/rails#111
解法: 需要升級 bundle update capistrano-rails 到 1.1.3 版本

錯誤: 更新Capistrano後，佈署時出現
cap aborted!
Capfile locked at 3.3.3, but 3.4.0 is loaded
解法: 怎麼解？因為Capistrano鎖版本 把config/deploy.rb 
lock '3.3.3'   改成    lock '>=3.3.3'

錯誤: Running /usr/bin/env passenger-config restart-app 和 you are not authorized to query the status....
解法: capistrano-passenger gem 需要 0.1.1 版本，並修改 config/deploy.rb 加上
set :passenger_restart_with_touch, true

## 設定 Nginx 網頁伺服器

	•	編輯 /opt/nginx/conf/nginx.conf 

		worker_processes  auto;
		 
		events {
		    worker_connections  4096;
		    use epoll;
		}
		 
		http {
		      passenger_root /usr/local/lib/ruby/gems/2.2.0/gems/passenger-5.0.20; # 這行 passenger_root 請依照原本的路徑不要改
		      passenger_ruby /usr/local/bin/ruby; # 這行 passenger_ruby 請依照原本的路徑不要改
		 
		      include       mime.types;
		      default_type  application/octet-stream;
		    
		      include    /opt/nginx/conf/vhost/\*.conf;
		      client_max_body_size 100m;
		       
		      gzip                on;
		      gzip_disable        "msie6";
		      gzip_comp_level    5;
		      gzip_min_length    256;
		      gzip_proxied       any;
		      gzip_vary          on;
		      gzip_types 
		    application/atom+xml
		    application/javascript
		    application/x-javascript
		    application/json
		    application/rss+xml
		    application/vnd.ms-fontobject
		    application/x-font-ttf
		    application/x-web-app-manifest+json
		    application/xhtml+xml
		    application/xml
		    font/opentype
		    image/svg+xml
		    image/x-icon
		    text/css
		    text/xml
		    text/plain
		    text/javascript
		    text/x-component;
		    
		    sendfile        on;
		    tcp_nopush     on;
		    tcp_nodelay on;
		        
		    # .... 以下不用改  
    }
 
	•	新增 /opt/nginx/conf/vhost/your_project_name.conf，一個 rails 專案搭配一個設定檔案：
這個檔名自由命名，副檔名是 .conf 即可
		server {
		      listen 80;
		      server_name your_domain;
		      root /home/deploy/your_project_name/current/public; 
		      passenger_enabled on;
		      
		        passenger_min_instances 1;
		        server_tokens       off;
		        
		        location ~ ^/assets/ {
		         expires 1y;
		         add_header Cache-Control public;
		         add_header ETag "";
		         break;
		        }
		  }

	•	其中 server_name your_domain 會換成你的 domain。如果 domain name 還沒註冊好，可以先用 server ip。但是如果你的 server 上有多個 Rails 專案或網站，就必須用不同 domain 來區分。
	•	如果有多個domain連到同一個伺服器，可以用空白區隔，例如：
	◦	server_name dureading.calvinchu.cc dureading.com www.dureading.com
	◦	這樣三個domain都會連到同一個server

	•	重開 Nginx 的指令是  sudo service nginx restart
	◦	如果有改任何 nginx config 需要重開
 
		加映: 如何用 Nginx 設定一個靜態網站

	•	假設靜態網頁檔案位置放在 /home/deploy/www 目錄下，
	◦	放一個 index.html
	•	例如新增 /opt/nginx/conf/vhost/www.conf

		server {
		     listen 80;
		     server_name 139.162.27.30;
		 
		     location / {
		            root   /home/deploy/www;
		            index  index.html index.htm;
		        }
		 
		}
		listen 80; 指port 80
	•	如果同一台 server 有多個網站或 Rails，那 server_name 得換不一樣的 domain 才能區分。
 
		常用指令 

	•	要 deploy code，都是先 git push 到github上，再 cap production deploy
 
	•	如何 SSH 登入?
	◦	(本機) ssh root@your_server_ip
	▪	su deploy 指令可以切換身分到 deploy
	▪	如果修改 nginx 設定必須要 root 身分、操作你的 rails 專案則用 deploy 身分 
	◦	(本機) 直接用 deploy 登入 ssh deploy@your_server_ip
	•	如何看遠端 log ?
	◦	用 deploy 身分登入
	◦	cd ~/your_project/current
	◦	tail -n 500 log/production.log 這樣會顯示最後500行
	◦	或是 tail -f log/production.log 這樣會一直掛著顯示
	◦	nginx log 在 /opt/nginx/logs 下，要用 root 身分才能看
	•	在遠端如何進 rails console? 
	◦	用 deploy 身分登入，例如 ssh deploy@your_server_ip 或用 root 登入再切換身分 sudo su deploy
	◦	cd ~/your_project/current
	◦	bin/rails c production 或 bundle exec rails c production
	•	在遠端跑 rake? 例如
	◦	RAILS_ENV=production bin/rake db:seed
	•	如何重開遠端 rails 而不重開 nginx ?
	◦	在遠端 ~/your_project/current 下執行 touch tmp/restart.txt
	◦	完全重開 nginx 的話 sudo service nginx restart 
 
		主機安全加強 (optional)

http://blog.dharanasoft.com/2011/06/09/securing-your-ubuntu-vps/

	•	通常會另開一個你的個人使用者有 sudo 權限，可以用 public key 登入
	◦	先用 root 帳號登入，用 visudo 指令可以編輯誰有 sudo 權限
	◦	接著就可以設定 root 帳號不可以 SSH 登入
	•	(optional) 可以設定 SSH 不允許密碼登入，只能用 public key
	•	設定防火牆 iptable 只允許 80, 443, 22 port。直接操作底層的 iptable 步驟比較複雜，可以用 ufw 這個工具：
	◦	sudo apt-get install ufw
	◦	sudo ufw default deny
	◦	sudo ufw allow 22
	◦	sudo ufw allow 80
	◦	sudo ufw allow 443
	◦	sudo ufw enable
	◦	sudo ufw status
	◦	sudo ufw insert 1 deny from 要ban掉的IP
	◦	sudo ufw reload
	•	安裝 fail2ban，自動 ban 掉亂試密碼的 ip
	◦	https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-12-04

		使用 logrotate 定期整理 Rails Log 檔案

http://ihower.tw/blog/archives/3565

		設定 Nginx SSL 
	•	產生 CSR request 憑證簽章要求
	◦	openssl req -new -newkey rsa:2048 -nodes -keyout staging.key -out staging.csr
	▪	其中 Common Name 必須是你的 domain name，例如 exercise.ihower.tw。如果是 wildcard certificate 則用 *.ihower.tw
	▪	最後的 password 可以不填，填的話之後每次 server 啟動需要打密碼
	▪	這會產生兩個檔案 staging.csr 和 staging.key，後者是你的私鑰 
	•	將 .csr 申請憑證檔案丟給憑證機構做申請，會拿到 .crt key
	◦	過程會需要認證你真的擁有這個 domain，通常會要求你設定一個特別的 CNAME
	◦	也可以自己簽發，只是瀏覽器會警告
	▪	openssl x509 -in staging.csr -out staging.crt -req -signkey staging.key -days 365
	◦	買 COMODO cert 的話，請參考 https://support.comodo.com/index.php?/Default/Knowledgebase/Article/View/789/0/certificate-installation-nginx
	▪	會需要認證你擁有此網址，通常方法是 1. 設定某個 CNAME 或 2. 寄信給 admin@your_domain，認證成功後，就可以下載憑證 .crt 檔案
	▪	cat your_domain_name.crt COMODORSADomainValidationSecureServerCA.crt COMODORSAAddTrustCA.crt  >  ssl-bundle.crt
	▪	或 cat your_domain_name.crt your_domain_name.ca-bundle >  ssl-bundle.crt
	•	將憑證 .crt 和私鑰 .key 放到 /opt/nginx/ 下，新加 vhost 支援 SSL：

		 server {
		        listen       443 ssl;
		        ssl on;
		        ssl_certificate       /opt/nginx/staging.crt;
		        ssl_certificate_key    /opt/nginx/staging.key;
		        server_name exercise.ihower.tw;
		         
		        root /home/deploy/rails-exercise/current//public;
		        passenger_enabled on;
		        
		        passenger_min_instances 1;
		        server_tokens       off;
		 
		location ~ ^/assets/ {
		  expires 1y;
		  add_header Cache-Control public;
		  add_header ETag "";
		  break;
		}
		}

如果只支援 SSL 連線，可以將所有 HTTP 連線重導到 HTTPS

		server {
		  listen  80;
		  server_name exercise.ihower.tw;
		  rewrite ^(.*) https://exercise.ihower.tw$1 permanent;
		  server_tokens off;
		}

		https://www.ssllabs.com/ssltest/index.html 可以檢測是否安裝完美

		SPDY (HTTP/2)

需要重裝 nginx 加上 spdy 支援：

passenger-install-nginx-module --extra-configure-flags=" --with-http_spdy_module"

編輯 vhost 檔案，修改成  listen       443 ssl spdy;

		Memcached 

		如果多個 Rails apps 共用一台，Rails 端要加 namespace

sudo apt-get install memcached
		1.4.14
		Redis 

sudo apt-get install redis-server
		2.8.4

		ElasticSearch

sudo apt-get install openjdk-7-jre
		Download from http://www.elasticsearch.org/download
wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.1.deb
sudo dpkg -i elasticsearch-1.4.1.deb
sudo /etc/init.d/elasticsearch start

		這會安裝到 /usr/share/elasticsearch 和啟動檔 /etc/init.d/elasticsearch
		設定檔在 /etc/elasticsearch/elasticsearch.yml
 
測試

curl -X GET 'http://localhost:9200'
		 
應該會看到

{
  "status" : 200,
  "name" : "Vishanti",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "1.4.1",
    "build_hash" : "89d3241d670db65f994242c8e8383b169779e2d4",
    "build_timestamp" : "2014-11-26T15:49:29Z",
    "build_snapshot" : false,
    "lucene_version" : "4.10.2"
  },
  "tagline" : "You Know, for Search"
}

應該檔掉外部連線：

sudo vi /etc/elasticsearch/elasticsearch.yml
		 
		network.bind_host: localhost
		 
sudo service elasticsearch restart

進到你的 rails product 重新索引：

RAILS_ENV=production bundle exec rake environment elasticsearch:import:all
		 
		Sidekiq

		如果多個 Rails apps 共用一台，Rails 端要加 namespace

https://github.com/seuros/capistrano-sidekiq

Gemfile 加上 gem 'capistrano-sidekiq'
Capfile 加上 require 'capistrano/sidekiq'

		Backups

https://github.com/meskyanichi/backup

		Next
 
 基本 Monitor 指令：
 
		 sudo passenger-status
		 sudo passenger-memory-stats
 
	•	升級 VPS 增加更多 Ram：增加 passenger processes 設定、增加 mysql buffer size 設定、增加 memcached, redis 記憶體、增加 sidekiq workers 數量
	•	Horizontally scale: https://www.digitalocean.com/community/tutorials/5-common-server-setups-for-your-web-application
	◦	改用 AWS 方案：將資料庫包給 RDS，使用 VPC, EC2, EBS, RDS, ElastiCache, SQS, SNS 等

		如果有做使用者檔案上傳 !!!注意!!! 
	•	用 paperclip 使用者上傳檔案放在 /public/system 內，要設定config/deploy.rb

		set :linked_dirs, fetch(:linked_dirs, []).push('bin', 'log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'vendor/bundle', 'public/system', 'public/uploads')

	•	將public/uploads(carrierwave)或public/system(paperclip)加進去。若沒加進去，每次佈署新程式，前面上傳的檔案會沒有被連結到，造成使用者會連不到

		如何多加 staging 環境和部署?

	1.	專案新增 config/environments/staging.rb 環境設定，內容同 config/environments/production.rb
	2.	專案新增 config/deploy/staging.rb，內容同 config/deploy/production.rb，佈署 ip 看看是不是同一台伺服器
	3.	server 上新增 /opt/nginx/conf/vhost/staging.conf 設定檔，內容跟 production 用的 nginx 設定檔一樣，除了 需要加一行 rack_env staging;  因為預設是 rack_env prodcution;
	4.	如果 staging 和 production 同一台 server 的話，網址(domain name)和佈署目錄得要不一樣：
	a.	上述的 nginx vhost 設定，裡面 1. server_name 的網址 和 2. root 目錄位置要改
	b.	將專案本來 config/deploy.rb 裡面的 set :deploy_to, '/home/deploy/shopping' 設定搬到 config/deploy/staging.rb 和 config/deploy/production.rb 並且改成不同目錄
	5.	Push code to github
	6.	cap staging deploy:check
	7.	到 server 上新增 shared/config/database.yml 和 secrets.yml，注意 yml 第一層要改成 staging。如果有其他 yml 設定也需要一併設定(例如 facebook.yml 和 email.yml 裡面要多加 staging)
	8.	在 server 上新增 staging 用的資料庫
	a.	mysql -u root -p 
	b.	CREATE DATABASE shopping_exercise_staging CHARACTER SET utf8;
	9.	cap staging deploy
	10.	重開 nginx:  sudo service nginx restart

		如何匯出本機資料庫匯入到遠端伺服器

	•	在本機匯出資料庫:
	◦	mysqldump -u root your_db_name > your_db_name.sql
	•	壓縮這個檔案:  
	◦	gzip your_db_name.sql
	•	上傳到遠端伺服器 deploy 帳號的家目錄下:
	◦	scp your_db_name.sql.gz deploy@your_server_ip:~/
	•	登入遠端伺服器:
	◦	ssh deploy@your_server_ip
	•	解壓縮:
	◦	gunzip your_db_name.sql.gz
	•	砍掉現有的資料庫，新增一個空的資料庫 (或是不砍舊的資料庫，新增一個不一樣名字的空資料庫，修改 database.yml 換個資料庫名字，匯入完最後再重開 rails)
	◦	mysql -u root -p
	◦	DROP DATABASE your_db_name;
	◦	CREATE DATABASE your_db_name CHARACTER SET utf8;
	•	匯入資料庫:        
	◦	mysql -u root -p your_db_name < your_db_name.sql

PostgreSQL 指令請參考 https://ihower.tw/blog/archives/8152

		如何加上 HTTP Basic Authentication

用途: 簡單保護 staging 伺服器防止外人看到

編輯 /opt/nginx/conf/vhost/your_project.conf 加上

		auth_basic "Restricted";
		auth_basic_user_file /opt/nginx/conf/.htpasswd;
		 
編輯 /opt/nginx/conf/.htpasswd 

your_account:{PLAIN}your_password


		加映: 如何裝 PHP 和 Wordpress

	•	sudo apt-get install php5-fpm php5-mysql php5-curl php5-cli php-apc
	•	sudo apt-get install php5-gd
	•	新增一個資料庫給 wordpress
	•	下載 wordpress https://tw.wordpress.org/install/ 並編輯 wp-config.php 設定資料庫帳號密碼
	•	把 nginx 使用者改成 www-data，修改 nginx.conf 裡的 user www-data;
	•	將 wordpress 目錄更改使用者為 www-data
	•	新增以下的 vhost 設定

		server {
		        listen   80;
		     
		        root /home/deploy/your_wordpress;
		        index index.php index.html index.htm;
		 
		        server_name wp.ihower.tw;
		        
		        error_page 404 /404.html;
		        error_page 500 502 503 504 /50x.html;
		        location = /50x.html {
		              root /usr/share/nginx/www;
		        }
		        
		       location / {
		        try_files $uri $uri/ /index.php?$args;
		       }
		        
		        # pass the PHP scripts to FastCGI server listening on /var/run/php5-fpm.sock
		        location ~ \.php$ {
		                try_files $uri =404;
		                fastcgi_pass unix:/var/run/php5-fpm.sock;
		                fastcgi_index index.php;
		                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		                include fastcgi_params;
		        }
		}


	•	最後啟動 PHP
	◦	sudo service php5-fpm restart

		加映: 如何裝 VPN Server (網咖和科學上網必備)

	•	sudo apt-get install pptpd
	•	編輯 /etc/pptpd.conf 打開以下設定
		localip 192.168.0.1
		remoteip 192.168.0.234-238,192.168.0.245
	•	編輯 sudo vi /etc/ppp/pptpd-options 打開 ms-dns 8.8.8.8
	•	編輯 vi /etc/ppp/chap-secrets 有哪些帳號密碼

		ihower pptpd yourpassword *

	•	sudo service pptpd restart
	•	iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
	•	iptables-save > /etc/iptables-rules
	•	編輯 vi /etc/network/interfaces 加上 
		pre-up iptables-restore < /etc/iptables-rules
	•	編輯 vi /etc/sysctl.conf 打開 net.ipv4.ip_forward=1

		其他
	•	修改本地的 sudo vi /etc/hosts 可以直接作 ip 和 domain name 的對應，不需要外部 DNS 
	◦	測試 web server 的時候很好用，不需要等 DNS 生效
	•	鳥哥vim編輯器說明  http://linux.vbird.org/linux_basic/0310vi.php#vi_ex
	•	網站架構演進 https://www.digitalocean.com/community/tutorials/5-common-server-setups-for-your-web-application
