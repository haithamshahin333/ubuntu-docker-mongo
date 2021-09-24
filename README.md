# Deploy Mongo DB as a container on Ubuntu VM

### Setup Ubuntu VM

1. Deploy the VM - if needed you can add more paramters to [az vm create](https://docs.microsoft.com/en-us/cli/azure/vm?view=azure-cli-latest#az_vm_create)

```
# create the ubuntu VM and specify appropriate values

az vm create \
  --resource-group $RG_NAME \
  --name $VM_NAME \
  --image UbuntuLTS \
  --admin-username azureuser \
  --ssh-key-values $SSH_PUBLIC_KEY_FILE
  --vnet-name $VNET_NAME
  --subnet $SUBNET_NAME
```

2. SSH into the VM by obtaining the public IP:

```
ssh -i <PATH_TO_PRIVATE_KEY> azureuser@<PUBLIC_IP>
```

## Install Docker on VM

3. Run the following to install docker - this is from the [Docker Documentation for Ubuntu Installations](https://docs.docker.com/engine/install/ubuntu/#installation-methods).

```
### update and install base packages
sudo apt-get update

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

### docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

### setup the stable repository
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

### update and install docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

### confirm install worked by running hello-world
sudo docker run hello-world

### post-install steps to run as non-root
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker 

### validate that sudo is not needed
docker run hello-world
```

4. [Install Docker-Compose](https://docs.docker.com/compose/install/)

```
### install docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

### validate the install
docker-compose --version
```

## Run MongoDB Container from Bitnami

5. Run Mongo DB using the Docker Compose File from Bitnami - [GitHub Repo Reference](https://github.com/bitnami/bitnami-docker-mongodb).

> IMPORTANT: By default, the mongo db container image will allow unauthenticated and unrestricted access to the container image. You should specify a username and password as shown in the [bitnami documentation on setting the root user & password](https://github.com/bitnami/bitnami-docker-mongodb#setting-the-root-user-and-password-on-first-run)

> IMPORTANT: The commands below shows how to add the environment variables to the docker-compose file so that there is a username and password for the image.

```
### Copy the file docker-compose.yaml from this repo to your VM
### This docker-compose file can be modified for your desired root username, password, and mongo db image version


### start the container using the compose file
### this will expose port 27017 and mount a local share to the container
### by default the volume is stored in /var/lib/docker
docker-compose up -d
```

6. Install the [mongo db shell](https://docs.mongodb.com/mongodb-shell/install/)

```
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -

echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list

sudo apt-get update
sudo apt-get install -y mongodb-mongosh
```

7. Connect to your mongo db container locally using the shell. From there you can run commands to insert data etc.:

```
### by default this will connect to mongodb://localhost:27017
mongosh
```

8. If required, you can expose your Mongo DB over a public IP by updating the NSG to include an inbound allow rule for destination port 27017 (or whatever port you specify MongoDB is exposed on). The following [Visual Studio Code extension](https://code.visualstudio.com/docs/azure/mongodb) is good for a GUI to connect to the DB.

> IMPORTANT: By default, the mongo db container image will allow unauthenticated and unrestricted access to the container image. You should specify a username and password as shown in the [bitnami documentation on setting the root user & password](https://github.com/bitnami/bitnami-docker-mongodb#setting-the-root-user-and-password-on-first-run). The docker-compose file shown here has an example of setting this.