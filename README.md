# DevOps CICD Pipeline Setup Through { Jenkins | Nexus | Sonarqube | Tomcat | Maven }


![alt text](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*BTYg-qvM70nF9kco40R_4w.png)


## Documentation
So as a part of this setup, we are going to use several tools first, our project is available on GitHub where the project source code will be available. So will take a project from GitHub and deploy it using different stages with the help of the CICD structure.

## The tools used in the project are
### Jenkins
### Maven
### SonarQube
### Nexus
### Apache tomcat server

### Jenkins
In order to build and deploy that application we are using Jenkins.there are other tools also but jenkins is mostly used and opensource  In Jenkins, we are using the CICD pipeline concept. So this Jenkins software is going to clone the repository from GitHub and will communicate with Maven.

### Maven
It is a build tool used to perform the build process for Java applications.

Now Jenkins will clone the repository from the github and maven will compile and package our java application. After the build process is completed Jenkins should be able to communicate with the Sonarqube.

### SonarQube
To perform code review Sonarqube plays the code quality checking software in the pipeline.

 After performing the code review of our application we want to build an artifact into the Nexus Repository.

 ### Nexus
 Nexus is the repository where all the artifacts are placed.

After uploading the artifact into the nexus repository we want to deploy the application war file into the apache tomcat server.

### Apache tomcat server
is a container that is used to run our web applications.

We will use the Tomcat server for running our Java application. My Jenkins is responsible for communicating with all the tools to automate the application build and deployment process. In order to perform all these steps we are going to create a pipeline in Jenkins.

After that, we will create AWS Linux Machine, you can use any machine in your comfort zone.

We would be creating 4 servers as described below on aws

## For installation of Tomcat Server:

There are two versions of amazon linux.
1. Amazon Linux
2. Amazon Linux 2 (successor of first one)

We are going to use Amazon Linux 2.
### See which packages are available in your repos, just for info
```
yum search tomcat
```
### Install tomcat and tomcat-admin-webapps
tomcat-admin-webapp (usually called manager) is a separate package that you have to install along with tomcat if you need a nice GUI to manage tomcat and deploy apps via GUI.
```
sudo yum install tomcat tomcat-admin-webapps
```
Note 1: Amazon Linux 2 doesn't even come with java. Required java version will be installed as it is a dependency for tomcat.

Note 2: Tomcat is installed in **/usr/share/tomcat** folder.

### Make manager app accessible from everywhere
By default the manager webapp is restricted to the host in which tomcat is installed. Lets make it accessible from anywhere.
```
sudo nano /usr/share/tomcat/webapps/manager/META-INF/context.xml
```
Add the following line inside the `<context>` tag:
```
<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />
```
### Create the user: admin
By default no users are created for the tomcat. Lets create an admin user:
```
sudo nano /usr/share/tomcat/conf/tomcat-users.xml
```
Add the following lines inside `<tomcat-users>` tag:
```
<role rolename="admin"/> 
<role rolename="admin-gui"/> 
<role rolename="admin-script"/> 
<role rolename="manager"/> 
<role rolename="manager-gui"/> 
<role rolename="manager-script"/> 
<role rolename="manager-jmx"/> 
<role rolename="manager-status"/> 
<user name="admin" password="adminadmin" roles="admin,manager,admin-gui,admin-script,manager-gui,manager-script,manager-jmx,manager-status" />
```
### Open ports in firewall
In the security group of your ec2 instance make sure you open port 8080.
### To start Tomcat
```
sudo systemctl start tomcat.service
```
### check of the status of the tomcat service
```
sudo service tomcat status -l
```
### You can also check what ports are open and listening in your server
```
sudo netstat -ntlp | grep LISTEN
```
you can find 8080 listening
### Access the manager webUI
Go to 
```
<ec2-instance-public-IP>:8080/manager/html
```
Login with the credentials given when you created the admin user

### To stop tomcat
```
sudo systemctl stop tomcat.service
```
<br/>
<br/>
<br/>
<br/>

# Commands you may find useful in debugging issues related to installation

