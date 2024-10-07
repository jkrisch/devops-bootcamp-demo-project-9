# Demo Projects - AWS

##  Deploy to EC2 server from Jenkins Pipeline
After the EC2 instance has been setup (using the aws console ui) and the IP adress is added to the security group I accessed the EC2 instance using my custom key pair.
Then I executed.
``` bash
sudo yum update
sudo yum -y install docker
sudo systemctl start docker.service
sudo systemctl start docker.socket
sudo usermod -aG docker $USER
```
with this the ec2 user can now run docker commands.

1. run vis ssh the following command (in the Jenkinsfile see: https://github.com/jkrisch/devops-bootcamp-demo-project-9/commit/4e3c373f3ffa86322fae61fe16fb4ada71f9bcd5#diff-e6ffa5dc854b843b3ee3c3c28f8eae2f436c2df2b1ca299cca1fa5982e390cf8L59)
``` bash
docker run -p 8080:8080 -d ${IMAGE_NAME}
```
2. switch to docker-compose by scp (secure copy) the [compose file](compose.yml) to the remote host on aws and by adjusting the command to:
``` bash
docker-compose up -d
```
***Note***:
requires docker-compose to be available on the EC2 instance.

3. now add the version increment stage (from lecture 8) and adjust the [](Jenkinsfile) accordingly.