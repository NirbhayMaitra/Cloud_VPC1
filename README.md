## Creating a web portal for our company with all the security as much as possible. So, we use Wordpress software with dedicated database server. Database should not be accessible from the outside world for security purposes.We only need to public the WordPress to clients. 
### Before explanation of my task, let me introduce you to these terms.

# VIRTUAL PRIVATE CLOUD(VPC)

<img src="images/aa.jpg">

Amazon Virtual Private Cloud (VPC) allows the users to use AWS resources in a virtual network. The users can customize their virtual networking environment as they like, such as selecting own IP address range, creating subnets, and configuring route tables and network gateways. The list of AWS services that can be used with Amazon VPC are :-
1. Amazon EC2
2. Amazon Route 53
3. Amazon WorkSpaces
4. Auto Scaling
5. Elastic Load Balancing
6. AWS Data Pipeline
7. Elastic Beanstalk
8. Amazon Elastic Cache
9. Amazon EMR
10. Amazon OpsWorks
11. Amazon RDS
12. Amazon Redshift

VPC actually Provides NaaS(Network-as-a-Service). In VPC world it is known as Subnets. In AWS, VPC provides Boundary to isolate our infrastructure.

<img src="images/ac.jpg">

#### Note:- If our instances are running on different Data-Center but they have the same VPC, they will have high-speed Connectivity. When we create AWS Account, by default they create a VPC and three Subnets for us and launch each Subnet in each Data-Center for disaster Recovery.

# AWS Subnets
A default Subnet is a public subnet, because the main route table sends the subnet's traffic that is destined for the internet to the internet gateway. You can make a default subnet into a private subnet by removing the route from the destination 0.0.0.0/0 to the internet gateway. However, if you do this, no EC2 instance running in that subnet can access the internet.

Instances that you launch into a default subnet receive both a public IPv4 address and a private IPv4 address, and both public and private DNS hostnames. Instances that you launch into a nondefault subnet in a default VPC don't receive a public IPv4 address or a DNS hostname. You can change your subnet's default public IP addressing behavior. 

<img src="images/ab.png">

## Public Subnets
A public subnet has an outbound route that sends all traffic through what AWS calls an Internet Gateway (IGW). The IGW lets traffic — IPv4 or IPv6 — out of the VPC without any constraints on bandwidth. Instances in public subnets can also receive inbound traffic through the IGW as long as their security groups and Network-ACLs allow it.

## Private Subnets
Contrast that with a private subnet which, if it needs to talk to the internet, must do so through a NAT(Network Access Translation) gateway. NATs are really common. Run a wireless router? The router itself does network address translation. Importantly a NAT won’t allow inbound traffic from the internet — that’s what makes a private subnet, well, private.

So what's the difference between Public Subnet and Private Subnet?

A <strong> public subnet </strong> has a route table that says, “send all outbound traffic (anything to the CIDR block 0.0.0.0/0) via this internet gateway.” A <strong> private subnet </strong> either does not allow outbound traffic to the internet or has a route that says, “send all outbound traffic via this NAT gateway.”

It’s important to note that all subnets in a VPC can talk to one another as long as the host’s security groups allow it. There’s always a route that says “keep traffic to {YourVPCsCIDRblock} inside the VPC.”

### What is a Gateway and What Does it Do?

<img src="images/AF.png">

A gateway is a node (router) in a computer network, a key stopping point for data on its way to or from other networks. Thanks to gateways, we are able to communicate and send data back and forth. The Internet wouldn't be any use to us without gateways (as well as a lot of other hardware and software).

Lets start with our Task Number 3. The Objectives are:-

## We have to create a web portal for our company with all the security as much as possible. So, we use Wordpress software with dedicated database server. Database should not be accessible from the outside world for security purposes.We only need to public the WordPress to clients.So here are the steps for proper understanding!

<strong> Steps:-

1) Write a Infrastructure as code using terraform, which automatically create a VPC.

2) In that VPC we have to create 2 subnets:

  a) public subnet [ Accessible for Public World! ] 

  b) private subnet [ Restricted for Public World! ]

3) Create a public facing internet gateway for connect our VPC/Network to the internet world and attach this gateway to our VPC.

4) Create a routing table for Internet gateway so that instance can connect to outside world, update and associate it with public subnet.

5) Launch an ec2 instance which has Wordpress setup already having the security group allowing port 80 so that our client can connect to our wordpress site.

Also attach the key to instance for further login into it.

6) Launch an ec2 instance which has MYSQL setup already with security group allowing port 3306 in private subnet so that our wordpress vm can connect with the same.

Also attach the key with the same.

Note: Wordpress instance has to be part of public subnet so that our client can connect our site. 

