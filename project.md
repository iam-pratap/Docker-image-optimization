# Docker Image Optimization

Docker file that you have written for application in your for your organization and you require python runtime to execute this application but if you look at the docker image that you have created it has 100 other thing it has a ubuntu base image will come with a lot of ubuntu system dependencies, apt packages and apt repositories it has a lot of overload on your docker image.

End of the day you just want to run this calculator application and that calculator application only requires python runtime not even the applications that you have installed. As part of the stages these are only required to build your application. So these are two different stages, running your application is required to build your application. Then docker introduced a concept called `Multistaged build`

You will split our Dockerfile into two parts

If you directly use python runtime image as a base image the problem would be you have to install a lot of things your docker image will became very complicated. you choose a very rich base image. base image that has all the dependencies. you are not even worried if this base image is 1GB or 2GB. The reason for that is base image will be removed and finally the docker image that you are creating will only have the last stage that is a final stage which will be a minimalistic image so it has to be a minimalistic image because by choosing a minimalistic image you will get the advantage of your docker multi stage.

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
#### Taken this calculator application from GitHub

Create `calculator.go` file and apply the below configuration in this file

```
package main

import (
        "bufio"
        "fmt"
        "os"
        "strconv"
        "strings"
)

func main() {
        fmt.Println("Hi Honey Pratap, I am a calculator app ....")

        for {
                // Read input from the user
                reader := bufio.NewReader(os.Stdin)
                fmt.Print("Enter any calculation (Example: 1 + 2 (or) 2 * 5 -> Please maintain spaces as shown in example): ")
                text, _ := reader.ReadString('\n')

                // Trim the newline character from the input
                text = strings.TrimSpace(text)

                // Check if the user entered "exit" to quit the program
                if text == "exit" {
                        break
                }

                // Split the input into two parts: the left operand and the right operand
                parts := strings.Split(text, " ")
                if len(parts) != 3 {
                        fmt.Println("Invalid input. Try again.")
                        continue
                }

                // Convert the operands to integers
                left, err := strconv.Atoi(parts[0])
                if err != nil {
                        fmt.Println("Invalid input. Try again.")
                        continue
                }
                right, err := strconv.Atoi(parts[2])
                if err != nil {
                        fmt.Println("Invalid input. Try again.")
                        continue
                }

                // Perform the calculation based on the operator
                var result int
                switch parts[1] {
                case "+":
                        result = left + right
                case "-":
                        result = left - right
                case "*":
                        result = left * right
                case "/":
                        result = left / right
                default:
                        fmt.Println("Invalid operator. Try again.")
                        continue
                }

                // Print the result
                fmt.Printf("Result: %d\n", result)
        }
}
```
### Do without Mutltistage build

Create `Dockerfile` and apply the below configuration

```

###########################################
# BASE IMAGE
###########################################

FROM ubuntu AS build

RUN apt-get update && apt-get install -y golang-go

ENV GO111MODULE=off

COPY . .

RUN CGO_ENABLED=0 go build -o /app .

ENTRYPOINT ["/app"]
```

Build this image using Dockerfile

```
docker build -t simplecalculator .
```

Check the docker is created or not
```
docker images
```
Output should be:
```
simplecalculator   latest    abbbc058f909   8 seconds ago   666MB
ubuntu             latest    edbfe74c41f8   3 weeks ago     78.1MB
hello-world        latest    d2c94e258dcb   16 months ago   13.3kB
```
### Do with Multistage build

Create `Dockerfile` and apply the below configuration

```
######################
# BASE IMAGE(stage@1)
######################

FROM ubuntu AS build

RUN apt-get update && apt-get install -y golang-go

ENV GO111MODULE=off

COPY . .

RUN CGO_ENABLED=0 go build -o /app .

####################
# stage@2
####################

FROM scratch

# Copy the compiled binary from the build stage
COPY --from=build /app /app

# Set the entrypoint for the container to run the binary
ENTRYPOINT ["/app"]
```
Build this image using Dockerfile
```
docker build -t multi-stage-test .
```
Output should be:
```
REPOSITORY         TAG       IMAGE ID       CREATED              SIZE
multi-stage-test   latest    4b19e4557276   About a minute ago   1.96MB
simplecalculator   latest    abbbc058f909   6 minutes ago        666MB
ubuntu             latest    edbfe74c41f8   3 weeks ago          78.1MB
hello-world        latest    d2c94e258dcb   16 months ago        13.3kB

```

create a container using `multi-stage-test` image
```
docker run -it --name app1 multi-stage-test /bin/bash
```

Output look like 
```
Hi Honey Pratap, I am a calculator app ....
Enter any calculation (Example: 1 + 2 (or) 2 * 5 -> Please maintain spaces as shown in example): 2 + 3
Result: 5
```
