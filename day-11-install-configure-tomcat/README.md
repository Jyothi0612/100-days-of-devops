# Day 11 - Install and Configure Apache Tomcat

## What was the task?

The goal of Day 11 was to install Apache Tomcat, configure it to use a custom port, deploy a Java web application, and verify that the application was actually available.

I used an Ubuntu 26.04 EC2 instance for this lab.

The overall flow was:

```text
Check Java
    ↓
Install Tomcat
    ↓
Verify service
    ↓
Verify listening port
    ↓
Deploy JSP application
    ↓
Change Tomcat port
    ↓
Restart and verify
    ↓
Create WAR file
    ↓
Deploy ROOT.war
    ↓
Verify application
```

---

## What is Apache Tomcat?

Apache Tomcat is a Java web application server.

A simple request flow looks like:

```text
Browser
   ↓
Tomcat
   ↓
Java web application
   ↓
Response
```

Tomcat can run applications built using technologies such as:

```text
Servlets
JSP
Java web applications
WAR files
```

---

# Checking Java

Tomcat requires Java.

I checked the installed Java version using:

```bash
java --version
```

The output showed:

```text
openjdk 25.0.4 2026-07-21
OpenJDK Runtime Environment (build 25.0.4+7-1-26.04-Ubuntu)
OpenJDK 64-Bit Server VM (build 25.0.4+7-1-26.04-Ubuntu, mixed mode, sharing)
```

So Java was already installed.

```text
Java available ✅
```

---

# Checking Whether Tomcat Was Installed

I ran:

```bash
dpkg -l | grep -i tomcat
```

There was no output.

That meant Tomcat was not installed yet.

---

# Updating Package Information

I ran:

```bash
apt update
```

This refreshed Ubuntu's package information before installing Tomcat.

---

# Installing Tomcat

I installed Tomcat using:

```bash
apt install tomcat10 -y
```

After installation, I checked the service:

```bash
systemctl status tomcat10 --no-pager
```

The important output was:

```text
Active: active (running)
```

The logs also showed:

```text
Starting Servlet engine: [Apache Tomcat/10.1.55 (Ubuntu)]
```

and:

```text
Starting ProtocolHandler ["http-nio-8080"]
```

and:

```text
Server startup
```

So Tomcat started successfully.

---

# Verifying Port 8080

I did not want to rely only on `systemctl`.

I checked whether something was actually listening on port 8080:

```bash
ss -ltnp | grep 8080
```

The output showed:

```text
LISTEN 0 100 *:8080 *:* users:(("java",pid=44485,fd=37))
```

This confirmed:

```text
Tomcat service running ✅
Java process running    ✅
Port 8080 listening     ✅
```

---

# Testing Tomcat with curl

I tested Tomcat locally:

```bash
curl http://localhost:8080
```

Tomcat returned a web page.

This was another important verification step.

```text
systemctl
→ Is the service running?

ss
→ Is the port listening?

curl
→ Is the application actually responding?
```

---

# Understanding Tomcat webapps

I checked Tomcat's deployment directory:

```bash
ls -l /var/lib/tomcat10/webapps
```

The server initially contained:

```text
ROOT
```

The directory:

```text
/var/lib/tomcat10/webapps/ROOT
```

is special.

Tomcat serves the ROOT application directly from:

```text
http://server:port/
```

Other application directories use their directory name in the URL.

For example:

```text
webapps/devops
```

becomes:

```text
http://server:port/devops/
```

---

# Creating a Custom Tomcat Application

I created:

```bash
mkdir -p /var/lib/tomcat10/webapps/devops
```

Then I created:

```text
/var/lib/tomcat10/webapps/devops/index.jsp
```

with:

```jsp
<%@ page contentType="text/html;charset=UTF-8" %>
<html>
<head>
    <title>DevOps Day 11</title>
</head>
<body>
    <h1>Hello from Tomcat</h1>
    <p>Day 11 Java web application is working.</p>
</body>
</html>
```

I tested it using:

```bash
curl http://localhost:8080/devops/
```

The output contained:

```html
<h1>Hello from Tomcat</h1>
<p>Day 11 Java web application is working.</p>
```