Mysql instance has to be part of private subnet so that outside world can't connect to it. </strong>

So Lets start:

>provider "aws" {          <br>
>  region  = "ap-south-1"  <br>
>  profile = "Zulu"        <br>
>}                         <br>

1. Choose the Provider whom Terraform will contact. In my case it is AWS.


>resource "aws_vpc" "main" {            <br>
>  cidr_block       = "192.168.0.0/16"  <br>
>  instance_tenancy = "default"         <br>
>                                       <br>
>  tags = {                             <br>
>    Name = "Nirbhay-VPC"               <br>
>  }                                    <br>
>}                                      <br>

2. Create IaaS code for terraform that will create a VPC for you. Now we have to create two subnets inside our VPC.

>resource "aws_subnet" "subnet1" {       <br>
>  vpc_id     = "${aws_vpc.main.id}"     <br>
>  cidr_block = "192.168.1.0/24"         <br>
>  map_public_ip_on_launch = true        <br>
>  availability_zone = "ap-south-1a"     <br>
>  tags = {                              <br>
>    Name = "Nirbhay-Subnet_Public-1a"   <br>
>  }                                     <br>
>}                                       <br>

3. Create a public Subnet that will be accessible to the public World.

>resource "aws_subnet" "subnet2" {        <br>
>  vpc_id     = "${aws_vpc.main.id}"      <br>
>  cidr_block = "192.168.2.0/24"          <br>
>  availability_zone = "ap-south-1a"      <br>
>  tags = {                               <br>
>    Name = "Nirbhay-Subnet_Private-1b"   <br>
>  }                                      <br>
>}                                        <br>

4. Create a private Subnet that will be restricted for the access to the public world.

>resource "aws_internet_gateway" "gw" {  <br>
>  vpc_id = "${aws_vpc.main.id}"         <br>
>  tags = {                              <br>
>    Name = "Nirbhay-Gateway"            <br>
>  }                                     <br>
>}                                       <br>

5. Create an Internet Gateway for our VPC so that it can be connected to the outside World.

>resource "aws_route_table" "rt" {                   <br>
>  vpc_id = "${aws_vpc.main.id}"                     <br>
>  route {                                           <br>
>    cidr_block = "0.0.0.0/0"                        <br>
>    gateway_id = "${aws_internet_gateway.gw.id}"    <br>
>  }                                                 <br>
>  tags = {                                          <br>
>    Name = "Nirbhay-pubic-rt"                       <br>
>  }                                                 <br>
>}                                                   <br>

6. Create a Route Table for our VPC so that our instances could connect to the outside world.

>resource "aws_route_table_association" "subnet_association" { <br>
>  subnet_id      = aws_subnet.subnet1.id                      <br>
>  route_table_id = aws_route_table.rt.id                      <br>
>}                                                             <br>

7. Associate your Route Table with Public Subnet.

>resource "aws_security_group" "sg_webserver" {   <br>
>  name        = "Security_group_for_Wordpress"   <br>
>  description = "Allow ssh and httpd"            <br> 
>  vpc_id      = "${aws_vpc.main.id}"             <br>
>  ingress {                                      <br>
>    description = "SSH Port"                     <br>
>    from_port   = 22                             <br>
>    to_port     = 22                             <br>
>    protocol    = "tcp"                          <br>
>    cidr_blocks = ["0.0.0.0/0"]                  <br>
>  }                                              <br>
>   ingress {                                     <br>
>    description = "HTTPD Port"                   <br>
>    from_port   = 80                             <br>
>    to_port     = 80                             <br> 
>    protocol    = "tcp"                          <br>
>    cidr_blocks = ["0.0.0.0/0"]                  <br>
>  }                                              <br>
>  egress {                                       <br>
>    from_port   = 0                              <br>
>    to_port     = 0                              <br>
>    protocol    = "-1"                           <br>
>    cidr_blocks = ["0.0.0.0/0"]                  <br> 
>  }                                              <br>
>  tags = {                                       <br>
>    Name = "Nirbhay_sg1"                         <br>
>  }                                              <br>
>}                                                <br>

8. Create a Security Group allowing Port 80 so that our client can connect to Wordpress and allowing port 22 so that our client can do SSH.

>resource "aws_security_group" "sg_database" {    <br>
>  name        = "for_MYSQL"                      <br> 
>  description = "Allow MySQL"                    <br>
>  vpc_id      = "${aws_vpc.main.id}"             <br>
>  ingress {                                      <br>
>    description = "MySQL"                        <br>
>    from_port   = 3306                           <br>
>    to_port     = 3306                           <br>
>    protocol    = "tcp"                          <br>
>  }                                              <br>
>  egress {                                       <br>
>    from_port   = 0                              <br>
>    to_port     = 0                              <br>
>    protocol    = "-1"                           <br> 
>    cidr_blocks = ["0.0.0.0/0"]                  <br>
>  }                                              <br>
>  tags = {                                       <br>
>    Name = "Nirbhay_sg2"                         <br>
>  }                                              <br>
>}                                                <br>

