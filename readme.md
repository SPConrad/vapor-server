Setting up Vapor for Ubuntu:

Install Vagrant 

Set up new Vagrant box
Choose OS
I went with Ubuntu 20.04 as it the listed supported Swift version for it is higher than the other listed OSs
Initialize Vagrant box 
vagrant init bento/ubuntu-20.04
vagrant up

How do I install Vapor? 
https://docs.vapor.codes/4.0/install/linux/

First I need to install Swift
How do I install Swift?

Visit Swift.org's Using Downloads guide for instructions on how to install Swift on Linux.
https://swift.org/download/#using-downloads

Find reference to Ubuntu 20.04
tar.gz file for "Platform", tar.gz file for "Toolchain", and lists a Docker tag. 
Containers are cool, let's see if I can't use Docker to make this easy.

Let's install Docker
Do a search for installing Docker on Ubuntu
https://docs.docker.com/engine/install/ubuntu/

Attempt to uninstall old versions just in case 
$ sudo apt-get remove docker docker-engine docker.io containerd runc

"Most users set up Docker's repositories and install from them", cool, let's do that. 

1. Set up the Repository: 

$ sudo apt-get update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

2. Add Docker's official GPG key: 

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

2. a. Verify that you now have the key with the fingerprint 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88, by searching for the last 8 characters of the fingerprint.

$ sudo apt-key fingerprint 0EBFCD88

(expected output:)
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]

(actual output:)
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]

spot on. next step

3. Use the following command to set up the stable repository. To add the nightly or test repository, add the word nightly or test (or both) after the word stable in the commands below. Learn about nightly and test channels.

Note: The lsb_release -cs sub-command below returns the name of your Ubuntu distribution, such as xenial. Sometimes, in a distribution like Linux Mint, you might need to change $(lsb_release -cs) to your parent Ubuntu distribution. For example, if you are using Linux Mint Tessa, you could use bionic. Docker does not offer any guarantees on untested and unsupported Ubuntu distributions.

$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

Docker Part 2: Install Docker Engine

1. Update the apt package index, and install the latest version of Docker Engine and containerd, or go to the next step to install a specific version:

 $ sudo apt-get update
 (this next step might take a few minutes, required 85mb of downloads and 381mb of disk space)
 $ sudo apt-get install docker-ce docker-ce-cli containerd.io 


# Skipping step 2 which regards installation of multiple Docker repositories as it is not relevant 

3. Verify that Docker Engine is installed correctly by running the hello-world image.

$ sudo docker run hello-world

(actual output:)
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:7f0a9f93b4aa3022c3a4c147a449bf11e0941a1fd0bf4a8e6c9408b2600777c5
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/


# Post-installation steps for Linux: 

Manage Docker as a non-root user

To create the docker group and add your user:

Create the docker group.

$ sudo groupadd docker
Add your user to the docker group.

$ sudo usermod -aG docker $USER
Log out and log back in so that your group membership is re-evaluated.

If testing on a virtual machine, it may be necessary to restart the virtual machine for changes to take effect.

On a desktop Linux environment such as X Windows, log out of your session completely and then log back in.

On Linux, you can also run the following command to activate the changes to groups:

$ newgrp docker 
Verify that you can run docker commands without sudo.

$ docker run hello-world
This command downloads a test image and runs it in a container. When the container runs, it prints an informational message and exits.

If you initially ran Docker CLI commands using sudo before adding your user to the docker group, you may see the following error, which indicates that your ~/.docker/ directory was created with incorrect permissions due to the sudo commands.

WARNING: Error loading config file: /home/user/.docker/config.json -
stat /home/user/.docker/config.json: permission denied
To fix this problem, either remove the ~/.docker/ directory (it is recreated automatically, but any custom settings are lost), or change its ownership and permissions using the following commands:

$ sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
$ sudo chmod g+rwx "$HOME/.docker" -R




