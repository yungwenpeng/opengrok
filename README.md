# OpenGrok - a wicked fast source browser
OpenGrok(https://github.com/oracle/opengrok/tree/master) is a fast and usable source code search and cross reference engine, written in Java. It helps you search, cross-reference and navigate your source tree. It can understand various program file formats and version control histories of many source code management systems.

- [1. Install Java OpenJDK](#1-install-java-openjdk)
- [2. Install Universal Ctags](#2-install-universal-ctags)
- [3. Install Tomcat 10](#3-install-tomcat-10)
- [4. Install OpenGrok](#3-install-opengrok)

## 1. Install Java OpenJDK
```sh
sudo apt install openjdk-11-jdk -y
sudo update-alternatives --config java
```

## 2. Install Universal Ctags
```sh
sudo apt install -y autoconf automake
git clone https://github.com/universal-ctags/ctags.git
cd ctags/
./autogen.sh
./configure
make
sudo make install
```

## 3. Install Tomcat 10
See [How to Install Tomcat 10 on Ubuntu 22.04](https://www.atlantic.net/dedicated-server-hosting/how-to-install-tomcat-10-on-ubuntu-22-04/)
1. Download Apache Tomcat
    ```sh
    wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.39/bin/apache-tomcat-10.1.39.tar.gz
    tar xzf apache-tomcat-10.1.39.tar.gz
    sudo mkdir -p /opt/tomcat10
    sudo mv apache-tomcat-10.1.15/* /opt/tomcat10/
    sudo chown -R tomcat: /opt/tomcat10
    sudo sh -c 'chmod +x /opt/tomcat10/bin/*.sh'
    sudo useradd -r -m -U -d /opt/tomcat10 -s /bin/false tomcat
    ```
2. Create a Tomcat User Account
    ```sh
    sudo nano /opt/tomcat10/conf/tomcat-users.xml
    ```
    Add the following lines above the last line.
    ```sh
    <role rolename="manager-gui" />
    <user username="manager" password="secure-password" roles="manager-gui" />
    <role rolename="admin-gui" />
    <user username="admin" password="secure-password" roles="manager-gui,admin-gui" />
    ```
3. Allow Remote Hosts to Access Tomcat
    ```sh
    sudo nano /opt/tomcat/webapps/manager/META-INF/context.xml
    ```
    Remove the following line.
    ```sh
      <Valve className="org.apache.catalina.valves.RemoteAddrValve"
             allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
    ```
    ```sh
    sudo nano /opt/tomcat/webapps/host-manager/META-INF/context.xml
    ```
    Remove the following line.
    ```sh
      <Valve className="org.apache.catalina.valves.RemoteAddrValve"
             allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
    ```
4. Create a Systemd Service File for Tomcat
    ```sh
    sudo nano /etc/systemd/system/tomcat10.service
    ```
    Add the following lines.
    ```sh
    [Unit]
    Description=Apache Tomcat 10 Web Application Server
    After=network.target
    
    [Service]
    Type=forking
    
    User=tomcat
    Group=tomcat
    
    Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64"
    Environment="CATALINA_HOME=/opt/tomcat10"
    Environment="CATALINA_BASE=/opt/tomcat10"
    Environment="CATALINA_PID=/opt/tomcat10/temp/tomcat.pid"
    Environment="CATALINA_OPTS=-Xms16G -Xmx32G -server -XX:+UseParallelGC"
    
    ExecStart=/opt/tomcat10/bin/startup.sh
    ExecStop=/opt/tomcat10/bin/shutdown.sh
    
    SuccessExitStatus=143
    PIDFile=/opt/tomcat10/temp/tomcat.pid
    
    [Install]
    WantedBy=multi-user.target
    ```
5. Modify Tomcat Homepage
    ```sh
    sudo nano /opt/tomcat10/webapps/ROOT/index.jsp
    ```
    Change the following lines.
    ```sh
    <%@ page session="false" pageEncoding="UTF-8" contentType="text/html; charset=UTF-8" %>
    <%
    java.text.SimpleDateFormat sdf = new java.text.SimpleDateFormat("yyyy");
    request.setAttribute("year", sdf.format(new java.util.Date()));
    request.setAttribute("opengrokUrl", "http://localhost:8080");
    %>
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8" />
            <title><%=request.getServletContext().getServerInfo() %></title>
            <link href="favicon.ico" rel="icon" type="image/x-icon" />
            <link href="tomcat.css" rel="stylesheet" type="text/css" />
            <style>
                /* Add this CSS rule for the hover effect */
                a:hover {
                    font-size: 1.5em; /* Increase font size */
                    font-weight: bold; /* Make text bold */
                    background-color: yellow; /* Change the background color */
                    color: red; /* Change the text color */
                    text-decoration: underline; /* Add underline */
                }
            </style>
        </head>
    
        <body>
            <div id="wrapper">
                <div id="navigation" class="curved container">
                    <span id="nav-home"><a href="${opengrokUrl}">Home</a></span>
                    <br class="separator" />
                </div>
                <div id="asf-box">
                    <h1>Android Code Search</h1>
                </div>
                <div id="upper" class="curved container">
                    <div id="PROJECTA">
                        <div class="container">
                            <h4>PROJECTA</h4>
                            <ul>
                                <li><a href="${opengrokUrl}/PROJECTA" target="_blank" rel="noopener noreferrer">PROJECTA</a></li>
                            </ul>
                        </div>
                    </div>
                    <div id="PROJECTB">
                        <div class="container">
                            <h4>PROJECTB</h4>
                            <ul>
                                <li><a href="${opengrokUrl}/PROJECTB" target="_blank" rel="noopener noreferrer">PROJECTB</a></li>
                            </ul>
                        </div>
                    </div>
                    <br class="separator" />
                </div>
                <p class="copyright">Copyright &copy;1999-${year} OpenGrok Software Foundation.  All Rights Reserved</p>
            </div>
        </body>
    
    </html>
    ```

6. Reload the systemd daemon to apply the changes.
    ```sh
    sudo systemctl daemon-reload
    sudo systemctl restart tomcat10.service
    sudo systemctl status tomcat10.service
    ```

## 4. Using OpenGrok authorization
See [Authorization](https://github.com/oracle/opengrok/wiki/Authorization)
Example for LDAP:
```sh
sudo nano /opt/tomcat10/conf/server.xml
```
Change the following lines.
```sh
        <!--        <Realm className="org.apache.catalina.realm.UserDatabaseRealm“
               resourceName="UserDatabase"/>
        -->
        <Realm className="org.apache.catalina.realm.JNDIRealm“
             connectionURL="ldap://localhost:389“
             userBase="ou=users,dc=example,dc=com“
             userSearch="(uid={0})“
             userSubtree="true“
             realm-name="OpenGrok LDAP"/>
```
Try to search LDAP user
```sh
ldapsearch -x -H ldap://localhost:389 -D "cn=admin,dc=example,dc=com" -W -b "ou=users,dc=example,dc=com" "(uid=user1)"
```

## 5. Install OpenGrok
See [How to setup OpenGrok](https://github.com/oracle/opengrok/wiki/How-to-setup-OpenGrok)
Downloading the distribution tar ball and setting up directory structure:
```sh
mkdir -p <workspace>/opengrok/{src,data,dist,etc,log}

cd <workspace>/opengrok
wget https://github.com/oracle/opengrok/releases/download/1.13.25/opengrok-1.13.25.tar.gz
tar -C dist --strip-components=1 -xzf opengrok-1.13.25.tar.gz

cd <workspace>
sudo cp <workspace>/opengrok/dist/lib/source.war  /opt/tomcat10/webapps/
```
1. Add insert.xml for LDAP authorization
    ```sh
    nano <workspace>/opengrok/insert.xml
    ```
    Add the following lines.
    ```sh
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app>
        <login-config>
            <auth-method>BASIC</auth-method>
            <realm-name>OpenGrok LDAP</realm-name>
        </login-config>
        <security-constraint>
            <web-resource-collection>
                <web-resource-name>API endpoints are checked separately by the web app</web-resource-name>
                    <url-pattern>/api/*</url-pattern>
            </web-resource-collection>
        </security-constraint>
        <security-constraint>
            <web-resource-collection>
                <web-resource-name>Protected Area</web-resource-name>
                <url-pattern>/*</url-pattern>
            </web-resource-collection>
            <auth-constraint>
                <role-name>*</role-name>
            </auth-constraint>
        </security-constraint>
        <security-role>
            <role-name>*</role-name>
        </security-role>
    </web-app>
    ```

2. Logging
    ```sh
    cp <workspace>/opengrok/dist/doc/logging.properties etc/
    nano etc/logging.properties 
    -java.util.logging.FileHandler.pattern = opengrok%g.%u.log
    +java.util.logging.FileHandler.pattern = <workspace>/log/opengrok%g.%u.log
    ```

3. Install management tools
    ```sh
    sudo apt install python3-venv
    cd <workspace>/opengrok/dist/tools
    python3 -m venv .venv
    . ./.venv/bin/activate
    pip3 install opengrok-tools.tar.gz
    ```

4. Setting up the sources / input data
    ```sh
    cd <workspace>/opengrok/src
    git clone https://github.com/githubtraining/hellogitworld.git PROJECTA/hellogitworld
    git clone https://github.com/OpenGrok/OpenGrok PROJECTA/OpenGrok
    ```

5. Deploy the web application
    ```sh
    cd <workspace>/opengrok/dist/tools/.venv/bin
    sudo ./opengrok-deploy --insert <workspace>/opengrok/insert.xml -c <workspace>/opengrok/etc/configuration.xml <workspace>/opengrok/dist/lib/source.war /opt/tomcat10/webapps
    ```

6. Indexing
  - For multiple project:
    ```sh
    cd <workspace>
    sudo cp -r /opt/tomcat10/webapps/source /opt/tomcat10/webapps/PROJECTA
    sudo nano /opt/tomcat10/webapps/PROJECTA/WEB-INF/web.xml
            <context-param>
                <description>Full path to the configuration file where OpenGrok can read its configuration</description>
                <param-name>CONFIGURATION</param-name>
                <param-value><workspace>/opengrok/etc/PROJECTA_configuration.xml</param-value>
            </context-param>
    sudo chown tomcat: -R /opt/tomcat10/webapps
    ```
    create index:
    ```sh
    cd <workspace>/opengrok/dist/tools/.venv/bin
    ./opengrok-indexer -J=-Djava.util.logging.config.file=<workspace>/opengrok/etc/logging.properties -J=-DALLOW_SYMBOLIC_LINKS=true -a <workspace>/opengrok/dist/lib/opengrok.jar -- -c /usr/local/bin/ctags -s <workspace>/opengrok/src/PROJECTA -d <workspace>/opengrok/data -P -S -G -W <workspace>/opengrok/etc/PROJECTA_configuration.xml -U http://localhost:8080/PROJECTA --progres
    ```
