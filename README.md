# Learn AWS in a month of lunches - David Clinton
- [bostonPHP message board](https://www.meetup.com/bostonphp/messages/boards/forum/30317386)
- [boston DevOps slack](https://bostondevops-invites.herokuapp.com/)

# Book summary Part 1
## Chapter 1 AWS account
AWS – cloud computing provider, pay as you use for computing resources

### Sign up https://aws.amazon.com
Create a Free Account, Personal, Add payment method, verify your phone number, support plan: basic
Free Tier permits 750 hours of a t2.micro, for 1 year
Diagrams of VPC: https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html

Level of Cloud service:
SaaS
PaaS/FaaS
CaaS
IaaS

Billing alerts: Account top right, Preferences, checked Receive Free Tier Usage Alerts, Receive Billing Alerts, Save Preferences

Regions(VPC) > Availability Zone(subnet) > EC2 host/instance
EC2 instance = Virtual Machine
EC2 Amazon Machine Image ( AMI ) = a template for replicating precise OS environments (chapters 2 and 7).
EBS volumes = data volumes (like hard drives) for an instance (chapter 2).
security group = controls the movement of data between your AWS resources and the big, bad internet beyond (chap-ter 2).
Simple Storage Service ( S3 ) bucket = object storage for users
Virtual private cloud (VPC) = AWS’s primary organizational grouping of virtual resources


## Chapter 2 install webserver onto EC2 instance
easier to track cost and sut down services if you use only one region
VPC = region scope
Subnet = AZ scope
### create EC2 instance using ubuntu AMI
configure security group: ports 22, 80
	name: my ec2-default

ssh into instance with .pem key file

$ sudo chmod 400 ec2.pem
$ ssh -i ec2.pem ubuntu@ec2-23-22-122-111.compute-1.amazonaws.com 

inside VM:
sudo apt update
sudo apt install lamp-server^
cd /var/www/html
sudo mv index.html index.html.backup
sudo nano index.html
<h1>Welcome!</h1>
We hope you <i>really</i> enjoy your stay here.
echo "<h1>Welcome</h1>We hope you <i>really</i> enjoy your stay here." > index.html

visit the public dns address
stop instance
create image
terminate instance

### Under the hood:
 Install the OpenSSH Server package— apt install openssh-server
 Start the OpenSSH server— systemctl start ssh
    generate keypair = ssh-keygen -t rsa
    extract the public key = ssh-keygen -y -f key.pem > key.pub
    then add public key to ~/.ssh/authorized_keys chmod 600

## Chapter 3 Wordpress instance
EC2 instance dashboard > Monitoring tab

EC2 instance type families
http: //calculator.s3.amazonaws.com/index.html
Make sure everything in the same region to monitor cost
spot instance = on-demand instance

inside the instance:
mysql -u root -p
CREATE DATABASE wordpressdb; CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'mypassword'; GRANT ALL PRIVILEGES ON wordpressdb.* To 'wpuser'@'localhost'; FLUSH PRIVILEGES;
exit
wget https: //wordpress.org/latest.tar.gz
tar xzf latest.tar.gz
cd wordpress
cp wp-config-sample.php wp-config.php
nano wp-config.php

/** The name of the database for WordPress */
define('DB_NAME', 'wordpressdb');
/** MySQL database username */
define('DB_USER', 'wpuser');
/** MySQL database password */
define('DB_PASSWORD', 'mypassword');

replace salt https: //api.wordpress.org/secret-key/1.1/salt
sudo cp -r * /var/www/html

publicIP/wp-admin/install.php

troubleshooting:
your PHP installation appears to be missing the MySQL extension which is required by WordPress.
This probably means that the legacy PHP MySQL module isn’t working.
You can either enable MySQLi using
$ sudo phpenmod mysqli
or remove the old mysql-common package and install mysql-server on
top:
$ sudo apt remove mysql-common
$ sudo apt install mysql-server
Either way, you should restart Apache, just to make sure it’s up to date
with all the latest changes:
sudo systemctl restart apache2

## Chapter 4 RDS
off-instance relational database for wordpress
http: //calculator.s3.amazonaws.com/index.html
Migrate current DB to RDS
1. create database dump
mysqldump -u wpuser -p wordpressdb > /home/ubuntu/mybackup.sql

AWS Database Migration Service
https: //aws.amazon.com/dms/
environment: test,
Multi-AZ: no,
Publicly Accessible to No
Create New Security Group, my-rds note the ID of the security group
EC2 dashboard to edit your security group my-rds to allow traffic between your EC2 and RDS instances.
  Inbound, source: custom/sg-used by wordpress myec2-default
  copy database endpoint

on VM:
cd /home/ubuntu
mysql -u wpuser -p --database=wordpressdb \
--host=wpdatabase.co7swtzbtfg6.us-east-1.rds.amazonaws.com \
< mybackup.sql
sudo nano /var/www/html/wp-config.php
  DB_HOST value to equal your RDS endpoint. In
my case, it’s wpdatabase.co7swtzbtfg6.us-east-1.rds.amazonaws.com

## Chapter 5 DNS
Route 53, Domain Registration, Get Started Now
Hosted Zones, Create Hosted Zone, Type: Public Hosted Zone
	$0.50 per month
Record set—A set of data records that defines a particular aspect of a domain’s behavior.
Default: SOA, and NS
Create Record Set, Name empty, Type A record, enter public IP in the Value box
Click Create Record Set again, enter www in the Name field, Alias radio button, Alias Target, Create.

EC2 dashboard, click the Elastic IP s link in the left panel, and then click Allocate New Address, 
click your IP , click Actions, click Associate Address, click once in the Instance field, click Associate

health check:
add a test.html file to the /var/www/html
Create Health Check, click the Domain name radio button to search by
domain name rather than IP address, select New SNS Topic; add a topic name and at least one email address to which alerts will be sent. Click Create Health Check

Traffic Policies, policy Name, A: IP Address value, Connect To box, Choose Failover
Primary, click New Endpoint, IP, Secondary, IP address, Create Traffic Policy
Create Policy Records, choose your hosted zone, Create Policy Records

Hosted zone—AWS’s domain traffic management definition profile
Routing policy—The way you’ve configured incoming traffic redirection
Record type—Kinds of DNS record, including A, AAAA, CNAME, and MX
  A record = route requests for a particular domain like stuff.com to a specified IPv4 address
  CNAME record, canonical name = redirection

## Chapter 6 S3 bucket
S3, create bucket
Objects tab, Upload, Add File
Standard $0.025/month per GB vs Standard-IA(infrequent access, $0.0125/month per GB)
Edit Post page of the WordPress admin interface, click the spot on the page where you want to insert the video, and paste the URL link of an S3 -hosted video (see figure 6.6)

Static Site on S3
less than $0.03/month per GB for storage—plus as much as $0.090/ GB ofdata transferred out

## Chapter 7 S3 for backups
Click Volumes in the EC2 dashboard, and note the Volume ID
Actions > Instance State > Stop
Snapshots > Create Snapshot
rename the volume WordPressInstanceBackup001
restart the instance
From the Snapshots page, click Actions, Create Image, name the image wp-clone1
click the Virtualization Type drop-down, select Hardware-Assisted Virtualization, click Create
select the image, click Actions and Launch

manual backup and restore
on EC2:
cd ~
tar czf mybackup.tar.gz /etc /var /home
tar ztf mybackup.tar.gz
$ sudo apt update
$ sudo apt install unzip
$ curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
$ unzip awscli-bundle.zip
$ sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
$ aws configure
credential: AWS Console, your account name in upper right, select Access Keys and then Create New Access Keys
us-east-1
aws s3 cp mybackup.tar.gz s3: //YourS3BucketName

EBS—AWS’s Elastic Block Store service.
Volume—A virtual hard drive managed by and made available through EBS.
Snapshot—A copy of an AWS EBS volume captured in its exact state at the
moment the copy was made.

## Chapter 8 IAM users, groups, and roles
policy = defines who can do what actions on which resources
Creating an admin user:
AWS Console, Identity and Access Management, Users, Add User, Steve, enable programmatic access
Permissions tab, Add Permissions, Attach Existing Policies Directly, AdministratorAccess
Use account-specific sign-in URL to sign in
IAM page to go to
Users. Then click Create New Group. Enter designteam for the Group
Name in the next window, and then click Next Step to go to the Attach
Policy page: AmazonS3FullAccess
Add Users ann, tony to designteam
Role—An IAM identity that users can adopt to assume its access permissions

## Chapter 9 Cost
https: //calculator.s3.amazonaws.com/index.html
  Sample on the right, click details
Total Cost of Owner-ship ( TCO ) Calculator https: //awstcocalculator.com

## Chapter 10 Resource tags
Select a security group, and
click the Tags tab in the bottom panel. Then, click the Add/Edit Tags
button. Click Create Tag, and choose a name
Resource Groups drop-down, Tag Editor
account, project, p-owner, environment
Resource Groups, Create a Resource Group
Resource group—A tag-based collection of links to AWS resources within a
single account

## Chapter 11 CloudWatch
AWS Budgets: your account name at
upper-right, My Billing Dashboard, Budgets link
Use root account:
Select Cost, Set the budget period, Set the start and end dates, Set Budgeted Amount
Include cost related to: service, tag
notification email
CloudWatch:
AWS Console, click CloudWatch in the Management
Tools section and then click the Billing link, Create Alarm
1. Select Metric, Billing, Total estimated charge, Next
2. Define Alarm, Name, EstimatedCharges > $10
cloud watch alert on VolumeIdleTime or CPU utilization
Simple Notification Service (SNS)—Amazon’s way of communicating between participants (both human and machine) to coordinate actions

## Chapter 12 CLI
sudo apt install awscli
or
sudo pip install awscli
sudo apt update && sudo apt install python
unzip curl ca-certificates
curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
unzip awscli-bundle.zip
sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
aws configure
aws ec2 describe-security-groups
aws configure --profile test-account
aws s3 cp myfile s3: //mystuff1019 --profile test-account
aws iam add-user-to-group help
aws iam add-user-to-group --user-name Bob --group-name Admins
aws s3 ls
aws ec2 run-instances --image-id ami-YOUR \
--count 1 --instance-type t2.micro \
--key-name plural --security-group-ids sg-YOUR

# Part 2
## Chapter 13 application optimization
Elasticity—The ability to increase and decrease available compute resources
to meet changing demands
Scalability—The ability of software or infrastructure to adapt to changes in
service volume
Scaling out—The addition of new server nodes to handle increased demand
(horizontal scaling)
Scaling up—The adoption of a more powerful server node to handle increased
demand (vertical scaling)

## Chapter 14 Virtual Private Cloud
route table = what traffic to through which device, typically external traffic goes through the igw
network access control list ( ACL ) = inbound/outbound rules that control what kinds of network traffic is allowed both into and out of the VPC
security group is primary line of defense
AWS Console, in the Networking and Content Delivery, VPC, Create
VPC, CIDR block of 192.168.1.0/24
Start VPC Wizard, VPC with Public and Private Subnets

Deploying a website across two availability zones:
EC2 dash-board, click Launch Instance
Configure Instance Details
Click the Subnet drop-down, click the Create New Subnet link
launch identical copies of a EC2 instance into separate subnets, and test to confirm that they’re both available.

## Chapter 15 Load balancing
 launch four EC2 instances into two separate availability zones
$0.025/hour to run a balancer—which is $18 for a month—and $0.008 per GB of transferred data. 100 GB of total monthly transfers would come to $0.80.
1. Create a target group, and configure a health check.
  - EC2 dashboard, Launch Instances, My AMIs tab at left, Select WordPress AMI, Instance Details, Number of Instances: 2, us-east-1a subnet, create a security group name **cluster-instances**, opens HTTP (port 80) access to any incoming traffic
  - subnet us-east-1a: instance az-1a-inst01, az-1a-inst02
  - subnet us-east-1b: instance az-1b-inst01, az-1b-inst02
1. Register your four instances with the target group.
  - Target Groups, Targets tab, Edit, select all four, and click Add to Registered, Click Save, pic 207
1. Create a load balancer, and associate it with the two subnets hosting your instances.
  - Load Balancers, Create Load Balancer, type Application LB, name wp-lb-prod
  - Availability Zones configuration: add both subnets to Selected subnets
1. Create a security group for both instances and for the load balancer.
  - Configure Security Settings, same security group as the instance
1. Associate your target group with the load balancer.
  - Target Group drop-down menu, select Existing Target Group **cluster-instances**
test LB's public DNS name
SSH into any one of the instances, and delete (or rename) the index.html file

Load balancing = route traffic load
Health check = Periodic attempt to load a resource on a remote server to con- firm that the server is running
Target group = AWS framework containing configuration information and health-check settings for registered instances

## Chapter 16 Auto scaling
Auto scaling = add or remove capacity
launch configuration = indicate the number of instances based on an AMI in a target group
Launch Configuration, Create Launch Configuration, name: cluster-config, wp-clone1 AMI, instance type t2.micro, IAM role, Enable CloudWatch

On-Demand = most expensive
Reserved = cheapest
spot = in between, often used for auto-scaling

User data, as text:
#!/bin/bash
systemctl start apache2
echo "Welcome to my webserver" > /var/www/html/index.html

Security Group page, click the Select an Existing Security Group radio button, and use the same cluster-instances group
Create Auto Scaling Group page, name: cluster-AS-group,
2 instances, two subnets: the same us-east-1a and us-east-1b
Advanced Details section of the Create Auto Scaling Group page, and select the Load Balancing check box, Target Groups box: cluster-target, Next: Configure Scaling Policies, Use Scaling Policies to Adjust the Capacity of This Group
Increase Group size, Add new alarm: when CPU >= 90 Percent for atleast 5 minutes
Decrease Group Size dialog, CPU utilization drops below 20% of capacity for 5 minutes

one of the instances on the EC2 Instances page, and reload the balancer’s DNS address in the browser, should see a new instance being launched

## Chapter 17 CDN, content-delivery networks
Cache = A data store from which data can be quickly retrieved
Edge locations = CloudFront servers positioned around the world to be as close as possible to end users
CloudFront origin servers:
- Static content stored in an S3 bucket
- Dynamically generated content from a web server
### Networking and Content Delivery, CloudFront, Create Distribution, Get Started(Web)
Origin Domain Name, select the ELB
The Free Tier allows you 50 GB of data transfer to the internet and 2 million HTTP and HTTPS requests each month.
around $5.00 for 100GB out
Request of import certificate, Domain name: *.YOUR.com, Click Review and Request
Default Root Object tells CloudFront to load a specific file on your origin server when viewers arrive at your root URL, set Distribution State to Enabled

# Part 3 Other considerations
## Chapter 18 hybrid infrastructure
Glacier = archive data
- S3, enable versioning, Lifecycle rule, Configure transition, current/previous versions, Transition to Amazon Glacier after 60 days
Storage Gateway = simulate local storage, can cache a local copy and upload another another copy
Snowball = ship physical drives to Amazon
AWS Direct Connect = dedicated network connection between your office/data center and your AWS-based resources
virtual private gateway service = to provide VPN tunnel connection to AWS VPC
AWS Directory Service = integration between any existing local Microsoft Active Directory (or AD -compatible) system and AWS resources
Disaster Recovery:
- pilot-light model = hot site mirror as failover if primary site failed
- warm-standby model = smaller scale than the hot site
EC2 Systems Manager = can manage on premise servers after installing SSM agent
Hybrid solution = A combination of cloud-based and on-premises infrastructure.
Migration = Shifting IT infrastructure between local and cloud environments.

## Chapter 19 Cloud automation
AWS Elastic Beanstalk = automate provisioning and deployment
https://console.aws.amazon.com/elasticbeanstalk
- Elastic Beanstalk dashboard, create web app, Platform drop-down> Multi-container Docker, select Upload Your Code
- Dockerrun.aws.json: 
```json
{
  "AWSEBDockerrunVersion": 2,
  "containerDefinitions": [
    {
      "name": "mariadb",
      "image": "mariadb:latest",
      "essential": true,
      "memory": 128,
      "portMappings": [
        {
          "hostPort": 3306,
          "containerPort": 3306
        }
      ],
      "environment": [
        {
          "name": "MYSQL_ROOT_PASSWORD",
          "value": "password"
        },
        {
          "name": "MYSQL_DATABASE",
          "value": "wordpress"
        }
      ]
    },
    {
      "name": "wordpress",
      "image": "wordpress",
      "essential": true,
      "memory": 128,
      "portMappings": [
        {
          "hostPort": 80,
          "containerPort": 80
        }
      ],
      "links": [
        "mariadb"
      ],
      "environment": [
        {
          "name": "MYSQL_ROOT_PASSWORD",
          "value": "password"
        }
      ]
    }
  ]
}
```
AWS EC2 Container Service = launch docker containers on an Amazon ECS-optimized Amazon Linux AMI
https://console.aws.amazon.com/ecs
AWS Lambda = serverless setup for execution of a function
CloudFormation, and Simple Queue Service

## Chapter 20 Everything else
https://forums.manning.com/forums/learn-amazon-web-services-in-a-month-of-lunches
DynamoDB = hosted NoSQL
- primary key, sort key, secondary index
Redshift = data warehouses, based on PostgreSQL
ElastiCache = in memory key-value store, based on Redis or Memcached
CodeCommit = hosted git repository. free for the first five active users, 50 GB. Each extra user costs $1/month.
CodeBuild = build system to automate builds, store artifacts on S3
CodeDeploy = deploy app and revisions to EC2 instances in a project’s deployment group, AppSpec file
CodePipeline = workflow tool combining CodeCommit, CodeBuild, and CodeDeploy
API Gateway = manage the opening and closing of API endpoints
CloudFormation = templates that orchestrate your stack, infrastructure as code. More detailed than Elastic Beanstalk
AWS WAF = web application firewall
AWS Shield = protect against DDoS attacks
Cognito = account management, can also enable federated identities with third party(Google, Facebook)
Simple Notification Service = send alerts
Simple Queue Service = message queue
Simple Email Service = manage email list
Guides at https://aws.amazon.com/quickstart

## Chapter 21 additional resources
https://aws.amazon.com/documentation
http://serverfault.com
https://forums.aws.amazon.com/index.jspa
AWS certifications
