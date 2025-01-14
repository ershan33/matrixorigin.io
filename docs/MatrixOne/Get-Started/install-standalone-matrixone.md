# **Deploy standalone MatrixOne**

The applicable scenario of the standalone MatrixOne is to use a single server, experience the smallest MatrixOne with the complete topology, and simulate the production deployment steps.

**The main steps of standalone MatrixOne deployment are as follows**:

<a href="#install_mo">Step 1: Install standalone MatrixOne</a><br>
<a href="#connect_mo">Step 2: Connect to standalone MatrixOne</a>

**Recommended operating system**

MatrixOne supports **Linux** and **MacOS**. For quick start, we recommend the following hardware specifications:

| Operating System    | Operating System Version | CPU   |Memory|
| :------ | :----- | :-------------- |  :------|
|CentOS| 7.3 and later| x86 CPU; 4 Cores | 32 GB |
|macOS| Monterey 12.3 and later | - x86 CPU；4 Cores<br>- ARM；4 Cores | 32 GB |

For more information on the required operating system versions for deploying MatrixOne, see [Hardware and Operating system requirements](../FAQs/deployment-faqs.md).

## <h2><a name="install_mo">Step 1: Install standalone MatrixOne</a></h2>

To facilitate developers or technology enthusiasts with different operating habits to install the MatrixOne standalone version most conveniently and quickly, we provide the following three installation methods. You can choose the most suitable installation method for you according to your needs:

- <p><a href="#code_source">Method 1: Building from source code</a>.</p> If you always need to get the latest version code, you can choose to install and deploy MatrixOne by **Building from source code**.
- <p><a href="#binary_packages">Method 2: Using binary package</a>.</p> If you like installing the package directly, you can choose to install and deploy MatrixOne by **Using binary package**.
- <p><a href="#use_docker">Method 3: Using Docker</a>.</p> If you usually use Docker, you can also install and deploy MatrixOne by **Using Docker**.

### <h3><a name="code_source">Method 1: Building from source code</a></h3>

#### 1. Install Go as necessary dependency

!!! note
    Go version 1.19 is required.

Click <a href="https://go.dev/doc/install" target="_blank">Go Download and install</a> to enter its official documentation, and follow the installation steps to complete the **Go** installation.

To verify whether **Go** is installed, please execute the code `go version`. When **Go** is installed successfully, the example code line is as follows:  

=== "**Linux Environment**"

     The example code line is as follows:  

     ```
     go version go1.19.3 linux/amd64
     ```

=== "**MacOS Environment**"

      The example code line is as follows:

      ```
      go version go1.19 darwin/arm64
      ```

#### 2. Build MatrixOne by using MatrixOne code

Depending on your needs, choose whether you want to keep your code up to date, or if you want to get the latest stable version of the code.

__Tips__: When using MatrixOne source code to build MatrixOne, there is no much difference between the source code for **Linux environment** and **MacOS environment**. This chapter focuses on building Matrixone source code for different versions.

=== "Get the MatrixOne(Develop Version) code to build"

     The **main** branch is the default branch, the code on the main branch is always up-to-date but not stable enough.

     1. Get the MatrixOne(Develop Version) code:

         ```
         git clone https://github.com/matrixorigin/matrixone.git
         cd matrixone
         ```

     2. Run `make build` to compile the MatrixOne file:

         ```
         make build
         ```

         __Tips__: You can also run `make debug`, `make clean`, or anything else our `Makefile` offers, `make debug` can be used to debug the build process, and `make clean` can be used to clean up the build process. If you get an error like `Get "https://proxy.golang.org/........": dial tcp 142.251.43.17:443: i/o timeout` while running `make build`, see [Deployment FAQs](../FAQs/deployment-faqs.md).

