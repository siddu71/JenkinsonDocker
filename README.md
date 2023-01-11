# JenkinsonDocker (ubuntu/64bit)
This setup shows how to install jenkins in a docker container.

reference : https://octopus.com/blog/jenkins-docker-install-guide

# 1. Install Docker : 

```
sudo apt-get update
sudu apt-get upgrade
sudo apt install docker.io
```

# 2. Create a Bridge network in Docker using docker network create command: 
```
docker network create jenkins
```
# 3. Docker run 
```
docker run -d --restart=on-failure -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk11
```
# 4. Check if container is running:
```
Docker ps -a
```
# 5. Check docker logs to find initial password or navigate to /var/jenkins_home/secrets/initialAdminPassword

```
docker logs [Container ID] 
```
```
docker exec -it [Container ID] bash

cd /var/jenkins_home/secrets/

cat initialAdminPassword
```
Copy and paste password during installation

# 6. Open https://[Server id]:8080 in browser 

![image](https://user-images.githubusercontent.com/52039971/198523088-6e2791e4-79e2-43fe-a36c-dfde595be3ae.png)

paste the password and continue the setup process and install suggested plugins.

wait for the setup process to complete.

Setup the user details.

# 7. You have sucessfully installed jenkins container in docker.
![image](https://user-images.githubusercontent.com/52039971/198523829-f1e00662-ec49-462e-a6ab-a53a3806872a.png)


