1. Download nexus wget  https://download.sonatype.com/nexus/3/nexus-3.68.0-04-java11-unix.tar.gz([ASC](https://download.sonatype.com/nexus/3/nexus-3.68.0-04-java11-unix.tar.gz.asc),[MD5](https://download.sonatype.com/nexus/3/nexus-3.68.0-04-java11-unix.tar.gz.md5),[SHA1](https://download.sonatype.com/nexus/3/nexus-3.68.0-04-java11-unix.tar.gz.sha1), [SHA256](https://download.sonatype.com/nexus/3/nexus-3.68.0-04-java11-unix.tar.gz.sha256), [SHA512](https://download.sonatype.com/nexus/3/nexus-3.68.0-04-java11-unix.tar.gz.sha512))
—> untar the package in /opt directory tar -xvzf name -d /opt 
2. create a user and give all permission to that user for nexus directory 
3. Go to  vi /opt/nexus:version/bin/nexus.rc 
4. change the user you have created with the default one
5. Start the nexus /opt/nexus:version/bin/nexus start 
6. It runs on port 8081
7. Take the password from cat /opt/sonatype/nexus3/admin.password
8. To create a Repo goto setting —> repository —> choose type —> create 
9. To upload artifact configured pom.xml  under the project block give id and url of the remote repo for snapshot and releases 
10. if you want the setting for whole project you can change it in setting.xml

# Integrate with jenkins

1. install config file provider plugin which store the setting of maven in that you have to mention id and password of your nexus server 
2. go to manage jenkins —> manage files —> add new config —>select maven global —> go to content --> go to server --> give id and password of nexus 
3. if you want upload snapshot change the version in pom.xml with the jenkins build number
4. if you want upload release artifact configure version as release 
5. Delete a artifact after a scheduled time with  help of clean up policy create a task and configured a cron job for deletion of artifact.
6. Take backup of nexus by copying the /opt/sonatype-nexus directory
7. For downloading the dependecy from central repo of maven with the help of nexus create a proxy repo and configured the  url of central of maven in it . 
8. Go to /.m2/setting.xml —> go to mirrors —> give id and url of proxy repo you have created 
9. If your using jenkins ensures that setting.xml has configured 

# **Configure Nexus to run as a service**

sudo vi /etc/systemd/system/nexus.service

Copy the below content.

```[Unit]

Description=nexus service

After=network.target

[Service]

Type=forking

LimitNOFILE=65536

User=nexus

Group=nexus

ExecStart=/opt/nexus/bin/nexus start

ExecStop=/opt/nexus/bin/nexus stop

User=nexus

Restart=on-abort

[Install]

WantedBy=multi-user.target

**Now Start Nexus**

sudo systemctl enable nexus

sudo systemctl start nexus

sudo systemctl status nexus
```
if it says stopped, review the steps above and you can troubleshoot by looking into Nexus logs by executing below command:
tail -f /opt/sonatype-work/nexus3/log/nexus.log

# Lets take an example

**Setting up Nexus**

1. **Download and Install Nexus:**
    - Use wget to download Nexus:
        
        ```bash
        wget https://download.sonatype.com/nexus/3/nexus-3.68.0-04-java11-unix.tar.gz
        ```
        
    - Verify the download using the SHA1 checksum file.
    - Untar the downloaded package in the /opt directory:
        
        ```bash
        tar -xvzf nexus-3.68.0-04-java11-unix.tar.gz -C /opt
        ```
        
2. **Create a Dedicated User for Nexus:**
    - Create a user to run Nexus:
        
        ```bash
        sudo useradd -r -m -U -d /opt/nexus nexus
        ```
        
    - Grant ownership of the Nexus directory to the newly created user:
        
        ```bash
        sudo chown -R nexus:nexus /opt/nexus-3.68.0-04
        ```
        
3. **Configure Nexus Startup Script:**
    - Open the nexus.rc file:
        
        ```bash
        sudo vi /opt/nexus-3.68.0-04/bin/nexus.rc
        ```
        
    - Change the **`run_as_user`** parameter to the newly created user.
4. **Start Nexus:**
    - Start Nexus using the provided script:
        
        ```bash
        sudo /opt/nexus-3.68.0-04/bin/nexus start
        ```
        
5. **Access Nexus:**
    - Nexus runs on port 8081 by default.
    - Access Nexus through your browser at **`http://your-server-ip:8081`**.