=== "Get the MatrixOne(Stable Version) code to build"

     1. If you want to get the latest stable version code released by MatrixOne, please switch to the branch of version **0.6.0** first.

         ```
         git clone https://github.com/matrixorigin/matrixone.git
         git checkout 0.6.0
         cd matrixone
         ```

     2. Run `make config` and `make build` to compile the MatrixOne file:

         ```
         make config
         make build
         ```

         __Tips__: You can also run `make debug`, `make clean`, or anything else our `Makefile` offers, `make debug` can be used to debug the build process, and `make clean` can be used to clean up the build process. If you get an error like `Get "https://proxy.golang.org/........": dial tcp 142.251.43.17:443: i/o timeout` while running `make build`, see [Deployment FAQs](../FAQs/deployment-faqs.md).

#### <h4><a name="launch">3. Launch MatrixOne server</a></h4>

=== "**Launch in the frontend**"

      This launch method will keep the `mo-service` process running in the frontend, the system log will be printed in real time. If you'd like to stop MatrixOne server, just make a CTRL+C or close your current terminal.

      ```
      # Start mo-service in the frontend
      ./mo-service -launch ./etc/quickstart/launch.toml
      ```

=== "**Launch in the backend**"

      This launch method will put the `mo-service` process running in the backend, the system log will be redirected to the `test.log` file. If you'd like to stop MatrixOne server, you need to find out its `PID` by and kill it by the following commands. Below is a full example of the whole process.

      ```
      # Start mo-service in the backend
      nohup ./mo-service -launch ./etc/quickstart/launch.toml &> test.log &

      # Find mo-service PID
      ps aux | grep mo-service

      [root@VM-0-10-centos ~]# ps aux | grep mo-service
      root       15277  2.8 16.6 8870276 5338016 ?     Sl   Nov25 156:59 ./mo-service -launch ./etc/quickstart/launch.toml
      root      836740  0.0  0.0  12136  1040 pts/0    S+   10:39   0:00 grep --color=auto mo-service

      # Kill the mo-service process
      kill -9 15277
      ```

      __Tips__: As shown in the above example, use the command `ps aux | grep mo-service` to find out that the process number running on MatrixOne is `15277`, and `kill -9 15277` means to stop MatrixOne with the process number `15277`.

#### 4. Connect to MatrixOne Server

When you finish installing and launching MatrixOne, many logs are generated in startup mode. Then you can start a new terminal and connect to MatrixOne, for more information on connecting to MatrixOne, see <a href="#connect_mo">Step 2: Connect to standalone MatrixOne</a>.

### <h3><a name="binary_packages">Method 2: Downloading binary packages</a></h3>

From 0.3.0 release, you can download binary packages.

#### 1. Install `wget` or `curl`

We'll provide a method of **Using binary package** to install MatrixOne. If you prefer to use the command line, you can pre-install `wget` or `curl`.

__Tips__: It is recommended that you download and install one of these two tools to facilitate future operations.

=== "Install `wget`"

     The `wget` tool is used to download files from the specified URL. `wget` is a unique file download tool; it is very stable and has a download speed.

     Go to the <a href="https://brew.sh/" target="_blank">Homebrew</a> page and follow the instructions to install **Homebrew** first and then install `wget`.  To verify that `wget` is installed successfully, use the following command line:

     ```
     wget -V
     ```

     The successful installation results (only part of the code is displayed) are as follows:

     - Linux environment:

     ```
     GNU Wget 1.21.3 built on linux-gnu.
     ...
     Copyright (C) 2015 Free Software Foundation, Inc.
     ...
     ```

     - MacOS environment:

     ```
     GNU Wget 1.21.3 is compiled on darwin21.3.0.
     ...
     Copyright © 2015 Free Software Foundation, Inc.
     ...
     ```