Configure Docker to start on boot
Most current Linux distributions (RHEL, CentOS, Fedora, Ubuntu 16.04 and higher) use systemd to manage which services start when the system boots. Ubuntu 14.10 and below use upstart.

systemd
$ sudo systemctl enable docker
To disable this behavior, use disable instead.

$ sudo systemctl disable docker
If you need to add an HTTP Proxy, set a different directory or partition for the Docker runtime files, or make other customizations, see customize your systemd Docker daemon options.

upstart
Docker is automatically configured to start on boot using upstart. To disable this behavior, use the following command:

$ echo manual | sudo tee /etc/init/docker.override
chkconfig
$ sudo chkconfig docker on

Use a different storage engine https://docs.docker.com/storage/storagedriver/
For information about the different storage engines, see Storage drivers. The default storage engine and the list of supported storage engines depend on your host’s Linux distribution and available kernel drivers.

Configure default logging driver
Docker provides the capability (https://docs.docker.com/config/containers/logging/) to collect and view log data from all containers running on a host via a series of logging drivers. The default logging driver, json-file, writes log data to JSON-formatted files on the host filesystem. Over time, these log files expand in size, leading to potential exhaustion of disk resources. To alleviate such issues, either configure an alternative logging driver such as Splunk or Syslog, or set up log rotation (https://docs.docker.com/config/containers/logging/configure/#configure-the-default-logging-driver) for the default driver. If you configure an alternative logging driver, see Use docker logs to read container logs for remote logging drivers.

https://docs.docker.com/config/containers/logging/dual-logging/

Configure where the Docker daemon listens for connections
By default, the Docker daemon listens for connections on a UNIX socket to accept requests from local clients. It is possible to allow Docker to accept requests from remote hosts by configuring it to listen on an IP address and port as well as the UNIX socket. For more detailed information on this configuration option take a look at “Bind Docker to another host/port or a unix socket” section of the Docker CLI Reference article.

Secure your connection

Before configuring Docker to accept connections from remote hosts it is critically important that you understand the security implications of opening docker to the network. If steps are not taken to secure the connection, it is possible for remote non-root users to gain root access on the host. For more information on how to use TLS certificates to secure this connection, check this article on how to protect the Docker daemon socket.

Configuring Docker to accept remote connections can be done with the docker.service systemd unit file for Linux distributions using systemd, such as recent versions of RedHat, CentOS, Ubuntu and SLES, or with the daemon.json file which is recommended for Linux distributions that do not use systemd.

systemd vs daemon.json

Configuring Docker to listen for connections using both the systemd unit file and the daemon.json file causes a conflict that prevents Docker from starting.

Configuring remote access with systemd unit file
Use the command sudo systemctl edit docker.service to open an override file for docker.service in a text editor.

Add or modify the following lines, substituting your own values.

[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375
Save the file.

Reload the systemctl configuration.

 $ sudo systemctl daemon-reload
Restart Docker.

$ sudo systemctl restart docker.service
Check to see whether the change was honored by reviewing the output of netstat to confirm dockerd is listening on the configured port.

$ sudo netstat -lntp | grep dockerd
tcp        0      0 127.0.0.1:2375          0.0.0.0:*               LISTEN      3758/dockerd
Configuring remote access with daemon.json
Set the hosts array in the /etc/docker/daemon.json to connect to the UNIX socket and an IP address, as follows:

{
"hosts": ["unix:///var/run/docker.sock", "tcp://127.0.0.1:2375"]
}
Restart Docker.

Check to see whether the change was honored by reviewing the output of netstat to confirm dockerd is listening on the configured port.

$ sudo netstat -lntp | grep dockerd
tcp        0      0 127.0.0.1:2375          0.0.0.0:*               LISTEN      3758/dockerd



errored out at this stage
"docker.socket: Failed with result 'service-start-limit-hit'."

Removing daemon.json fixed it???????

DUH

systemd vs daemon.json

Configuring Docker to listen for connections using both the systemd unit file and the daemon.json file causes a conflict that prevents Docker from starting.

