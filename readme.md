# Deploying Flask App on EC2 machine

Get into EC2 instance via ssh and then follow these steps:

1. Pre-requisites
2. Gunicorn Setup(WSGI http server)
3. Configure Nginx as reverse proxy
4. Run Website


## 1. Pre-requisites
```bash
# Update and install python venv
$ sudo apt update
$ sudo apt-get install mysql-client python3-venv

# Clone github repo for this project
$ cd ~
$ git clone 
$ cd Flask-Blog-App

# create virtual environment and activate it
$ python3 -m venv venv
$ source venv/bin/activate

# install Flask and other packages
$ pip install flask flask-sqlalchemy pymysql gunicorn

```

## 2. Gunicorn Setup

In the Project Directory, test whether gunicorn works correctly.
```bash
(venv) $ gunicorn --bind 0.0.0.0:5000 wsgi:app 
```

Now create a custom service, for running app through gunicorn.
```bash 
$ sudo vim /etc/systemd/system/flask.service
```

Write following configuration in it,
```
[Unit]
Description=Gunicorn instance for Flask Blog App
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/Flask-Blog-App
Environment="PATH=/home/ubuntu/Flask-Blog-App/venv/bin"
ExecStart=/home/ubuntu/Flask-Blog-App/venv/bin/gunicorn --bind 0.0.0.0:5000 wsgi:app
Restart=always

[Install]
WantedBy=multi-user.target
```

Start and enable newly created `flask` service
```bash
$ systemctl daemon-reload

# start service
$ systemctl start flask          

# enable service on startup
$ systemctl enable flask         

# check service status
$ systemctl status flask         
```


## 3. Configure Nginx as reverse proxy
```bash
# install nginx
$ sudo apt install nginx        
```

Delete default nginx config files.
```bash
# delete default nginx configurations
$ sudo rm -rf /etc/nginx/sites-available/default
$ sudo rm -rf /etc/nginx/sites-enabled/default
```

Then, write new nginx config file for our app.
```bash
$ sudo vim /etc/nginx/conf.d/flask.conf
```

Write following configurations in it,
```
server {
    listen 80;
    
    location / {
        include proxy_params;
        proxy_pass http://127.0.0.1:5000;
    }
}
```


Perform test and restart nginx.
```bash
# Test new config file
$ sudo nginx -t

# Restart nginx
$ sudo systemctl restart nginx
```


Firewall configuration (Additional) 
```bash
$ sudo ufw delete allow 5000
$ sudo ufw allow "Nginx Full"
$ sudo ufw status
```

Reading error logs for nginx (Additional)
```bash
$ sudo tail /var/log/nginx/errors.log 
```


## 4. Run Website
Enter the public IP of EC2 machine in your browser. And the App will be working via nginx server.


Lastly, you can also follow this 
[Digital Ocean Guide](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-22-04). And Good luck!

