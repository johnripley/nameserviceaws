# NameService on AWS

#### Instructions on moving our Spring Boot based nameservice into an EC2 instance on AWS 

 1. Log into your AWS console
    - *this tutorial assumes you already have an AWS account* 
 2. Create an S3 bucket to store your standalone Spring Boot jar
    - bucket name is globally unique, I will use *johnripley-nameserviceaws*
    - upload nameservice/target/nameservice-2016.0.1.jar into your new bucket
 3. Create an EC2 instance to host the Spring boot nameservice
    - Navigate to EC2 on AWS console
    - Launch Instance
    - Choose AMI (Amazon Linux AMI (2017.09.1) will be fine, usually the top one listed is good)
    - Choose an Instance Type (t2.micro-free tier) will be fine for this example) 
    - Configure Instance Details
      - Network (use default vpc)
      - Subnet (no prefs)
      - Auto-assign Public IP (Enable)
      - IAM Role - Create new IAM Role (see IAM Role below)
      - IAM Role (NameServiceWebServer)
    - Add Storage (select defaults)
    - Tags (select defaults or tag as you want)
    - Configure Security Groups
      - Create a new security group (name = NameServiceWebServer)
      - Add/edit rules
	    1. SSH - TCP - 22 - anywhere
	    2. Custom TCP - TCP - 8080 - anywhere
	    3. All ICMP v4 - ICMP - (0-65535) - anywhere (optional allows you to ping service)
    - Launch
      - Create a new key-pair (name = nameservice)
      - Download (nameservice.pem)
      - Launch Instances
      - View Instances
 4. Wait until AWS ECS instance is ready (will say checks 2/2 in status)
 5. Log into you new instance via SSH
    - In AWS Console/EC2 - Right click on instance - choose connect
    - Note instructions regarding chmod 400 nameservice.pem
    - Copy the ssh example (ssh -i "nameservice.pem" ec2-user@xxxxxxxxxx.amazonaws.com) to clipboard
    - Open up terminal
    - Naviagate to directory where you downloaded pem file
    - Run chmod 400 nameservice.pem
    - Paste.run ssh command from clipboard
    - Accept finger print prompt
    - You are now logged into your EC2 instance
 6. Upgrade to Java 8
    - sudo yum install -y java-1.8.0-openjdk.x86_64
    - sudo /usr/sbin/alternatives --set java /usr/lib/jvm/jre-1.8.0-openjdk.x86_64/bin/java
    - sudo /usr/sbin/alternatives --set javac /usr/lib/jvm/jre-1.8.0-openjdk.x86_64/bin/javac	
    - run java -version (should say akin to openjdk version "1.8.0_161")
 7. Copy standalone spring boot jar to ec2 instance
    - aws s3 cp s3://johnripley-nameserviceaws/nameservice-2016.0.1.jar .
 8. Run the spring boot jar
    - java -jar nameservice-2016.0.1.jar
 9. Test service from another machine
    - from AWS EC2 console, get the <public ip address of your EC2 instance>
    - curl -X GET 'http://<public ip address of your EC2 instance>:8080/nameservice/gender?name=Riley&year=2014' -H 'Accept:application/json'	
 10. Don't forget to shut down your EC2 instance when you are done
    - In AWS EC2 console, right click on instance and select
      - Instance State/Stop to shut down the instance (you can restart it later)
      - Instance State/Terminate to destroy the image (you will have to repeat steps 3-9 again)
      - Understand how these states will affect your AWS billing

	
#### IAM Role
  Create Role
  - Trusted Entity - AWS Service - EC2 - EC2 (Allows EC2 instances to call services on your behalf)
  - Permissions - select AmazonS3ReadOnlyAccess
  - Role name - NameServiceWebServer
  - Create Role