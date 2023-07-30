# Traefik Proxy for Laravel Forge

[![Laravel Forge Site Deployment Status](https://img.shields.io/endpoint?url=https%3A%2F%2Fforge.laravel.com%2Fsite-badges%2Fed322e5f-fb4d-4ab8-9fab-d0d54ce4f4e3%3Fdate%3D1%26commit%3D1&style=plastic)](https://forge.laravel.com/servers/699880/sites/2038639)

[Traefik Proxy](https://doc.traefik.io/traefik/) will help you get routing configured on your server between multiple services.

This repo is a fork of [ijpatricio/traefik-for-forge](https://github.com/ijpatricio/traefik-for-forge), so be sure to check that out. There is an original blog post that accompanied that repo, https://blog.jpat.dev/how-to-deploy-docker-applications-with-laravel-forge, that gives a great overview of how to set up Traefik on Laravel Forge. That post was very helpful in getting Traefik working on Laravel Forge.

However, the post didn't allow me to get Plausible Analytics working with Traefik.

The TLDR was that Traefik and Plausible were not always defaulting to use the same network and specifying the network names solved the issue.

This forked repo is an attempt to share what I discovered when getting Plausible to work with Traefik. This repo shows how I have Traefik configured to run on Laravel Forge.

## Getting started

If you're just going to dive right in without reading the [original post](https://blog.jpat.dev/how-to-deploy-docker-applications-with-laravel-forge), be sure to set up the most basic server in Laravel Forge, a _Worker Server_, because it will have only minimal services installed. You'll need to install Docker and Docker Compose on your server. I've included a Laravel Forge Recipe that you can use to install Docker and Docker Compose. See the [Laravel Forge preparation](#laravel-forge-preparation) section below for more details.

Another repo that I created for testing purposes is available at [johnfmorton/example-static-docker-website-for-traefik](https://github.com/johnfmorton/example-static-docker-website-for-traefik). This is the repo I used during my debugging process. It's a very basic static website that I used to test Traefik routing. It displays a "Hello World" message if you can successfully reach it. This repo was useful because it helped me prove to myself that my Traefik Proxy was working correctly. Notice the references to the `proxy` network in the [docker-compose.yml](https://github.com/johnfmorton/example-static-docker-website-for-traefik/blob/main/docker-compose.yml) of that repo.

Lastly, you'll probably also want to check out the accompanying repo for Plausbile Analytics, [johnfmorton/plausible-with-traefik-update-for-laravel-forge](https://github.com/johnfmorton/plausible-with-traefik-update-for-laravel-forge). Specifically, check out the [Traefik reverse proxy file](https://github.com/johnfmorton/plausible-with-traefik-update-for-laravel-forge/blob/master/reverse-proxy/traefik/docker-compose.traefik.yml) in that repo where you will find refereces to the `proxy` network defined in this Traefik repo.

## Laravel Forge preparation

This Traefik Proxy repo assumes you have Docker installed already. If you don't, you can install it with the following set of commands. You run them as `root` user. The commands are ideally set up as a Recipe in Laravel Forge so that you can reuse them across multiple servers.

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
sudo apt install -y docker-ce
systemctl status docker
usermod -aG docker forge
COMPOSE_VERSION=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d\" -f4)
sh -c "curl -L https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose"
chmod +x /usr/local/bin/docker-compose
```

This is a slight modification of the [original version of the script](https://github.com/ijpatricio/docker-for-forge/blob/main/resources/forge-recipe.sh). I've added the `sudo` to add the `apt install -y docker-ce` line for consistency with the other `sudo app install` commands in the script. But, since the user for this script is `root`, I think you could potentially eliminate all use of `sudo` in the script entirely, but I did not try that in my tests. The [updated script](./forge-recipe-install-docker.sh) is also available in this repo.

## DNS setup

You will need to set up DNS to point your domain name with an A record to your server you've created with Laravel Forge. For example, `traefik.yourdomain.com` should point to your server's IP address.

## Setting up the `.env` file

In the .env file, set `TRAEFIK_DASHBOARD_HOST` to this domain name. For example, `analytics.yourdomain.com`. No `https://` or `http://` is needed. This is the URL were you Traefik dashboard will be available.

The `LETS_ENCRYPT_EMAIL` is the email address that will be used to register your SSL certificate with Let's Encrypt. This is required for Traefik to work.

The `REDIRECT_IP_ADDRESS_TO_URL` is a URL that will be used as a catch-all if someone tries to visit your IP address.

See the [`example.env`](/example.env) for reference.

## Other repositories

As mentioned in the intro, there is an accompanying repo for Plausible Analytics. I also created a basic static website repo for testing purposes. Here are links to both repos.


1. The Plausible Analytics docker-compose file that hosts the Plausible Analytics service. It has been modified slightly to work with Traefik. https://github.com/johnfmorton/plausible-with-traefik-update-for-laravel-forge
2. A basic static website that is used as an example for Traefik to route to
https://github.com/johnfmorton/example-static-docker-website-for-traefik. This is the repo I used during my debugging process. It's a very basic static website that I used to test Traefik routing. It displays a "Hello World" message if you can successfully reach it.


## Reference links

I referenced and borrowed from all of the following links. I'm grateful the authors have all shared their knowledge.

* https://blog.jpat.dev/how-to-deploy-docker-applications-with-laravel-forge
* https://putyourlightson.com/articles/replacing-google-analytics-with-self-hosted-analytics
* https://plausible.io/docs/self-hosting
* https://www.digitalocean.com/community/tutorials/how-to-use-traefik-as-a-reverse-proxy-for-docker-containers-on-ubuntu-20-04

## Additional notes

The original README from the Traefik repository is below. The command sets up the basic auth used to protect your Traefik dashboard.

### Original instructions from forked repo

Step 1. Deploy Script
Refer to (deploy script)[./Taskfile], in function `baseDeployScript`

Step 2. Configure ENV variables, and set Traefik Dashboard password
!!! Recommended doing via SSH, even you could fdo it from Forge's UI "Run command"
```
./Taskfile copyEnv
./Taskfile auth username password
```

## License

The source code for the site is licensed under the MIT license, which you can find in
the MIT-LICENSE.txt file.