This confirmed the JSP application was deployed successfully.

---

# Checking JSP Permissions

I checked:

```bash
ls -l /var/lib/tomcat10/webapps/devops
```

The output was:

```text
-rw-r--r-- 1 root root 210 Sep 11 20:09 index.jsp
```

This means:

```text
Owner  → root
Group  → root

Owner  → read + write
Group  → read
Others → read
```

Tomcat could serve the file because it had permission to read it.

---

# Finding the Tomcat Port Configuration

Tomcat was initially using:

```text
8080
```

I searched the configuration:

```bash
grep -n '8080' /etc/tomcat10/server.xml
```

The output showed:

```text
68: Define a non-SSL/TLS HTTP/1.1 Connector on port 8080
70: <Connector port="8080" protocol="HTTP/1.1"
78: port="8080" protocol="HTTP/1.1"
```

Before editing anything, I inspected the surrounding lines:

```bash
sed -n '64,82p' /etc/tomcat10/server.xml
```

This showed that the active connector was:

```xml
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443"
           maxParameterCount="1000"
           />
```

Another connector containing `8080` was inside a comment.

This taught me an important lesson:

```text
Find matching text
      ↓
Inspect context
      ↓
Understand active configuration
      ↓
Then edit
```

I should not blindly replace every occurrence of a value.

---

# Backing Up the Configuration

Before changing `server.xml`, I created a backup:

```bash
cp /etc/tomcat10/server.xml /etc/tomcat10/server.xml.bak
```

This gave me a rollback copy if something went wrong.

---

# Changing Tomcat from Port 8080 to 3003

I changed the active HTTP connector:

```bash
sed -i 's/<Connector port="8080" protocol="HTTP\/1.1"/<Connector port="3003" protocol="HTTP\/1.1"/' /etc/tomcat10/server.xml
```

Then I verified it:

```bash
grep -n '<Connector port=' /etc/tomcat10/server.xml
```

The output showed:

```text
70: <Connector port="3003" protocol="HTTP/1.1"
92: <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
```

So the normal HTTP connector had been changed to:

```text
3003
```

while the HTTPS-related connector remained:

```text
8443
```

---

# Restarting Tomcat

Configuration changes do not automatically affect the already-running process.

I restarted Tomcat:

```bash
systemctl restart tomcat10
```

Then verified:

```bash
systemctl status tomcat10 --no-pager
```

The service showed:

```text
Active: active (running)
```

---

# Verifying the New Port

I checked:

```bash
ss -ltnp | grep 3003
```

The output was:

```text
LISTEN 0 100 *:3003 *:* users:(("java",pid=44654,fd=37))
```

This proved Tomcat had moved from:

```text
8080
```

to:

```text
3003
```

---

# Testing the Application on Port 3003

I tested:

```bash
curl http://localhost:3003/devops/
```

The application still worked.

So changing the connector port did not break the deployed application.

The new flow was:

```text
Client
   ↓
Port 3003
   ↓
Tomcat
   ↓
devops JSP application
```

---

# Testing the ROOT Application

I also ran:

```bash
curl http://localhost:3003/
```

The default Tomcat page returned:

```text
It works !
```

The page also showed that the default ROOT application was located at:

```text
/var/lib/tomcat10/webapps/ROOT/index.html
```

So I learned:

```text
webapps/ROOT
→ /

webapps/devops
→ /devops/
```

---

# Checking the Exact Tomcat Version

I ran:

```bash
/usr/share/tomcat10/bin/version.sh
```

The important information was:

```text
Server version: Apache Tomcat/10.1.55 (Ubuntu)
Server number:  10.1.55.0
OS Name:        Linux
OS Version:     7.0.0-1006-aws
Architecture:   amd64
JVM Version:    25.0.4+7-1-26.04-Ubuntu
JVM Vendor:     Ubuntu
```

So my environment was:

```text
Tomcat → 10.1.55
Java   → 25.0.4
Linux  → AWS kernel
Arch    → amd64
```

---

# Understanding the Warning from version.sh

The version command also displayed warnings related to Java native access and OpenSSL library detection.

