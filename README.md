# Traefik for Laravel Forge

This repo was forked from

There is an original blog post, https://blog.jpat.dev/how-to-deploy-docker-applications-with-laravel-forge, that gives a good overview of how to set up Traefik on Laravel Forge. However, the instructions didn't allow me to get Plausible Analytics working with Traefik.

This repo is an attempt to provide additional information that I discovered when getting Plausible to work with Traefik. The TLDR was that Traefik and Plausible were not always defaulting to use the same network and specifying the network names solved the issue.
## Laravel Forge preparation

The repo assumes you have Docker installed already. As mentioned in the original blog post, if you don't, you can install it with the following set of commands. You run them as `root` user. The commands are ideally set up as a Recipe in Laravel Forge so that you can reuse them across multiple servers.

```
# Recipe Name: Install Docker and Docker-Compose
# Run as user: root
# Recipe:
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
UBUNTU_VERSION=$(lsb_release -cs)
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $UBUNTU_VERSION stable"
apt-cache policy docker-ce
apt install -y docker-ce
systemctl status docker
usermod -aG docker forge
COMPOSE_VERSION=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d\" -f4)
sh -c "curl -L https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose"
chmod +x /usr/local/bin/docker-compose
```

### Deploy


## Other repositories

There are 2 additional repositroies that are used in this project:

1. A basic static website that is used as an example for Traefik to route to
https://github.com/johnfmorton/example-static-docker-website-for-traefik
2. The Plausible Analytics docker-compose file that hosts the Plausible Analytics service. It has been modified slightly to work with Traefik. https://github.com/johnfmorton/plausible-with-traefik-update-for-laravel-forge

## Original instructions

The original README from the Traefik repository is below.

Step 1. Deploy Script
Refer to (deploy script)[./Taskfile], in function `baseDeployScript`

Step 2. Configure ENV variables, and set Traefik Dashboard password
!!! Recommended doing via SSH, even you could fdo it from Forge's UI "Run command"
```
./Taskfile copyEnv
./Taskfile auth username password

```
