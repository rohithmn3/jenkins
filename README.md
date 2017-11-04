# jenkins
JENKINS:

1) #which java (check whether java is installed or not)
2) #netstat -tulpn | grep 8080 (check whether something running on tomcat port)
3) If java is not installed, download rpm package from oracle website and install it:
   jdk (rpm): http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
   file name: jdk-8u121-linux-x64.rpm
   #rpm -Uvh jdk-8u121-linux-x64.rpm
   #which java

4)  #alternatives --install /usr/bin/java java /usr/java/latest/bin/java 200000
    #alternatives --install /usr/bin/java javac /usr/java/latest/bin/javac 200000
    #alternatives --install /usr/bin/java jar /usr/java/latest/bin/jar 200000
	#vi /etc/rc.local
	    export JAVA_HOME="/usr/java/latest"

=======================================================================================================================================
INSTALL JENKINS:

1) #wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
   #cat /etc/yum.repos.d/jenkins.repo
   #rpm --import http://pkg.jenkins.io/redhat-stable/jenkins.io.key (import the signing key)
   #yum list jenkins
   ----
   ----
   #yum list jenkins-2.19.4-1.1
   #yum install -y jenkins-2.19.4-1.1
   
   Be sure to disable the source as described at the end of the video as well, so you don't accidentally get an update. 
   The certification test is built around this version specifically. That's done with the following:
   #yum-config-manager --disable jenkins
   
   Check 8080 is open: 
   #netstat -tulpn | grep 8080
   
   Start jenkins
   #systemctl start jenkins
   
   [root@ip-172-31-21-19 ec2-user]# service jenkins start
Starting Jenkins                                           [  OK  ]
[root@ip-172-31-21-19 ec2-user]# netstat -tulpn | grep 8080
tcp        0      0 :::8080                     :::*                        LISTEN      23283/java
[root@ip-172-31-21-19 ec2-user]#

	#chkconfig jenkins on
	
	Launch Web GUI:
	http://<public dns>:8080
	
=======================================================================================================================================

USER MANAGEMENT AND SECURITY:
-> Login with our admin user "rohithmn03"
-> Manage Jenkins -> Configure Global Security -> Authorization -> check "Matrix Based Security"
    Add user which we have already created "rohithmn03" and give all permissions.
	Create new developer user -> and again add that user under "Matrix Based Security" -> Provide only required permissions
	
=======================================================================================================================================

ADDING A JENKINS SLAVE:
-> In this lesson we add a jenkins slave using SSH
-> We have have slave nodes to distribute the build load.
-> We can either specify the subset of slave nodes or a particular slave node, so that the job runs on it.
	on master:
	#su jenkins -s /bin/bash
	#cat /var/lib/jenkins/.ssh/id_rsa.pub
	
	on client:
	#sudo su 
	#useradd -d /var/lib/jenkins jenkins
	#mkdir /var/lib/jenkins/.ssh
	#vi /var/lib/jenkins/.ssh/authorized_keys (copy the generated rsa pub key)
	- make sure java is installed here as well

-> Login to jenkins with Admin User
-> Manage jenkins -> Manage Nodes -> New Node
		Number of executors: 2
		Remote root directory: /var/lib/jenkins
		Usage: Use this node as much as possible
		Launch method: Launch slave agents via SSH
		
-> Create a test freestyle jobs and make run on that slave node: check "Restrict where this project can be run" Label Expression= salve1
	Build this project and verify the output.
	
=======================================================================================================================================

SETTING UP GIT HUB:
-> Sign up for a new Git Hub account
-> Update the is_rsa_pub key in your git settings.
-> Create a new project "jenkins" and clone it on your Desktop
-> $git clone git@github.com:rohithmn3/jenkins.git

=======================================================================================================================================

PLUGIN MANAGER:
-> Login into jenkins as a admin user
-> Manage Jenkins -> Manage plugins (we can even download the *.hsi file and upload it for new plugin installation)
	
=======================================================================================================================================

SCM - Source Code Management and the Git Plugin:

