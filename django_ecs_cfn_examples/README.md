## DJANGO BLOG APP STACK CREATION USING CLOUDFORMATION
## What it does :
It creates a autoscalable stack for django blog application with following components created automatically :
- EC2 Instances from CentOS 6.5 base image where dango application is hosted 
- MySQL database on RDS and active endpoint/dbname/password/username are pulled into a file while centos box is booted up and eventually used by django application .It also creates all tables required by django application.
- All aws resoures are put under defined vpc and subnet .
- Autoscaling groups / policies and lauch config .
- Security groups for ec2 instance / RDS are automatically created and assigned . RDS could only be accessed from these instances . Instances sg has just port 22,443,80 open.
- Tags assignment to EC2 instances
- Running scripts for preparing OS enviornment as we used centos 6.5 base verion.
- Running scripts for Django application envoirnment setup and deployment.
- keeps a check on parameter values we provide in cf template
- Enables cloudwatch and new relic monitoring.
- Logs of installation of linux, pythong packages and django application deployment is captured under files /tmp/ossetup.log and /var/log/django_app_installer.log . These can be used to troubleshoot if any instace did not come up correctly. 

## Scaling :
- Scaling of Web server (ec2 instances) happens automatically and could be modified in CF template .
- MySQL db sits on RDS hence could be scaled up with only a few mouse clicks or an API call and no downtime. Frequent backups are another feature.
- Scaled up instances comes under LB and LB only puts them once we have service running.

## Security :
There are measure taken on different levels of stack to make this stack secure. Here are few main points:
- All resorcues are sitting under your vpc and subnet.
- Instances can only be pinged on ports 22/443/80 . And from web end , load balancer has just port 80 opened .
- MySQL RDS db can only be reached by ec2 instances which are attached with a particualar security group.
- Root user cannot ssh into ec2 instance direclty .
- centos uses sha512 instead of md5 for password protection
- cron jobs and wireless are disabled for all users.
- user passwords expire in 90 days and logged in idle users will be removed after 30 minutes .

## Disaster Management :
- If ec2 instance suddenly reboots , nginx service and django app will automatically start on bootup.
- There are frequent backups for MySQL DB which can help in db recovery if something goes wrong.

#Files:
- cf_scripts/behance_cf_template.json : cloudformation template file (s3 link https://s3.amazonaws.com/sample-django-cf-scripts/behance_cf_template.json)
- cf_scripts/os_pkg_installer : Script to prepare Centos for successfull installation of django app and also make ec2 instance secure (s3 link https://s3.amazonaws.com/sample-django-cf-scripts/os_pkg_installer)
- cf_scripts/django_app_installer : Script to install python 2.7 , create virtiualenv , install python package dependencies and then deploy django app. (s3 link : https://s3.amazonaws.com/sample-django-cf-scripts/django_app_installer)
- django_app_scripts/* : settings file for django app, django_app_init file for running django server as service and default.conf file for nginx to run as proxy.

## How to run it :
To deploy sample_cf_tmp.json , you will need to make very minimal changes to this template :
- VpcId ( this is the vpc under which you want to deploy this app)
- InstanceWebSubnets (provide subnet under this vpc, present in us-east-1a as we are using this region)
- ELBInternetSubnets (provide subnets under defined vpc for atleast 2 regions)
- DBSubnets (provide subnets under defined vpc for atleast 2 regions)
- You can assign IAM role to ec2 instance in CF template by changing "IamInstanceProfile" key-value under LaunchConfig Properties. 

** Make sure that we have internet connectivity as we are downloading django app and installation files from internet.
 
Once you make these changes , you can go to aws console and upload this template in Cloudformation service , create stack. Just follow next clicks and create it.