To check if java has been installed or to find out the version of java installed:
```
java -version
```
To see which packages are installed (for ex: to check if tomcat is installed or not):
```
rpm -qa|grep tomcat
```
To remove packages (for ex: remove all tomcat packages):
```
sudo yum remove tomcat*
```
To get the PID of tomcat process running (2nd column in the output)
```
ps -ef|grep tomcat
```
Get listening ports and search for this process ID
```
sudo netstat -nlp | grep <PID>
```
To find the command that started a process
```
ps aux |grep <PID>
```
OR
```
ps -p <PID> -o cmd
```
To find the directory from where a command is found in path or to find where is your command in the filesystem
```
whereis <yourcommand> (eg: `whereis java`)
```

## For installation of SonarQube :
## SonarQube Installation And Setup In AWS EC2 Redhat Instance.
##### Prerequisite
+ AWS Acccount.
+ Create Redhat EC2 T2.medium Instance with 4GB RAM.
+ Create Security Group and open Required ports.
   + 9000 ..etc
+ Attach Security Group to EC2 Instance.
+ Install java openJDK 1.8+ for SonarQube version 7.8

## 1. Create sonar user to manage the SonarQube server
```sh
#As a good security practice, SonarQuber Server is not advised to run sonar service as a root user, 
# create a new user called sonar and grant sudo access to manage sonar services as follows

sudo useradd sonar
# Grand sudo access to sonar user
sudo echo "sonar ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/sonar
# set hostname for the sonarqube server
sudo hostnamectl set-hostname sonar 
sudo su - sonar
```
## 1b. Assign password to sonar user
```sh
sudo passwd sonar
```
## 2. Enable PasswordAuthentication in the server
```sh
sudo sed -i "/^[^#]*PasswordAuthentication[[:space:]]no/c\PasswordAuthentication yes" /etc/ssh/sshd_config
sudo service sshd restart
```
### 3. Install Java JDK 1.8+ required for sonarqube to start

``` sh
cd /opt
sudo yum -y install unzip wget git
sudo yum install  java-11-openjdk-devel
```
### 4. Download and extract the SonarqQube Server software.
```sh
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.8.zip
sudo unzip sonarqube-7.8.zip
sudo rm -rf sonarqube-7.8.zip
sudo mv sonarqube-7.8 sonarqube
```

## 5. Grant file permissions for sonar user to start and manage sonarQube
```sh
sudo chown -R sonar:sonar /opt/sonarqube/
sudo chmod -R 775 /opt/sonarqube/
```
### 6. start sonarQube server
```sh
sh /opt/sonarqube/bin/linux-x86-64/sonar.sh start 
sh /opt/sonarqube/bin/linux-x86-64/sonar.sh status
```

### 7. Ensure that SonarQube is running and Access sonarQube on the browser
### sonarqube default port is = 9000 get the sonarqube public ip address 
### publicIP:9000
```sh
curl -v localhost:9000
54.236.232.85:9000
default USERNAME: admin
default password: admin
```

# Setting up Nexus on AWS Linux

## SSH into Linux server

Using putty or MobaXterm or any ssh client

## Install java 

sudo yum install java-1.8.0-openjdk -y


## Install Nexus 3

```
  cd /opt/
  
  sudo wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
```

Untar the file

```
  sudo tar xvf latest-unix.tar.gz
```

Change the ownership

```
  sudo chown -R ec2-user:ec2-user nexus-3.19.1-01 sonatype-work
  
```

## Run nexus as a service

### 1 Change user for nexus
open bin/nexus.rc and make sure the following line is present
```
  run_as_user="ec2-user"
```

### 2 Execute following command

```
  sudo ln -s /opt/nexus-3.19.1-01/bin/nexus /etc/init.d/nexus
 
  cd /etc/init.d
  sudo chkconfig --add nexus
  sudo chkconfig --levels 345 nexus on
  sudo service nexus start
```

## Loging to Nexus3 using browser

```
  http:public-ip:8081
  and follow instructions to get access to nexus
```

## Store artifacts into nexus

We are going to use following repositories to store artifacts

Nexus settings --> repositories
  - maven-release
  - maven-snapshots

## Under Maven configure Nexus details

### 1. Configure nexus user/password.

Open maven settings($MAVEN_HOME/conf/settings.xml) file and add following snippet

```XML
   <servers>
    <server>
      <id>nexusRepo</id>
      <username>admin</username>
      <password>javahome</password>
    </server>
  </servers>
```

