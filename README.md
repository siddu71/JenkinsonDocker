# JenkinsonDocker (ubuntu/64bit)
This setup shows how to install jenkins in a docker container.

```
#!/bin/bash

# Function to check if the command was successful
check_success() {
  if [ $? -eq 0 ]; then
    printf "\n + $1 installation succeeded\n"
  else
    echo "$1 installation failed"
    exit 1
  fi
}

# INSTALL DOCKER

echo -n "Updating system packages..."
sudo apt-get update -y > /dev/null 2>&1
check_success "System package update"

if dpkg --get-selections | grep -q "^docker.*install$" >/dev/null; then
    echo -n "Removing old versions of Docker..."
    sudo apt-get remove docker docker-engine docker.io containerd runc > /dev/null 2>&1
    check_success "Docker removal"
else
    echo "Docker not installed, skipping removal"
fi

echo -n "Updating system packages again..."
sudo apt-get update -y > /dev/null 2>&1
check_success "System package update"

echo -n "Installing ca-certificates, curl, and gnupg..."
sudo apt-get install ca-certificates curl gnupg -y > /dev/null 2>&1
check_success "Installation of ca-certificates, curl, and gnupg"

echo -n "Setting up Docker repository..."
sudo install -m 0755 -d /etc/apt/keyrings > /dev/null 2>&1
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg > /dev/null 2>&1
sudo chmod a+r /etc/apt/keyrings/docker.gpg > /dev/null 2>&1
check_success "Docker repository setup"

echo -n "Adding Docker repository to apt sources..."
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null 2>&1 

check_success "Adding Docker repository to apt sources"

echo -n "Updating system packages again..."
sudo apt-get update -y > /dev/null 2>&1
check_success "System package update"

echo -n "Installing Docker..."
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y > /dev/null 2>&1
check_success "Docker installation"

echo -n "Running Docker Hello World..."
sudo docker run hello-world > /dev/null 2>&1
check_success "Docker Hello World"

echo -n "Setting Docker permissions..."
sudo chmod 777 /var/run/docker.sock > /dev/null 2>&1
check_success "Setting Docker permissions"


# Create Jenkins Docker network
echo -n "Creating Jenkins Docker network..."
docker network create jenkins
check_success "Jenkins Docker network creation"

# Run Jenkins container
echo -n "Running Jenkins Docker container..."
docker run -d --name=jenkins --restart=on-failure -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk11
check_success "Running Jenkins Docker container"

# Function to setup Jenkins user
setup_jenkins() {
  echo -n "Waiting for Jenkins to start..."

  # We'll use a variable to keep track of whether Jenkins has started yet
  password_file_exists=""

  while [ -z $password_file_exists ]
  do
    # Check if the initialAdminPassword file exists inside the Docker container
    password_file_exists=$(docker exec jenkins bash -c "if [ -f /var/jenkins_home/secrets/initialAdminPassword ]; then echo 'yes'; else echo ''; fi")
    sleep 2
  done
  echo "Jenkins started."

# Fetch the initial admin password
initial_admin_password=$(docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword)

echo "Initial Admin Password: $initial_admin_password"

# Define your preferred Jenkins user and password here
username="new_user"
password="new_password"

# Install OpenJDK
sudo apt install default-jdk -y

# Download jenkins-cli.jar
wget http://localhost:8080/jnlpJars/jenkins-cli.jar

# Create Jenkins user through Jenkins CLI
echo 'jenkins.model.Jenkins.instance.securityRealm.createAccount("'$username'", "'$password'")' | java -jar ./jenkins-cli.jar -s "http://localhost:8080" -auth admin:$initial_admin_password -noKeyAuth groovy =

# Check success of user creation
if [ $? -eq 0 ]; then
    echo "Jenkins user setup successful"
else
    echo "Jenkins user setup failed"
fi

}

# Run the Jenkins setup function
setup_jenkins
```

reference : https://octopus.com/blog/jenkins-docker-install-guide

# 1. Install Docker : 

```
sudo apt-get update -y
sudu apt-get upgrade -y
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


Note: Add 8080 to inbound rules to access the server from browser.
