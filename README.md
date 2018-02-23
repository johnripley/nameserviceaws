# NameService on AWS Elastic Beanstalk

#### Instructions on moving our Spring Boot based nameservice into onto AWS using Elastic Beanstalk 

 1. Log into your AWS console
    - *this tutorial assumes you already have an AWS account* 
 2. Navigate to the ElasticBeanstalk service
    - click "Create New Application" in top right of welcome screen
    - choose an application name and description (nameservice), click next
    - click "Create web server" button
    - select platform (Java), environment type (Load balancing, auto scaling), click next
    - application source (Upload your own), Choose file (upload your spring boot jar - nameservice/target/nameservice-2016.0.1.jar), click next
    - environment name (nameservice-prod), click next
    - check "Create this environment inside a VPC", click next
    - instance type (t2.micro), EC2 keypair (choose the one you created previously, nameservice), click next
    - add environment tags (optional), click next
    - select ELB and EC2 from two AZs, click next
    - permissions - use defaults, click next
    - review - click Launch
    - wait...
    - note the url at the top of the page (nameservice-prod.us-east-1.elasticbeanstalk.com) <= this is your ELB's external address
    - however, we will need to 
      - open up port 8080 in our VPC
      - redirect port 80 in ELB to 8080 of our EC2-based server
 3. Open up ports
    - navigate to EC2 service
    - select security groups in left pane (you will see security groups (one for the VPC, one for the Load Balancer)
    - click on VPC Security group
    - edit Inbound, add Custom TCP, protocol=TCP, port=8080, source=anywhere, click save
    - click on Load Balancer group
    - edit Outbound, add Custom TCP, protocol=TCP, port=8080, source=anywhere, click save
    - click on "Load Balancers" in left hand pane, edit "Listeners", change instance port from 80 to 8080
 4. Test the service
    - curl -X GET 'nameservice-prod.us-east-1.elasticbeanstalk.com/nameservice/gender?name=Connor&year=2014' -H 'Accept:application/json'