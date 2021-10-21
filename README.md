Packer & AMIs 

Instructions to build custom AMI using Packer

Steps to configure and build AMI packer:
1. First create vpc including all subnets, route tables and Internet Gateways in your aws account.
2. Verify if VPC and its resources are created.
3. Set appropriate variables in the buildAmi.sh file.
4. Use default VPC's subnet id for the variable - subnet_id.
5. Use Ubuntu Server 20.04 LTS (HVM), SSD Volume Type as your source image to create custom AMI
   using Packer - As of 10/14/21, the current AMI ID is ami-09e67e426f25ce0d7.
6. Set profile to access your respective aws profile user.
7. Save and open terminal in your VS code editor.
8. Build the AMI by running the command ./buildAmi.sh in terminal.
9. Verify if connection to SSH is successfull from the build logs.
10. Once the build is completed, verify by logging into aws account (dev/prod):
	--> Got to Services > Instances > Verify if an instance is created with the respective instance ID (from build logs) and in "stopped" status.
	--> Open Images dropdown > AMI > Verify if an AMI is created with respective AMI Id (from build logs) and in "available" status. 
	--> If these checks are verified, then build is successful.
	
Steps to launch EC2 instance: Go to instances and click on Launch instances.
Step 1: Choose an Amazon Machine Image (AMI)
	--> Go to My AMIs tab and select the respective AMI that you created and click next.

Step 2: Choose an Instance Type
	--> Select t2.micro and click next.
	
Step 3: Configure Instance Details
	--> Select VPC that you created, in the Network dropdown. (Not Default VPC)
	--> Select any one subnet from the subnet dropdown. 
	--> click next
	
Step 4: Add Storage
	--> Add 8GB size if not added already.
	--> Check the "Delete on termination" box
	--> Click next
	
Step 5: Add Tag - Skip for now and click next
Step 6: Configure Security Group
	--> Select from existing security group or create a new one
	--> Create a new Security group (SG)-
			> Give name and description for SG
			> Add rules for types - HTTP (Port 80), SSH (Port 22), Custome TCP Rule (Port 8080) and HTTPS (Port 443)
			> Add 0.0.0.0/0 and ::/0 for all the above types.
			> Click review and launch
			
Step 7: Review Instance Launch - Review your instance details and click on Launch.
	--> Select existing key pair (RSA key pair if already created), acknowledge and click launch.
	Else
	--> Create new key pair:
		>Choose RSA and give a key pair name 
		>Download the key pair file (.pem file) and place it in ~/.ssh/ folder
		>Click on launch
		
Step 8: Go to instances and check if the newly created instance is running.

Steps to setup and run application on EC2 instance:
Step1: Open new terminal and SSH to your running instance - ssh -i ~/.ssh/KeyPair.pem SSH-username@ec2publicipv4address.amazonaws.com -p 22
	--> Click yes to continue midway
	--> If successful, then SSH to instance is successful
	--> If .pem file has permission issue, locate the .pem file in root directory (locate ~/.ssh)
			> Do ls -ltrh > It shows the existing permissions of all files in ~/.ssh folder. 
			> If the permission on .pem file is r--rw--rw--, change it to r----- by running command chmod 400 filename.pem
			> review permission by running command ls -ltrh. Permission must be changed to r------
			> Now try to ssh again to your instance.

Step2: Setup Mysql database configuration
	--> login on MySQL as root user, type command:
		sudo mysql
	--> Next, Create a database, type command:
		CREATE DATABASE mynewdatabase;
	--> Verify if database is created by running below command:
		show databases;
	-->Create a new user for your new MySQL database and provide a strong password, use the command:
		CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'mypassword';
	-->Add user privileges by following command:
		GRANT ALL PRIVILEGES ON mynewdatabase.* TO 'myuser'@'localhost' WITH GRANT OPTION;
	-->To apply the database changes without reloading your MySQL service. Type command:
		FLUSH PRIVILEGES;
	-->Switch to database you created using below command:
		use mynewdatabase;
	--> Create user entity table by running create table script or by manually writing the query.
	--> Verify if table is created by running command - show tables;
	--> run command exit to exit MySQl.
	
Step3: Verify if MySQL is running by running command sudo service mysql status --> MySQL must be running and in Active status

Step4: Copy application.jar file to EC@ instance using scp command:
	-->Extract application.jar file from your application.
	--> Open a new terminal (any location) and type below command to copy application.jar file to EC2 instance.
		scp -i ~/.ssh/filename.pem /home/user/ApplicationFolder/Target/Application.jar ubuntu@ec2publicipv4address.amazonaws.com:~
	--> result should display percentage completion and file size.

Step5: Go back to your terminal for ec2 instance.
	--> Start your application by running below command:
		java -jar application.jar
	--> Above command must start your springboot application.

Step6: Test your APIs on postman by giving following url
	--> POST API - http://ec2PublicIPv4Addrress.amazonaws.com:8080/v1/user/createUser
	--> GET API - http://ec2PublicIPv4Addrress.amazonaws.com:8080/v1/user/self
	--> PUT API - http://ec2PublicIPv4Addrress.amazonaws.com:8080/v1/user/self
	
Step7: Close terminal once done with testing APIs. 
Step8: Terminate instance (Terminate EC2 instance, delete AMI and VPC in prod account)


		

			

	
	