### 1. Configure pom.xml of your project

Make suer the following snippet exists in pom.xml

```XML
  <distributionManagement>
		 <snapshotRepository>
		    <id>nexusRepo</id>
		    <url>http://13.233.230.166:8081/repository/maven-snapshots/</url>
		 </snapshotRepository>
		
		<repository>
		    <id>nexusRepo</id>
		    <url>http://13.233.230.166:8081/repository/maven-releases/</url>
		</repository>
  	</distributionManagement>

```

# 1. Setup AWS Ubuntu Server
## 1.1 Launching an AWS Instance
Firstly, click on the `Launch instance` button to start the process of launching a new instance. And name that instance link `Jenkins`.

<img src="./assets/Screenshot (76).png" alt="" width="600" />

<br/>

Choose the Amazon Machine Image (AMI) that you want to use for your instance. This will typically be the operating system you want to use (Ubuntu).

<img src="./assets/Screenshot (77).png" alt="" width="600" />

<br/>

Configure additional details such as the key pair settings and security group (Network) settings.

<img src="./assets/Screenshot (78).png" alt="" width="600" />
<img src="./assets/Screenshot (79).png" alt="" width="600" />

<br/>

Review all settings and click the `Launch` button to create an instance.
Once your instance is launched, it can be connect using a remote desktop or SSH client and start configuring  software and applications.

<img src="./assets/Screenshot (82).png" alt="" width="600" />

<br/>

## 1.2 Adding Jenkins Port to AWS Instance
Select the `Jenkins` instance that you want to add Jenkins port to. Click on the `Security` tab in the details pane at the bottom of the screen. Under `Security details` menu click on the `Security group` link.

<img src="./assets/Screenshot (83).png" alt="" width="600" />

<br/>

Then under the `Inbound rules` tab Click on the `Edit inbound rules` button. 

Inside `Edit inbound rules` section Click on the `Add rule` button to create a new inbound rule.

Select `Custom TCP` rule from the `Type` dropdown menu and enter `8080` as the `Port range`. Then Select `Anywhere` (or a specific IP range) as the `Source` to allow access from any IP address or a specific range of IP addresses.
Finally, click on the `Save rules` button to apply the changes.

<img src="./assets/Screenshot (84).png" alt="" width="600" />
<img src="./assets/Screenshot (85).png" alt="" width="600" />

<br/>
<br/>

# 2. Installing Jenkins on AWS Ubuntu Server

## 2.1 Installing Jenkins
First, update the default Ubuntu packages lists for upgrades with the following command:
```bash
sudo apt update
```
Then, run the following command to install JDK:
```bash
sudo apt install default-jdk -y
```
Now, we will install Jenkins itself. Issue the following four commands in sequence to initiate the installation from the Jenkins repository:
```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update

sudo apt install jenkins -y
```
<br/>

## 2.2 Setting Up Jenkins
When the Jenkins has been installed on the Ubuntu, next step is to make the Jenkins enable using the systemctl command:
```bash
sudo systemctl enable jenkins
```
![](./assets/Screenshot%20(92).png)

<br/>

Once that’s done, start the Jenkins service with the following command:
```bash
sudo systemctl start jenkins
```
![](./assets/Screenshot%20(94).png)

<br/>

To confirm its status, use:
```bash
sudo systemctl status jenkins
```
![](./assets/Screenshot%20(95).png)

<br/>

## 2.3 Allowing Jenkins Port [8080]
With Jenkins installed, we can proceed with adjusting the firewall settings. By default, Jenkins will run on port 8080.

In order to ensure that this port is accessible, we will need to configure the built-in Ubuntu firewall (ufw). To open the 8080 port and enable the firewall, use the following commands:
```bash
sudo ufw allow 8080
```
![](./assets/Screenshot%20(97).png)

<br/>

Then we will enable the UFW(Ubuntu Firewall) service:
```bash
sudo ufw enable
```
![](./assets/Screenshot%20(98).png)

<br/>

Once done, test whether the firewall is active using this command:
```bash
sudo ufw status
```
![](./assets/Screenshot%20(99).png)

<br/>