Instead of assuming Tomcat was broken, I checked the actual service logs.

This was important because:

```text
Warning in a command
        ≠
Running service is broken
```

---

# Checking Tomcat Logs

I ran:

```bash
journalctl -u tomcat10 -n 20 --no-pager
```

The logs showed:

```text
Loaded Apache Tomcat Native library [2.0.14]
OpenSSL successfully initialized
Initializing ProtocolHandler ["http-nio-3003"]
Starting service [Catalina]
Starting Servlet engine: [Apache Tomcat/10.1.55 (Ubuntu)]
Deploying web application directory [/var/lib/tomcat10/webapps/devops]
Deploying web application directory [/var/lib/tomcat10/webapps/ROOT]
Starting ProtocolHandler ["http-nio-3003"]
Server startup in [2241] milliseconds
```

This confirmed that the actual Tomcat service was healthy.

The warnings did not stop the server from starting.

---

# Creating a WAR Application

I also learned how Tomcat applications can be deployed using WAR files.

WAR means:

```text
Web Application Archive
```

It is an archive containing a Java web application.

I created a temporary application directory:

```bash
mkdir -p /tmp/rootapp
```

Then created:

```text
/tmp/rootapp/index.jsp
```

with:

```jsp
<%@ page contentType="text/html;charset=UTF-8" %>
<html>
<head>
    <title>DevOps Day 11</title>
</head>
<body>
    <h1>Hello from ROOT WAR</h1>
    <p>Tomcat WAR deployment is working on port 3003.</p>
</body>
</html>
```

---

# Missing jar Command

I initially tried:

```bash
cd /tmp/rootapp && jar -cvf /tmp/ROOT.war .
```

but received:

```text
Command 'jar' not found
```

This taught me an important difference:

```text
Java runtime
→ enough to run Java applications

JDK
→ development tools such as jar and javac
```

Tomcat could run because Java was installed, but the `jar` utility was not available.

---

# Installing the JDK Tools

I installed:

```bash
apt install openjdk-25-jdk-headless -y
```

I used the headless JDK because this is a server and no graphical desktop tools are needed.

Then I verified:

```bash
jar --version
```

The `jar` command was now available.

---

# Creating ROOT.war

I ran:

```bash
cd /tmp/rootapp && jar -cvf /tmp/ROOT.war .
```

This created:

```text
/tmp/ROOT.war
```

I verified its contents using:

```bash
jar -tf /tmp/ROOT.war
```

The WAR contained the application files including:

```text
META-INF/
META-INF/MANIFEST.MF
index.jsp
```

---

# Deploying ROOT.war

Before replacing the existing ROOT application, I stopped Tomcat:

```bash
systemctl stop tomcat10
```

Then I kept a backup of the original ROOT directory:

```bash
mv /var/lib/tomcat10/webapps/ROOT /var/lib/tomcat10/webapps/ROOT.bak
```

This meant I still had a rollback copy.

Then I copied the WAR:

```bash
cp /tmp/ROOT.war /var/lib/tomcat10/webapps/
```

The deployment file became:

```text
/var/lib/tomcat10/webapps/ROOT.war
```

---

# Starting Tomcat After WAR Deployment

I started Tomcat again:

```bash
systemctl start tomcat10
```

Then verified:

```bash
systemctl status tomcat10 --no-pager
```

Tomcat returned:

```text
Active: active (running)
```

---

# Testing ROOT.war

I tested:

```bash
curl http://localhost:3003/
```

The new root application worked.

Instead of the original Tomcat homepage, the application returned the page from my WAR.

The deployment flow was:

```text
ROOT.war
   ↓
Tomcat detects WAR
   ↓
Tomcat deploys application
   ↓
Application becomes ROOT context
   ↓
http://localhost:3003/
```

---

# What I Learned About ROOT.war

The WAR filename affects the application URL.

For example:

```text
devops.war
→ /devops/
```

while:

```text
ROOT.war
→ /
```

So:

```text
ROOT
```

has special meaning in Tomcat.

It becomes the application served directly from the base URL.

---

# Commands Used

Check Java:

```bash
java --version
```

Check Tomcat packages:

```bash
dpkg -l | grep -i tomcat
```

