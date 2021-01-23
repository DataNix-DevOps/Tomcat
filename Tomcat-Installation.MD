 #                           How To Install Apache Tomcat 9.0 / 8.5 on CentOS 7 / RHEL 7


Apache Tomcat is an open source web server and servlet container developed by the Apache Software Foundation (ASF). Tomcat implements the Java Servlet and the JavaServer Pages (JSP) specifications from Oracle and provides a “pure Java” HTTP web server environment for running the Java codes.

Here is the step by step guide to install Apache Tomcat 9.0 / 8.5 on RHEL 7 / CentOS 7.
Prerequisites

Open a terminal and switch to root user.
Tomcat requires having java installed on your machine. You can install either Oracle JDK or OpenJDK.

READ: How to Install Oracle JAVA 8 on CentOS 7 / RHEL 7

For this post, I am using OpenJDK.
```sh
yum install -y java-1.8.0 wget vim
yum install java-11-openjdk-devel wget vim
```
You can also verify it, by issuing the following command.
```sh
java -version
```
Output:

openjdk version "1.8.0_212"
OpenJDK Runtime Environment (build 1.8.0_212-b04)
OpenJDK 64-Bit Server VM (build 25.212-b04, mixed mode)

Create Tomcat Service Account

For best practice, Tomcat should not be run as root user. So, create a normal user for running the Tomcat service.
```sh
groupadd tomcat
useradd -g tomcat -d /opt/tomcat -s /bin/nologin tomcat
```
Download & Setup Apache Tomcat

Download the latest version of the Apache Tomcat from the website and save it to your working directory.
Browser

Apache Tomcat 9:

Download Apache Tomcat 9.0

Apache Tomcat 8.5:

Download Apache Tomcat 8.5
Terminal

### Apache Tomcat 9.0 ###
```sh
wget https://www-us.apache.org/dist/tomcat/tomcat-9/v9.0.19/bin/apache-tomcat-9.0.19.tar.gz
```
### Apache Tomcat 8.5 ###
```sh
wget https://www-us.apache.org/dist/tomcat/tomcat-8/v8.5.40/bin/apache-tomcat-8.5.40.tar.gz
```
Extract the Tomcat archive and move it to your desired (/opt/tomcat) directory.
```sh
tar -zxvf apache-tomcat-*.tar.gz
mv apache-tomcat-*/* /opt/tomcat/
```
Change the ownership of the directory so that tomcat user can write files in it.
```sh
chown -R tomcat:tomcat /opt/tomcat/
```
Start Apache Tomcat
Manual

Apache Tomcat can be started and stopped by the script which comes with the package. Start the Apache Tomcat.
```sh
cd /opt/tomcat/bin/
sh startup.sh
```
You will get the following output.

Using CATALINA_BASE: /opt/tomcat
Using CATALINA_HOME: /opt/tomcat
Using CATALINA_TMPDIR: /opt/tomcat/temp
Using JRE_HOME: /usr
Using CLASSPATH: /opt/tomcat/bin/bootstrap.jar:/opt/tomcat/bin/tomcat-juli.jar
Tomcat started.

Stop the Apache Tomcat.
```sh
cd /opt/tomcat/bin/
sh shutdown.sh
```
Systemd

We can also configure systemd to start the Tomcat service. Skip the below step in case you do not want to use systemd for managing Tomcat service.

Tomcat’s systemd service file requires java location. So, run the following command to list the java versions available on your system.
```sh
alternatives --list  | grep ^java
```
Output:

java    manual  /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64/jre/bin/java

At this time, I have Java 1.8 on my system.

Create a tomcat systemd service file. Green ones depend on the environment, so change them accordingly.
```sh
vi /etc/systemd/system/tomcat.service
```
Add below information to Tomcat systemd service file.
```sh
[Unit]
Description=Apache Tomcat Web Application Container
Wants=network.target
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64/
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1G -Djava.net.preferIPv4Stack=true'
Environment='JAVA_OPTS=-Djava.awt.headless=true'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh
SuccessExitStatus=143

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```
Reload systemd daemon.
```sh
systemctl daemon-reload
```
To start the Tomcat service; run:
```sh
systemctl start tomcat
```
Check the status of Tomcat, run:
```sh
systemctl status tomcat
```
Enable the auto start of Tomcat service on system start, run:
```sh
systemctl enable tomcat
```
Verify Apache Tomcat

By default, Tomcat runs on port no 8080. Use netstat command to check whether the service is listening on port 8080 or not.
```sh
netstat -antup | grep 8080
```
```sh
Output:

tcp6       0      0 :::8080                 :::*                    LISTEN      2428/java
```
Firewall

You may need to allow port 8080 in the firewall so that we can access Tomcat from external networks.
```sh
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload
```
Configure Apache Tomcat Web UI

Tomcat can be managed through the Web Manager. With Web Manager, we can

    Deploy new application, deploy
    Deploy new application on the specified context
    List the active or inactive applications
    Start and stop the web applications

Also, Tomcat has Host Manager to manage its virtual hosts.

Both The Web and Host Manager is password protected, requires username and password to access. Only the user with the “manager-gui” and “admin-gui” role is allowed to access.

These users and roles are defined in tomcat-users.xml.
```sh
vi /opt/tomcat/conf/tomcat-users.xml
```
Place the following two lines just above the last line.
```sh
<role rolename="admin-gui,manager-gui"/>
<user username="admin" password="tomcat" roles="manager-gui,admin-gui"/>
```
For security reason, Web and Host Manager is accessible only from localhost, i.e., from the server itself.

If you want to access Web and Host manager from remote systems, then you need to add your source network in allow list. To do that, edit the below two files.
```sh
vi /opt/tomcat/webapps/manager/META-INF/context.xml

vi /opt/tomcat/webapps/host-manager/META-INF/context.xml
```
Update the below line on both files with source IP from which you’re accessing the Web and Host Manager. .* will allow everyone to have access to Web and Host manager.
```sh
allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|.*" />
```
OR

You can also allow part of your network only. For example: To allow 10.20.0.0/24 network only, you can use the below values.
```sh
allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|10.20.*" />
```
Restart the Tomcat service.
```sh
systemctl restart tomcat
```
Access Tomcat

Open the web browser and point to
http://ip.add.re.ss:8080


That’s All.
