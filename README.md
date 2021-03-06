# Script for AWS Infrastructure

## CloudFormation
The files `AWS.yaml`  is a  CloudFormation template for provisioning the whole application.
The Architecture involves , two web servers hosting the IIS application will reside in Private subnets each within separate AZ , while two RDGW/jump servers residing in separate public subnets within the same AZs as the web subnets.Additional RDGW will ensure redundancy in case of any failures.

The template involves :  
1.) Asking the user to mention the CIDR for:
    VPC (default is 192.180.0.0/16)  
    Public subnet 1 (default is 192.180.10.0/24)  
    Public subnet 2 (default is 192.180.20.0/24)  
    Web subnet 1/Private Subnet 1 ( default is 192.180.11.0/24)    
    Web subnet 2 / Privae Subnet 2 ( default is  192.180.21.0/24)    

2.) The VPC name is also asked ( default is OPS)

3.) Once the above values are set , the below AWS resources will be created :   

•	VPC  --> The Private VPC to be setup  
•	PublicSubnetA   --> Public Subnet 1  
•	PublicSubnetB  --> Public Subnet 2  
•	WebSubnetA     --> Web server subnet 1 ( private Subnet 1 )  
•	WebSubnetB     --> Web server subnet 2 ( Private subnet 2)  
•	InternetGateway  --> To allow PublicSubnetA and PublicSubnetB to get traffic from the internet  
•	AttachGateway  --> To attach the above internet gateway to the VPC  
•	PublicRouteTable  --> To create a public route table in the VPC to allow traffic to/from the public subnets and internet.  
•	RouteToIGW  --> To create the route to/from the internet  
•	PubSubARouteTableAssociation  --> To allow Public Subnet A send/receive traffic to/from the  internet by associating it to the above created route table.  
•	PubSubBRouteTableAssociation  --> To allow Public Subnet B send/receive traffic to/from the internet by associating it to the above created route table.  
•	ElasticIPA  --> To allocate an Elastic IP to the  NAT gateway in Subnet A.  
•	ElasticIPB  --> To allocate an Elastic IP to the  NAT gateway in Subnet B.  
•	NATGatewayA  --> To create a NAT Gateway in Public Subnet A.  
•	NATGatewayB  --> To create a NAT Gateway in Public Subnet B.  
•	WebSubnetARouteTable  --> Route table to connect the Web SUbnet A to NAT gateway A.  
•	RouteToNATGatewayAfromWeb  --> Create route from Web Subnet A to NAT gateway A via the above route table.  
•	WebSubnetARouteTableAssociation  --> To Associate WebSubnetARouteTable to Web SUbnet A.  
•	WebSubnetBRouteTable  --> Route table to connect the Web SUbnet B to NAT gateway B.  
•	RouteToNATGatewayBfromWeb  --> Create route from Web Subnet B to NAT gateway B via the above route table.  
•	WebSubnetBRouteTableAssociation  --> To Associate WebSubnetBRouteTable to Web SUbnet B.  
•	albext1  --> an Application Load balancer facing the internet to receive HTTP traffic and direct to the instances in Web Subnet A and B , the alb is associated to PublicSubnetA and PublicSubnetB.  
•	albcert  --> To create an HTTPS/TLS certificate using ACM for the load balancer.  
    *Note : This is created once the domain "test-ticketek.tk" is available , as this is essential to provision ACM certificate. Used a free domain provider for the same.  Also ensure that the CNAME generated by the ACM certificate is added to the DNS to ensure the certificate gets validated.   
•	albext1lstn1  --> Listener for albcert created above. This enables the albcert to point to the below created target group.  
•	albext1tg1  --> This is the target group which has the two EC2 instance in WebSubnetA and B assigned.This takes care of the health check on the two EC2 targets.  
•	EC2WebRoleProfile  -->This creates a Webrole profile to allow EC2 assume a role.  
•	EC2WebRole  --> This creates the role for EC2 instances in WebSubnetA and B to assume EC2 role, this is required to be able to RDP to these instances from the jump servers.  
•	Ec2externalWebserverSubneta   --> This is the EC2 intance located in WebSubnetA , this hosts one instance of the webapp.  
•	Ec2externalWebserverSubnetb  --> This is the EC2 intance located in WebSubnetB , this hosts the second instance of the webapp.  
•	RDGWSubnetA  --> This is the RDGW/jump server located in subnet A. This enables us to RDP to the instances in WebSubnetA and B which are located in private subnets.  
•	RDGWSubnetB  -->This is the RDGW/jump server located in subnet B. This enables us to RDP to the instances in WebSubnetA and B which are located in private subnets.  
•	SGalbext1 --> This is the security group which the albext is associate to.  
•	SGexternalWebserver --> This is the security group which is associated to the two EC2 instances in WebSubnetA and B.  
•	SGrdgwserveraccess --> This is the security group to which the above two jump servers are associated.  
•	S3configbucket --> This creates an S3 bucket which is to be used to copy application package onto the instance and run powershell command  to deploy the application within the user-data section of the template.

# Post Environment Build

Now , once you have completed with setting up the AWS infrastruture using CloudFormation , please follow the document 'Build and Publish to IIS.docx' to build and package the IIS application.    

Please ensure to use Visual Studio 2013 to build and publish the package.  

Once done , please proceed with deploying the packaged zip onto a test IIS instance.  

This will ensure we can bake our own custom AMI out of the above configured IIS application and spin up the two EC2 Web Instances using this new AMI.  

Please note to configure a new Administrator password before baking the AMI , use Ec2 Sysprep settings to do this.  

As they will be behind the load balancer , traffic will be routed using the default routing algorithm.  

As mentioned , the Opserver application doesnt support a scalable architecture , there wasnt an auto scaling group created for the same.   

The application architecture needs to be understood before we take any steps to make modifications and attempt to launch it in an auto scaling group.  














