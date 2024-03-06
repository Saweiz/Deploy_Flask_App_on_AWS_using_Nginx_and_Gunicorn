# Deploying Flask App on EC2 machine using Gunicorn and Nginx

### App description

This is `Flask` based Blog app with `MySQL` (ORM) at backend. It has four sections:

1. Posts Page:
   Global users who visit website can see the posts uploaded by admin. Pagination is implemented at backend.

2. Contact Page
   If they want to contact admin, they can fill the contact form which asks there name, contact no., email, and any message they want to deliver.

3. About Page
   Here admin can write introductory information of his Blog website, his purpose, and any information he wants to deliver to his readers.

4. Dashboard (For Admin only)
   Admin (website owner) has to login via credentials. And then he can create, delete, and update posts. 


### Quick Start (Running the app locally)

Clone the repo and enter the project directory
```bash
git clone https://github.com/Saweiz/Flask-Blog-App
cd Flask-Blog-App
```

Create virtual environment and activate it
```bash
python3 -m venv venv
source venv/bin/activate
```

Install necessary python modules within virtual environment
```bash
pip install flask flask-sqlalchemy pymysql

```

## Deploying App on EC2 using Gunicorn and Nginx
Get into EC2 instance via ssh and then follow these steps:

1. Pre-requisites
2. Gunicorn Setup (WSGI http server)
3. Configure Nginx as reverse proxy
4. Run Website


## 1. Pre-requisites
Update and install python venv.
```bash
sudo apt update
sudo apt-get install mysql-client python3-venv
```

Clone github repo for this project.
```bash
cd ~
git clone https://github.com/Saweiz/Flask-Blog-App
cd Flask-Blog-App
```

Create virtual environment and activate it.
```bash
python3 -m venv venv
source venv/bin/activate
```

Install Flask and other python packages.
```bash
pip install flask flask-sqlalchemy pymysql gunicorn

```

## 2. Gunicorn Setup

In the Project Directory, test whether gunicorn works correctly. And run this command with virtual environment activated, after this step you can deactivate it.
```bash
gunicorn --bind 0.0.0.0:5000 wsgi:app 
```

Now create a custom service, for running app through gunicorn.
```bash 
sudo vim /etc/systemd/system/flask.service
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

Reload unit files, and start + enable newly created `flask` service

```bash
systemctl daemon-reload
```
```bash
systemctl start flask          
systemctl enable flask         
```

Check service status
```bash
systemctl status flask         
```


## 3. Configure Nginx as reverse proxy
Install Nginx.
```bash
sudo apt install nginx        
```

Delete default nginx config files.
```bash
sudo rm -rf /etc/nginx/sites-available/default
sudo rm -rf /etc/nginx/sites-enabled/default
```

Then, write new nginx config file for our app.
```bash
sudo vim /etc/nginx/conf.d/flask.conf
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
sudo nginx -t
```
```bash
sudo systemctl restart nginx
```


Firewall configuration (Additional) 
```bash
sudo ufw delete allow 5000
sudo ufw allow "Nginx Full"
sudo ufw status
```

Reading error logs for nginx (Additional)
```bash
sudo tail /var/log/nginx/errors.log 
```


## 4. Run Website
Enter the public IP of EC2 machine in your browser. And the App will be working via nginx server.

If you want further detail, you can also see this 
[Digital Ocean Guide](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-22-04) on this topic. 

Good luck! have a nice day. 

