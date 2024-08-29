# Docker Image Optimization

#### Go to AWS Management console and Create an Ubuntu Ec2 instance

Switch the user as root and update the server
```
sudo su
apt update -y
```
### Install Docker On Ubuntu Ec2 Instance

```
apt install docker.io -y
```
#### Always ensure that the docker daemon is up and running

Verify your Docker installation is by running the below command
```
docker run hello-world
```
If the output says:
```
docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/create": dial unix /var/run/docker.sock: connect: permission denied.
See 'docker run --help'.
```
That means docker daemon is not or there is a problem related to access

To verify if the docker daemon is actually started and Active
```
systemctl status docker
```
If you notice that the docker daemon is not running, you can start the daemon using the below command
```
systemctl start docker
```
### Grant Access to your user to run docker commands
To grant access to your user to run the docker command, you should add the user to the Docker Linux group. Docker group is create by default when docker is installed.
```
usermod -aG docker ubuntu
```
**Note**:: You need to logout and login back for the changes to be reflected.

## Docker is Installed, up and running

Use the same command again, to verify that docker is up & running.
```
docker run hello-world
```

Output should look like:
```
....
....
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
...
```