Update packages:

```bash
apt update
```

Install Tomcat:

```bash
apt install tomcat10 -y
```

Check service:

```bash
systemctl status tomcat10 --no-pager
```

Check port:

```bash
ss -ltnp | grep 8080
```

Test Tomcat:

```bash
curl http://localhost:8080
```

Check webapps:

```bash
ls -l /var/lib/tomcat10/webapps
```

Create application:

```bash
mkdir -p /var/lib/tomcat10/webapps/devops
```

Check application permissions:

```bash
ls -l /var/lib/tomcat10/webapps/devops
```

Find port configuration:

```bash
grep -n '8080' /etc/tomcat10/server.xml
```

Inspect configuration:

```bash
sed -n '64,82p' /etc/tomcat10/server.xml
```

Back up configuration:

```bash
cp /etc/tomcat10/server.xml /etc/tomcat10/server.xml.bak
```

Change port:

```bash
sed -i 's/<Connector port="8080" protocol="HTTP\/1.1"/<Connector port="3003" protocol="HTTP\/1.1"/' /etc/tomcat10/server.xml
```

Verify connectors:

```bash
grep -n '<Connector port=' /etc/tomcat10/server.xml
```

Restart Tomcat:

```bash
systemctl restart tomcat10
```

Check new port:

```bash
ss -ltnp | grep 3003
```

Test applications:

```bash
curl http://localhost:3003/
curl http://localhost:3003/devops/
```

Check Tomcat version:

```bash
/usr/share/tomcat10/bin/version.sh
```

Check logs:

```bash
journalctl -u tomcat10 -n 20 --no-pager
```

Create temporary WAR directory:

```bash
mkdir -p /tmp/rootapp
```

Install JDK tools:

```bash
apt install openjdk-25-jdk-headless -y
```

Check jar:

```bash
jar --version
```

Create WAR:

```bash
cd /tmp/rootapp && jar -cvf /tmp/ROOT.war .
```

Inspect WAR:

```bash
jar -tf /tmp/ROOT.war
```

Stop Tomcat:

```bash
systemctl stop tomcat10
```

Back up ROOT:

```bash
mv /var/lib/tomcat10/webapps/ROOT /var/lib/tomcat10/webapps/ROOT.bak
```

Deploy WAR:

```bash
cp /tmp/ROOT.war /var/lib/tomcat10/webapps/
```

Start Tomcat:

```bash
systemctl start tomcat10
```

---

# What I Learned

Day 11 taught me that installing a service is only the beginning.

I need to verify several layers:

```text
Package installed
      ↓
Service running
      ↓
Port listening
      ↓
Application deployed
      ↓
HTTP request succeeds
```

I also learned how Tomcat maps applications:

```text
webapps/ROOT
→ /

webapps/devops
→ /devops/
```

and how WAR deployment works:

```text
Application files
      ↓
Package into WAR
      ↓
Copy WAR into webapps
      ↓
Tomcat deploys it
      ↓
Application becomes available
```

---

# Troubleshooting Lessons

This lab gave me several useful troubleshooting examples.

### Service says running

I still verified the port:

```bash
ss -ltnp
```

### Port is listening

I still verified HTTP:

```bash
curl
```

### Warning appears

I checked whether it actually affected the running service.

### `jar` command missing

I did not reinstall Tomcat.

I identified that the missing tool belonged to the JDK and installed the required package.

This follows the troubleshooting mindset I learned earlier:

```text
Observe
   ↓
Understand the error
   ↓
Find the root cause
   ↓
Fix only that problem
   ↓
Verify
```

---

# Main Takeaway

The biggest lesson from Day 11 was:

```text
Running service
≠
Working application
```

A better verification process is:

```text
Install
   ↓
Check service
   ↓
Check port
   ↓
Check logs
   ↓
Send HTTP request
   ↓
Verify deployed application
```

I also learned the basic Tomcat deployment model:

```text
Java
  +
Tomcat
  +
server.xml
  +
WAR/JSP application
  ↓
Java web application
```

Day 11 helped me understand how a Java application moves from files on a Linux server to something that can actually respond to HTTP requests.