9. Create a Security Group allowing Port 3306 so that our wordpress can connect to MySQL.

>variable ssh_key_name {                                     <br>
>    default = "mykey5555"                                   <br>
>}                                                           <br>
>resource "tls_private_key" "key-pair" {                     <br>
>  algorithm = "RSA"                                         <br>
>  rsa_bits = 4096                                           <br>
>}                                                           <br>
> resource "local_file" "private-key"{                       <br>
> content = tls_private_key.key-pair.private_key_pem         <br>
> filename = "${var.ssh_key_name}.pem"                       <br>
> file_permission = "0400"                                   <br>
>}                                                           <br>
>                                                            <br>
>resource "aws_key_pair" "deployer" {                        <br> 
>  key_name   = var.ssh_key_name                             <br>
>  public_key = tls_private_key.key-pair.public_key_openssh  <br>
>}                                                           <br>
>                                                            <br>

10. Create a Key-Pair for our Instances or use already created key-pair. Here i created the Key with RSA Algorithm which Encrypt our data.

>resource "aws_instance" "web" {                                           <br>
>  ami           = "ami-000cbce3e1b899ebd"                                 <br>
>  instance_type = "t2.micro"                                              <br>
>  availability_zone = "ap-south-1a"                                       <br>
>  subnet_id      = "${aws_subnet.subnet1.id}"                             <br>
>  associate_public_ip_address = true                                      <br>
>  key_name = "${var.ssh_key_name}"                                        <br>
>  vpc_security_group_ids = ["${aws_security_group.sg_webserver.id}"]      <br>
>                                                                          <br>
>  tags = {                                                                <br>
>    Name = "NirbhayOS-wordpress"                                          <br>
>  }                                                                       <br>
>}                                                                         <br>

11. Launch an EC2 instance that has already wordpress in it and attach it with the security group having enabled port 80 so that our clients could connect it.

>resource "aws_instance" "Mysql-OS" {                                      <br>
>  ami           = "ami-0019ac6129392a0f2"                                 <br>
>  availability_zone = "ap-south-1a"                                       <br>
>  instance_type = "t2.micro"                                              <br>
>  subnet_id      = "${aws_subnet.subnet2.id}"                             <br>
>  key_name = "${var.ssh_key_name}"                                        <br>
>  vpc_security_group_ids = ["${aws_security_group.sg_database.id}"]       <br>
>                                                                          <br>
>  tags = {                                                                <br>
>    Name = "NirbhayOS-mysql"                                              <br>
>  }                                                                       <br>
>}                                                                         <br>

12. Launch an EC2 instance that has already MySQL in it and attach it with the security group having enabled port 3306 so that our wordpress could connect to it.

>resource "null_resource" "nullremote1" {
>  connection {
>    type = "ssh"
>    user = "bitnami"
>    host = aws_instance.web.public_ip
>    private_key = file("${var.ssh_key_name}.pem")
>  }
>
>  provisioner "remote-exec" {
>    inline = [
>      "sudo /opt/bitnami/ctlscript.sh restart apache",
>      "sudo /opt/bitnami/ctlscript.sh status",
>
>    ]
>  }
>}

13. For testing purpose you can create null resource and checking the SSH and Public IPs and getting the Public IPs and configuring our Apache Server.


### So our setup is Ready and we are good to go. But first let me show you i haven't created anything beforehand.

<img src="images/14.JPG">
No instances created.

<img src="images/15.JPG">
No VPC and Subnets created.

<img src="images/16.JPG">
Now Run Terraform init command. It will download the Plugins for you. Then run Terraform apply -auto-approve. If everything is well it will add resources and you can cross-check 
it by going inside your AWS console.

<img src="images/17.JPG">

Wordpress instance has been deployed.

<img src="images/18.JPG">

MySQL instance has been deployed.

<img src="images/19.JPG">

VPC has been created.

<img src="images/20.JPG">

Public and Private Subnets have been created

<img src="images/21.JPG">

Route Table has been created.

<img src="images/22.JPG">

Internet Gateway has been created.

<img src="images/23.JPG">

Now go to Wordpress instance and copy the public IP of the instance.

<img src="images/24.JPG">

Paste it on your Browser and you will notice the Wordpress site come up. 

## Task completed Successfully. If you have any questions, Feel free to contact me or mail me at nirbhaymaitra684@gmail.com. 