6. **Retrieve Admin Password:**
    - Obtain the admin password:
        
        ```bash
        cat /opt/sonatype/nexus3/admin.password
        ```
        
7. **Create Repositories:**
    - Navigate to "Settings" > "Repositories" and choose the repository type (e.g., hosted, proxy, group).
    - Follow the prompts to configure the repository according to your requirements.
    - Example **`pom.xml`** Configuration:

```xml
<project>
    ...
    <distributionManagement>
        <repository>
            <id>nexus-releases</id>
            <url>http://your-nexus-url/repository/maven-releases/</url>
        </repository>
        <snapshotRepository>
            <id>nexus-snapshots</id>
            <url>http://your-nexus-url/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
    ...
</project>
```

Replace **`your-nexus-url`** with the URL of your Nexus Repository Manager instance.

**Integration with Jenkins**

1. **Install Config File Provider Plugin:**
    - Navigate to "Manage Jenkins" > "Manage Plugins" > "Available" tab.
    - Search for "Config File Provider" and install the plugin.
2. **Configure Maven Settings:**
    - Go to "Manage Jenkins" > "Managed files" > "Add a new Config".
    - Select "Maven Global Settings" and provide your Nexus credentials.
    - Save the configuration.
    - Example **`settings.xml`** Configuration:

```xml
<settings>
    ...
    <servers>
        <server>
            <id>nexus-releases</id>
            <username>your-nexus-username</username>
            <password>your-nexus-password</password>
        </server>
        <server>
            <id>nexus-snapshots</id>
            <username>your-nexus-username</username>
            <password>your-nexus-password</password>
        </server>
    </servers>
    ...
    <mirrors>
        <mirror>
            <id>nexus</id>
            <mirrorOf>*</mirrorOf>
            <url>http://your-nexus-url/repository/maven-public/</url>
        </mirror>
    </mirrors>
    ...
</settings>
```

Replace **`your-nexus-url`**, **`your-nexus-username`**, and **`your-nexus-password`** with your Nexus Repository Manager URL, username, and password respectively.

1. **Upload Artifacts with Jenkins:**
    - Update your project's pom.xml to include the repository details for snapshot and release versions.
    - Use Jenkins variables (e.g., **`${BUILD_NUMBER}`**) to dynamically set version numbers.
2. **Artifact Cleanup:**
    - Configure cleanup policies in Nexus to delete artifacts after a scheduled time.
    - Create a task in Nexus and set up a cron job for automated cleanup.
3. **Backup Nexus:**
    - Take regular backups by copying the **`/opt/sonatype/nexus3`** directory to a secure location.

By following these steps, you'll set up Nexus Repository Manager and integrate it with Jenkins, ensuring efficient artifact management and reliable builds within your CI/CD pipeline.

# **Configure Nexus to Run as a Service:**

1. Create a systemd service file for Nexus:
    
    ```bash
    sudo vi /etc/systemd/system/nexus.service
    ```
    
2. Copy and paste the following content into the service file:
    
    ```
    [Unit]
    Description=Nexus service
    After=network.target
    
    [Service]
    Type=forking
    LimitNOFILE=65536
    User=nexus
    Group=nexus
    ExecStart=/opt/nexus/bin/nexus start
    ExecStop=/opt/nexus/bin/nexus stop
    Restart=on-abort
    
    [Install]
    WantedBy=multi-user.target
    ```
    
3. Save the file and exit the editor.

**Start Nexus Service:**

1. Enable the Nexus service to start on boot:
    
    ```bash
    sudo systemctl enable nexus
    ```
    
2. Start the Nexus service:
    
    ```sql
    sudo systemctl start nexus
    ```
    
3. Check the status of the Nexus service to ensure it's running:
    
    ```lua
    sudo systemctl status nexus
    ```
    
    If it says stopped, review the steps above and troubleshoot by looking into Nexus logs:
    
    ```bash
    tail -f /opt/sonatype-work/nexus3/log/nexus.log
    ```
    

By configuring Nexus to run as a service, it will automatically start and stop with the system, providing seamless management and ensuring continuous availability of your Nexus Repository Manager instance.

# change port of nexus with the help of this file

/opt/nexus/etc/nexus.properties default port is 8081

for furher help 

[https://www.coachdevops.com/search/label/Nexus 3](https://www.coachdevops.com/search/label/Nexus%203)