With the firewall configured, it’s time to set up Jenkins itself. Type in the IP of your EC2 along with the port number. The Jenkins setup wizard will open.

<br/>
## 2.4 Jenkins Credentials
To check the initial password, use the cat command as indicated below:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

All Set! You can now start automating...

## So we will breakdown the project into five phases which are as followed

Stage 1: Clone Repository

Stage 2: Maven Build

Stage 3: Code Review

Stage 4: Upload Artifacts

Stage 5: Deploy the App

### Stage 1: Clone the Repository

So in this pipeline’s first stage, we’re going to clone the git repository in my Github account. 

You can find the details above 

Now let’s start configuring the Jenkins 

Firstly we need to set up maven as a global tool configuration in the Jenkins dashboard.

click on globle tool configuration on jenkins dashbord
go to maven section  click on maven installation
enter version and name 
click on save and apply

## Now start building tahe pipeline
enter name as per project requirements
select free style pipeline project
then click ok

## for source code management we use git so select sample setup  as git
enter repository URl
enter Branch
enter credentials
now apply and save
now run the Build 
check the Councole Output
check the councle it get success in any problem in credentials it will failed

so we have completed our firststage now we proceed to the second stage which is maven Build

## stage 2:maven build
go pipeline 
add maven as build tool save and apply the changes
Just Run the pipeline and see whether the second stage is processing or not. So Click Build Now.
we can check the build get run if any errors in code the build gets failed  
as the second stage is completed which is Maven build and now we will proceed to the third stage which is Code Review.

## Stage 3: Code Review
This stage will be entirely covered by Sonarqube for the code review and analysis. For this, we need to integrate Sonarqube with Jenkins Server. In order to integrate Sonarcube with Jenkins we have to install a plugin named as SonarQube Scanner. Select this plugin and install it without restarting.
Now Login into SonarQube then move into Administrator >> Account >> Security >> Generate Token

Here Generate a Token according to your feasible name and copy the token to the Jenkins panel.

Add to your new Credentials of SonarQube into Jenkins by moving to the path Dashboard >> Credentials >> System >> Global Credentials (unrestricted). Add your Secret Key over here.
In Jenkins Dashboard move into Manage Jenkins >> Configure System >> SonarQube Servers >> Add SonarQube.
add all login credentials to jenkins 

Click on Apply and Save. With this, you have configured Jenkins with SonarQube. Now, let us move into the pipeline and add a stage for the SonarQube Stage

Click on Apply and Save it for further procedures.

Now Run the Build Now and see whether the pipeline is fully functional or not. This is the time we have three stages in the pipeline.

If the Build is in the success stage you will find that your maven web app is in the SonarQube.

Now let’s build our fourth stage.

## Stage 4: Upload the Artifacts

For this stage, we have to configure the Nexus Artifact Repository System into Jenkins.

So move into the Jenkins Dashboard >> Manage Jenkins >> Plugin Manger >> Available

Install the Nexus Artifact Uploader

Let’s add the fourth stage in the Pipeline for the fourth stage.

Provide the details of the Nexus Repository as per your configuration in Jenkins pipeline syntax.

Create a Repository in Sonatype Nexus and pass the user credentials into the Jenkins Pipeline Details.

Now add the details to the Artifact Tabs.

Click on Apply and Save.

Now, let's run the Build now and check whether the pipeline Upload the Artifacts is functioning or not. At this time we have 4 different stages to run.

Now Check Your Nexus Repository, you will find the Artifacts uploaded into the directory system.

Now we can see that the 4 stages have been completed successfully. Now we will proceed to our final stage for Deploying the app.

## Stage 5: Deploy the App

For deployment, we need an ssh-agent. For this move to Dashboard >> Manage Jenkins >>Plugins Manager >> Available

Here install search for the SSH Agent. Install it without restarting.

Now let’s add a stage for the deployment. Use pipeline syntax to create the script.

Now add credentials for the Tomcat Server.
Provide the private key .pem file for the tomcat server and click on ADD. Now Generate the Pipeline Syntax for the Stage.

Now we need to add the Fifth stage to the script for the deployment of our web app.
Click on Apply and Save and Run the Build Now.
As your build is initiated successfully you can now check the tomcat server for the running web application.


### So this is the complete CICD pipeline.