=== "Install `curl`"

     `curl` is a file transfer tool that works from the command line using URL rules. `curl` is a comprehensive transfer tool that supports file upload and download.  

     Go to the <a href="https://curl.se/download.html" target="_blank">Curl</a> website according to the official installation guide to install `curl`.  To verify that `curl` is installed successfully, use the following command line:

     ```
     curl --version
     ```

     The successful installation results (only part of the code is displayed) are as follows:

     - Linux environment:

     ```
     curl 7.84.0 (x86_64-pc-linux-gnu) libcurl/7.84.0 OpenSSL/1.1.1k-fips zlib/1.2.11
     Release-Date: 2022-06-27
     ...
     ```

     - MacOS environment:

     ```
     curl 7.84.0 (x86_64-apple-darwin22.0) libcurl/7.84.0 (SecureTransport) LibreSSL/3.3.6 zlib/1.2.11 nghttp2/1.47.0
     Release-Date: 2022-06-27
     ...
     ```

#### 2. Download binary packages and decompress

=== "**Linux Environment**"

      **Download Method 1** and **Download Method 2** need to install the download tools `wget` or `curl` first.

     + **Downloading method 1: Using `wget` to install binary packages**

        ```bash
        wget https://github.com/matrixorigin/matrixone/releases/download/v0.6.0/mo-v0.6.0-linux-amd64.zip
        unzip mo-v0.6.0-linux-amd64.zip
        ```

     + **Downloading method 2: Using `curl` to install binary packages**

        ```bash
        curl -OL https://github.com/matrixorigin/matrixone/releases/download/v0.6.0/mo-v0.6.0-linux-amd64.zip
        unzip mo-v0.6.0-linux-amd64.zip
        ```

     + **Downloading method 3: If you want a more intuitive way to download the page, you can go to the following page link and download**

        Go to the [version 0.6.0](https://github.com/matrixorigin/matrixone/releases/tag/v0.6.0), pull down to find the **Assets** column, and click the installation package *mo-v0.6.0-linux-amd64.zip* can be downloaded.

=== "**MacOS Environment**"

      **Download Method 1** and **Download Method 2** need to install the download tools `wget` or `curl` first.

     + **Downloading method 1: Using `wget` to install binary packages**

        ```bash
        wget https://github.com/matrixorigin/matrixone/releases/download/v0.6.0/mo-v0.6.0-darwin-x86_64.zip
        unzip mo-v0.6.0-darwin-x86_64.zip
        ```

     + **Downloading method 2: Using `curl` to install binary packages**

        ```bash
        curl -OL https://github.com/matrixorigin/matrixone/releases/download/v0.6.0/mo-v0.6.0-darwin-x86_64.zip
        unzip mo-v0.6.0-darwin-x86_64.zip
        ```

     + **Downloading method 3: If you want a more intuitive way to download the page, you can go to the following page link and download**

        Go to the [version 0.6.0](https://github.com/matrixorigin/matrixone/releases/tag/v0.6.0), pull down to find the **Assets** column, and click the installation package *mo-v0.6.0-darwin-x86_64.zip* can be downloaded.

!!! info
    MatrixOne only supports installation on ARM chipset with source code build; if you are using MacOS M1 and above, for more information on using source code build to install MatrixOne, see <a href="#code_source">Method 1: Building from source code</a>. Using release binary files from X86 chipset will lead to unknown problems.

#### 3. Launch MatrixOne server

Launch MatrixOne server in the frontend or backend as <a href="#launch">3. Launch MatrixOne server</a> suggests in **Building from source code**.

=== "**Launch in the frontend**"

      This launch method will keep the `mo-service` process running in the frontend, the system log will be printed in real time. If you'd like to stop MatrixOne server, just make a CTRL+C or close your current terminal.

      ```
      # Start mo-service in the frontend
      ./mo-service -launch ./etc/quickstart/launch.toml
      ```

=== "**Launch in the backend**"

      This launch method will put the `mo-service` process running in the backend, the system log will be redirected to the `test.log` file. If you'd like to stop MatrixOne server, you need to find out its `PID` by and kill it by the following commands. Below is a full example of the whole process.

      ```
      # Start mo-service in the backend
      nohup ./mo-service -launch ./etc/quickstart/launch.toml &> test.log &

      # Find mo-service PID
      ps aux | grep mo-service

      [root@VM-0-10-centos ~]# ps aux | grep mo-service
      root       15277  2.8 16.6 8870276 5338016 ?     Sl   Nov25 156:59 ./mo-service -launch ./etc/quickstart/launch.toml
      root      836740  0.0  0.0  12136  1040 pts/0    S+   10:39   0:00 grep --color=auto mo-service

      # Kill the mo-service process
      kill -9 15277
      ```

      __Tips__: As shown in the above example, use the command `ps aux | grep mo-service` to find out that the process number running on MatrixOne is `15277`, and `kill -9 15277` means to stop MatrixOne with the process number `15277`.

#### 4. Connect to MatrixOne Server

When you finish installing and launching MatrixOne, many logs are generated in startup mode. Then you can start a new terminal and connect to MatrixOne, for more information on connecting to MatrixOne, see <a href="#connect_mo">Step 2: Connect to standalone MatrixOne</a>.

### <h3><a name="use_docker">Method 3: Using docker</a></h3>

__Tips__: When using Docker to build MatrixOne, there is no much difference between the source code for **Linux environment** and **MacOS environment**.

#### 1. Download and install Docker

Click <a href="https://docs.docker.com/get-docker/" target="_blank">Get Docker</a>, enter into the Docker's official document page, depending on your operating system, download and install the corresponding Docker.  

After the installation, click to open Docker and go to the next step to verify whether the installation is successful.

#### 2. Verify that the Docker installation is successful  

You can verify the Docker version by using the following lines:

```
docker --version
```

The successful installation results are as follows:

```
Docker version 20.10.17, build 100c701
```

#### 3. Check the Docker running status  

Run the following command to start Docker and check whether the running status is successfully. The check suggestions for different operating environments are as follows:

- **Linux Environment**:

In Linux environment, you can execute the following command in your terminal:

```
systemctl start docker
systemctl status docker
```

The following code example indicates that Docker is running. `Active: active (running)` shows that Docker is running.

```
docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Sat 2022-11-26 17:48:32 CST; 6s ago
     Docs: https://docs.docker.com
 Main PID: 234496 (dockerd)
    Tasks: 8
   Memory: 23.6M
```

- **MacOS Environment**:

In MacOS environment, You can open your local Docker client and launch Docker.

#### 4. Create and run the container of MatrixOne

It will pull the image from Docker Hub if not exists. You can choose to pull the stable version image or the develop version image.

=== "Stable Version Image(0.6.0 version)"

      ```bash
      docker pull matrixorigin/matrixone:0.6.0
      docker run -d -p 6001:6001 --name matrixone matrixorigin/matrixone:0.6.0
      ```

=== "Develop Version Image"

      If you want to pull the develop version image, see [Docker Hub](https://hub.docker.com/r/matrixorigin/matrixone/tags), get the image tag. An example as below:

      ```bash
      docker pull matrixorigin/matrixone:nightly-commitnumber
      docker run -d -p 6001:6001 --name matrixone matrixorigin/matrixone:nightly-commitnumber
      ```

      __Notes__: The *nightly* version is updated once a day.

If you need to mount data directories or customize configure files, see [Mount the directory to Docker container](../Maintain/mount-data-by-docker.md)。

For the information on the user name and password, see <a href="#connect_mo">Step 2: Connect to standalone MatrixOne</a>.

## <h2><a name="connect_mo">Step 2: Connect to standalone MatrixOne</a></h2>

### Before you start

#### Install MySQL Client

!!! note
    MySQL client version 8.0.30 or later is recommended.

If the **MySQL client** has not been installed, you can find different installation methods of operation system in <a href="https://dev.mysql.com/doc/refman/8.0/en/installing.html" target="_blank">Installing and Upgrading MySQL</a>. You can follow the *Installing and Upgrading MySQL* to install it.

Or you can click <a href="https://dev.mysql.com/downloads/mysql" target="_blank">MySQL Community Downloads</a> to enter into the MySQL client download and installation page. According to your operating system and hardware environment, drop down to select **Select Operating System**, then drop down to select **Select OS Version**, and select the download installation package to install as needed.

#### Configure the MySQL client environment variables

After the installation is completed, configure the MySQL client environment variables:

=== "**Linux Environment**"

     1. Open a new terminal window and enter the following command:

         ```
         cd ~
         sudo vim /etc/profile
         ```

     2. After pressing **Enter** on the keyboard to execute the above command, you need to enter the root user password, which is the root password you set in the installation window when you installed the MySQL client. If no password has been set, press **Enter** to skip the password.

     3. After entering/skiping the root password, you will enter *profile*, click **i** on the keyboard to enter the insert state, and you can enter the following command at the bottom of the file:

        ```
        export PATH=/software/mysql/bin:$PATH
        ```

     4. After the input is completed, click **esc** on the keyboard to exit the insert state, and enter `:wq` at the bottom to save and exit.

     5. Enter the command `source  /etc/profile`, press **Enter** to execute, and run the environment variable.

     6. To test whether MySQL is available:

         - Method 1： Enter `mysql -u root -p`, press **Enter** to execute, the root user password is required, if `mysql>` is displayed, it means that the MySQL client is enabled.

         - Method 2: Run the command `mysql --version`, if MySQL client is installed successfully, the example code line is as follows: `mysql  Ver 8.0.31 for Linux on x86_64 (Source distribution)`

     7. If MySQL is available, close the current terminal and browse the next chapter **Connect to MatrixOne Server**.

=== "**MacOS Environment**"

     1. Open a new terminal window and enter the following command:

        ```
        cd ~
        sudo vim .bash_profile
        ```

     2. After pressing **Enter** on the keyboard to execute the above command, you need to enter the root user password, which is the root password you set in the installation window when you installed the MySQL client. If no password has been set, press **Enter** to skip the password.

     3. After entering/skiping the root password, you will enter *.bash_profile*, click **i** on the keyboard to enter the insert state, and you can enter the following command at the bottom of the file:

        ```
        export PATH=${PATH}:/usr/local/mysql/bin
        ```

     4. After the input is completed, click **esc** on the keyboard to exit the insert state, and enter `:wq` at the bottom to save and exit.

     5. Enter the command `source .bash_profile`, press **Enter** to execute, and run the environment variable.

     6. To test whether MySQL is available:

         - Method 1： Enter `mysql -u root -p`, press **Enter** to execute, the root user password is required, if `mysql>` is displayed, it means that the MySQL client is enabled.

         - Method 2: Run the command `mysql --version`, if MySQL client is installed successfully, the example code line is as follows: `mysql  Ver 8.0.31 for macos12 on arm64 (MySQL Community Server - GPL)`

     7. If MySQL is available, close the current terminal and browse the next chapter **Connect to MatrixOne Server**.

__Tips__: Currently, MatrixOne is only compatible with the Oracle MySQL client. This means that some features might not work with the MariaDB client or Percona client.

### **Connect to MatrixOne**

You can use the MySQL command-line client to connect to MatrixOne server. Open a new terminal window and enter the following command:

```
mysql -h IP -P PORT -uUsername -p
```

After you enter the preceding command, the terminal will prompt you to provide the username and password. You can use our built-in account:

- user: dump
- password: 111

You can also use the following command line on the MySQL client to connect to the MatrixOne service:

```
mysql -h 127.0.0.1 -P 6001 -udump -p
Enter password:
```

Currently, MatrixOne only supports the TCP listener.

## Reference

- For more information on the method of connecting to MatrixOne, see [Connecting to MatrixOne with Database Client Tool](../Develop/connect-mo/database-client-tools.md), [Connecting to MatrixOne with JDBC](../Develop/connect-mo/java-connect-to-matrixone/connect-mo-with-jdbc.md), and [Connecting to MatrixOne with Python](../Develop/connect-mo/python-connect-to-matrixone.md).

- For more information on the questions of deployment，see [Deployment FAQs](../FAQs/deployment-faqs.md).

- For more information on distributed installation, see [Deploy MatrixOne Cluster on Kubernetes Overview](../Deploy/install-and-launch-in-k8s.md)
