I have just completed the development of my personal blog and want to bring it online so that my parents can visit (as an evidence to convince them that I'm really studying XDD).

I pick up AWS and decide to deploy it step by step instead of using auto-deployment tools like beanstalk or heroku as I want a chance to review the AWS topic and understand more about the deployment process.

#### Step 1 - Launch EC2
I launched a EC2 instance Amazon Linux 2 AMI (centos) - t2.micro, with 1 vCPU and 1G memoryIt is free tier and enough to handle my tiny blog since I'm not expecting large amount of visitors. Edit the ec2 security group to allow 0.0.0.0/0 all inbound traffic (we will further modify this later after create ELB).

Then SSH to the EC2

~~~~
[root@ip-xx]$ ssh -i /path/<key-pair>.pem ec2-user@<ec2 public dns>
~~~~


Retrieve and install Tomcat

~~~
[root@ip-xx]$ wget https://apache.website-solution.net/tomcat/tomcat-9/v9.0.34/bin/apache-tomcat-9.0.34.tar.gz
[root@ip-xx]$ tar -zvxf apache-tomcat-9.0.34.tar.gz
~~~

Navigate to the folder bin under tomcat folder, run the start up script. If everything goes right we should be able to see the default tomcat page when access http://<ec2 ip>:8080 in browser. Port 8080 is the default port of tomcat and it also can be customised.

#### Step 2 - Launch RDS
I choose MySQL free tier and put it into the same VPC as the ec2 instance which hosts our web app.

Edit the inbound rule to allow access from ec2. Here we use default port 3306 and attach ec2's security group.

| Type         | Protocol | Port | Source         |
|--------------|----------|------|----------------|
| MYSQL/Aurora | TCP      | 3306 | web-app-ec2-sg |

*[rds security group]*

Don't forget to create necessary tables for your web app after create the RDS instance.

#### Step 3 - Upload java web app

Now wrap up the java war. My blog is a maven project. When I tried to wrap up I encountered with some keystore problems when trying to download maven-surefire-plugin during test process in packaging. Unfortunately I cannot solve it so I just skip the test for packaging.

~~~~
mvn clean package -Dmaven.test.skip=true
~~~~

Upload the war to EC2.

~~~~
scp -i /path/<key-pair>.pem /path/<webapp>.war ec2-user@<ec2 public dns>:~/
~~~~

Put the war under part tomcat/webapps. I rename the war to ROOT.war and remove the original ROOT folder under webapps to make my blog the default web.

Then restart tomcat - BOOM~~~ we now can visit the blog via http://<ec2 public IP>:8080.

#### Step 4 - Domain and SSL cert
I don't want visitors to access our blog via raw IP, besides I want to enable https to secure the access.

First I buy a domain (AWS provides the service, but I myself brought my domain from namecheap. there are also other providers to buy domain).

Then back to AWS Rout53, create a hosted zone, bring the domain we just brought to rout53 (also need to input the aws NS to namecheap). Setup CNAME record for our domain in rout53. Note things may mess up if we use CNAME record for root domain, so instead we use A record (alias) for root domain in AWS.

The last step is to get our domain certified in AWS Certificate Manager. It's pretty straightforward and do make our life easier comparing with all the steps we need to walk through if we are creating our cert with 3rd party e.g ssls.com.

Once our cert is ready in ACM, we can directly import it to ELB.

#### Step 5 - Link ELB to EC2
To make the access to the blog more secure and more stable, I want all visiting requests flow to ELB and let ELB direct the traffics to my web app EC2.

So, go create the ELB. After creation, add the SSL cert we created in ACM in previous step to ELB https port 443, and set http port 80 to redirect to our https path.

| Listener ID | Security policy | SSL Certificate | Rules                                        |
|-------------|-----------------|-----------------|----------------------------------------------|
| HTTP : 80   | N/A             | N/A             | Default: redirecting to HTTPS://#{host}:443/ |
| HTTPS : 443 | elb-sp          | <cert from acm> | Default: forwarding to <webapp>              |

*[elb listener ]*

Now we can further modify the ec2 security group, to allow inbound from ELB and SSH only.

| Type       | Protocol | Port | Source           |
|------------|----------|------|------------------|
| Custom TCP | TCP      | 8080 | load-balancer-sg |
| SSH        | TCP      | 22   | 0.0.0.0/0        |

*[ec2 security group]*

And we use S3 to store images if any in our blog.

The final chart is:

![flowchart](https://acloudgurulab-2019-yc.s3.amazonaws.com/flowchart.png)

Done!