-> Login to jenkins master server and slave node and install git.
-> sudo root
-> #yum install -y git
-> Login to Jenkins GUI -> click on the Free style project just created -> configure
under Source Code Management -> select Git
Repository URL: Clonable link
Credentials: Add Git credentials
Branches: master (if we leave it blank, it will build any branch on the repository)
Repository Browser: githubweb
URL: Same as the GitHub project URL (https://github.com/rohithmn3)
-> Build Triggers:
select POLL SCM -> This will poll any changes in our repo at the schedules time.
schedule -> set it for every 15 mins -> H15 * * * *
-> Build:
Execute Shell = git log
SAVE
=======================================================================================================================================
GIT Hooks and Other Build Triggers:

-> Login to Jenkins GUI -> click on the Free style project just created -> configure
-> Now we will try git hooks to build trigger as soon as one commits to the branch.
Build Triggers -> Select POLL SCM -> don't fill anything under schedule area. -> save
Login to GitHub Account -> clink on our project -> settings -> Integration & Services -> Add Service -> Jenkins Git Plugin ->Jenkins URL -> Active -> DONE
Jenkins URL: https://<your AWS lab server>:8080
Now login to Git Bash on our local machine -> Push some changes to the jenkins repo -> Now Build would have automatically triggered.
-> Now, we will select Jenkins GitHub plugin. (both are almost same)
Build Triggers -> Select GitHub hook trigger -> save
Login to GitHub Account -> clink on our project -> settings -> Integration & Services -> Add Service -> Jenkins GitHub plugin -> Jenkins hook URL -> Active
Jenkins URL: https://<your AWS lab server>:8080/github-webhook/
Now login to Git Bash on our local machine -> Push some changes to the jenkins repo -> Now Build would have automatically triggered.
========================================================================================================================================
Workspace Environment Variables:

-> Login to Jenkins GUI -> click on the Free style project just created -> configure
-> Add one more build step -> Execute Shell -> enter the below code which has Jenkins ENV variables
echo "Generic Jenkins Variables"
echo "Build Number $BUILD_NUMBER"
echo "Node Name: $NODE_NAME"
echo "Job_Name: $JOB_NAME"
echo "Executor Number: $EXECUTOR_NUMBER"
echo "Workspace: $WORKSPACE" (path to the workspace where this is running)
-> Save and Build it

-> Add one more build step -> Execute Shell -> enter the below code which has GIT ENV variables
echo "GIT Environment Variables"
echo "GIT commit $GIT_COMMIT"
echo "GIT Branch $GIT_BRANCH"
echo "GIT Previous Commit $GIT_PREVIOUS_COMMIT"
echo "GIT URL $GIT_URL" (clonable URL)
=========================================================================================================================================
PARAMETEREIZED PROJECTS:

-> Login to Jenkins GUI -> click on the Free style project just created -> configure
-> General -> Select "This Project is PARAMETEREIZED" -> Add Parameter -> 
Choice Parameter and Extended choice parameter(animals-dog,cat,lion,tiger)
String Parameter
==========================================================================================================================================

Upstream/Downstream projects and the Parametrized Trigger Plugin:

-> Login to Jenkins GUI -> create new project by name "DownStream_Project" -> Build Triggers -> Build After other projects are build->project="freestyle_proj"
-> our upstream project will be our Free style project.
Upstream project: A job that triggers the other job.
Downstream Project: A job that will be triggered by another job.
-> Manage Plugins -> Available -> search for "parametrized trigger plugin" -> install and restart

=========================================================================================================================================

FOLDERS:
-> If there are hundred of jobs created on jenkins, it will be difficult to manage all. Folders will help us to manage huge number of jobs.
-> New Item -> Enter the Folder Name("Freestyles") -> select Folder option -> OK
Name: Freestyles
Health Metrix: Recursive  this (means if one of the job failed inside this folder, then the health of the entire folder will be shown NOT OK )
-> Now move the jobs to this newly created folder.

=========================================================================================================================================

VIEWS:
-> CLick "+" on the home dashboard.
-> There will be two views that we can select 
Build Pipeline View: Shows the jobs in a build pipeline view. 
List view
My View
-> We can make any view as the default view under -> Manage Jenkins -> Configure System -> Default View -> <View Name>
-> We can create User specific views, which can be viewed only by that user.
==========================================================================================================================================

OUR JAVA PIPELINE PROJECT

-> Here we are creating a simple Java project to work on jenkins.
-> Login to github.com -> Create a new repository -> clone that repo to local machine
	$ cd java-project
	$ mkdir src
	$ mkdir lib
	$ mkdir bin
	$ mkdir dist
	$ mkdir docs
	$ mkdir reports
	$ vim .gitignore
		bin/*
		dist/*
		reports/*
		docs/*
	$ touch jenkinsfile
	$ vim src/Rectangle.java
		public class Rectangle {
			public int length;
			public int width;
			
			public Rectangle(int length, int width) {
				this.length=length
				this.width=width
			}
			
			public int getArea() {
				return length * width;
			}
			
			public int getPerimeter() {
				return 2 * (length+width)
			}
		}
	
	$ vim src/Rectangulator.java
		public class Rectangulator {
			public static void main(String args[]) {
				int length=Integer.parseInt(args[0]);
				int width=Integer.parseInt(args[1]);
				
				Rectangle myRectangle=new Rectangle(length,width);
				
				String output=String.format("*** Your Rectangle *** \n\nLength: %d\nWidth: %d\nArea: %d\nPerimeter: %d\n\n", myRectangle.length, myRectangle.width, myRectangle.getArea(), myRectangle.getPerimeter());
				System.out.println(output)
				
			}
		}
-> Now add, commit and push the java code to the repo.
	$ git add .
	$ git commit -m "Initial Java Commit"
	$ git push origin master
-> Create a new branch called development
	$ git checkout -b development
	$ git push origin development

==========================================================================================================================================

DOCKER INSTALL:
-> We get Docker installed on the Jenkins Master and Slave node.
-> Remove any existing Docker containers.
	#yum remove docker docker-common container-selinux
	#yum install -y yum-utils (confirming whether yum-utils package is available or not)
	#yum-config-manager --add-repo https://dowload.docker.com/linux/docker-ce.repo (command to create docker repo)
	#cat /etc/yum.repos.d/docker-ce.repo
	#yum clean all (to clear the cache)
	#yum install -y docker-ce-17.03.0.ce-1.e17.centos

-> Docker is installed, now we need to add the Jenkins user to the Docker group
	# gpasswd -a jenkins docker (adding jenkins user to docker group)
	# systemctl start docker
	
-> Restart Jenkins on Master
	# systemctl restart jenkins
	
-> Login into Jenkins -> Configure Free style project -> Execute Shell -> docker run hello-world (this spins up container) -> Save
	
==========================================================================================================================================

INSTALLING AND CONFIGURING ANT:
-> First install ANT on both the master and the slave nodes.
-> # wget http://www.us.apache.org/dist/ant/binaries/apache-ant-1.10.1-bin.tar.gz
   # tar xvfz apache-ant-1.10.1-bin.tar.gz -C /opt
-> Create a symbolic link
   # ln -s /opt/apache-ant-1.10.1/ /opt/ant
-> Set /opt/ant as out ANT_HOME env variables
   # sh -c 'echo ANT_HOME=/opt/ant >> /etc/environment'
-> Another symbolic link to get ant running
   # ln -s /opt/ant/bin/ant /usr/bin/ant
   #ant -version
   ******************** OR ***********************
   27  java -version
   28  yum remove java-1.7.0-openjdk
   29  yum install java-1.8.0-openjdk-devel -y
   30  vim /etc/profile
   31  ls /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-1.b12.35.amzn1.x86_64/jre
   32  source /etc/profile
   33  javac
   34  java -version
   35  cd
   36  wget http://www-us.apache.org/dist//ant/binaries/apache-ant-1.9.9-bin.tar.gz
   37  ls
   38  tar -xvzf apache-ant-1.9.9-bin.tar.gz -C /opt/
   39  ln -s /opt/apache-ant-1.9.9/ /opt/ant
   40  sh -c 'echo ANT_HOME=/opt/ant >> /etc/environment'
   41  ln -s /opt/ant/bin/ant /usr/bin/ant
   42  ant -version
   43  ant
   44  clear
   45  history
   *************************************************
-> Go to Git Bash(our java-project repo) -> create a file build.xml where .gitignore is present (i.e., on the root project directory)
	build.xml (https://github.com/linuxacademy/content-jenkins-java-project)
	
-> Try building the java-project manually using build.xml and later we can delete it and build it from Jenkins.
   Login to GitBash
   $git add .
   $git commit -m "build.xml changes"
   $git push origin development
   Go to jenkins master node
   $ su jenkins -s /bin/bash
   $ cd /var/lib/jenkins
   Temporarily clone the repo: (we can fetch the url from Git bash java-project repo #cat .git/config take the URL under remote "origin")
   $ git clone <url>
   $ cd java-project
   $ ls
   $ git checkout development
   $ ls (now we can see the build file build.xml)
   $ ant -f build.xml -v
   $ ls -l
   $ cd dist
   $ ls -l
   $ java -jar rectangle.jar 4 5 (4 and 5 are arguments for our java project rectangle.java length and width)
   
   Now delete the repo cloned
   $ rm -rf java-project
   
==========================================================================================================================================
