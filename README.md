<h1  align="center">
<br>
<a  href="https://jenkins.io/"><img  src="https://wiki.jenkins.io/download/attachments/2916393/logo.png?version=1&modificationDate=1302753947000&api=v2"  alt="Jenkins"  width="150"></a>
<br>
Jenkins Reverse Proxy
<br>
</h1>

<h4  align="center">A reverse proxy setup for Nginx to run Jenkins behind a subdomain with SSL on Ubuntu 18.04</h4>

## Setup
This guide assumes that you have Jenkins installed and configured to run on 
http://<span></span>example.<span></span>com:8080.

If you do not yet have Jenkins installed, you can follow this link to [install Jenkins on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-ubuntu-18-04)

## Configure Jenkins

From your Jenkins dashboard, navigate to  **Manage Jenkins > Configure System**.  Here you will find the **Jenkins URL** field. Be sure to set this to your new subdomain,, ie. https:<span></span>//jenkins.<span></span>example.<span></span>com/

## Nginx Config

We need to create the nginx server block for the subdomain:

 ```bash
 # Create the server block
$ sudo nano /etc/nginx/sites-available/jenkins.example.com
```

Add the following to the file, replacing the server_name and ssl certificate keys where applicable:

```javascript
upstream  jenkins {
	server  127.0.0.1:8080  fail_timeout=0;
}

server {
	listen  80;
	
	server_name  jenkins.example.com;
	
	return  301 https://$host$request_uri;
}

server {
	listen  443  ssl;
	
	server_name  jenkins.example.com;

	ssl_certificate  /etc/nginx/ssl/server.crt;
	ssl_certificate_key  /etc/nginx/ssl/server.key;

	location  / {
		proxy_set_header  X-Real-IP  $remote_addr;
		proxy_set_header  Host $host:$server_port;
		proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
		proxy_set_header  X-Forwarded-Proto  $scheme;
		proxy_pass http://jenkins;
		proxy_redirect http:// https://;

		proxy_http_version  1.1;
		proxy_request_buffering  off;
		proxy_buffering  off;

		add_header  'X-SSH-Endpoint'  'jenkins.example.com:50022'  always;
	}
}
```

Enable the Server Block and restart Nginx:

```bash
# create the server block symlink
$ sudo ln -s /etc/nginx/sites-available/jenkins.example.com /etc/nginx/sites-enabled/

# test for nginx config errors
$ sudo nginx -t

# restart nginx
$ sudo systemctl restart nginx
```

## Prevent port 8080 access

By default Jenkins runs on port 8080 of your domain. You can prevent access to Jenkins via http://<span></span>example.<span></span>com:8080 by disabling the port in the firewall:

```bash
# Block access to port 8080 in the firewall
$ sudo ufw deny 8080

# Check that the port was successfully blocked
$ sudo ufw status
```

