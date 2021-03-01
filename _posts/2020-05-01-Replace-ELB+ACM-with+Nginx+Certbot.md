My blog is hosting on AWS in the structure of ELB (with ACM) + EC2 as app server + RDS as database (detail implementation in previous blog). Now I want to replace ELC and ACM with Nginx and Certbot.

Nginx is a web server which can be used as a reverse proxy, load balancer, mail proxy and HTTP cache. Here I use it as a reverse proxy. Since I only have one app server so load balancing is unnecessary.

Let’s Encrypt is a free certificate authority (CA) runs by Internet Security Research Group (ISRG). The process to manually create, apply and maintain certs can be miserable. Luckily Certbot offers free tool to handle these automatically.

By the way, use AWS public CA is free, just we cannot export the cert and can only use the cert within AWS services via AWS Certificate Manager (ACM).

## Part 1 - Nginx
#### Installation
I launch a new ubuntu server to host Nginx in AWS in the same PVC and same security group as my ELB (permission for accessing app server had been configured in previous blog).

The first step is to install Nginx. Just few commands following the official guide http://nginx.org/en/linux_packages.html.

After installation, set and run nginx:
```
sudo systemctl enable nginx
sudo systemctl start nginx
```

And confirm everything is alright:
```
sudo systemctl status nginx
```

#### Configuration
Some default paths for reference:
* Main config file: /etc/nginx/nginx.conf
* Other config files: /etc/nginx/conf.d/
* Access log: /var/log/nginx/access.log
* Static files: /usr/share/nginx/html

I create a customised blog.conf and include it into main config file. A full config file sample is available here https://www.nginx.com/resources/wiki/start/topics/examples/full/. Server block is the key part to modify, multiple server blocks and multiple locations in server block can be added.
```
server {
    listen      <port, by default 80>;
    server_name <domain>;

    location / {
        proxy_pass http://<app server IP>:8080;
    }
}
```

## Part 2 - Certbot
The Certbot guide is pretty clear https://certbot.eff.org/lets-encrypt/ubuntufocal-nginx. Just follow the guide step by step and Certbot automatically helps to issue certs and config Nginx. 

How Certbot verify our domain? A random generated strings is attached to certain location in Nginx config, Cerbot server tries access the location for verification. Once complete, these sections are auto deleted.


All Nginx configurations modified by Certbot have append # managed by Certbot.

Also forward requests from http 80 port to https 443 port.

Finally the blog is equipped with new cert. The cert is valid for 3 months and will be renew automatically by Certbot.
![let's cert](https://acloudgurulab-2019-yc.s3.amazonaws.com/letsencryptcert.png)

## Part 3 - AWS Route 53
Change the record set of my blog domain to map to Nginx server.

### Conclusion
![flowchart](https://acloudgurulab-2019-yc.s3.amazonaws.com/nginxflowchart.png)

[End]
