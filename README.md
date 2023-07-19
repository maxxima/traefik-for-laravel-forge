# Traefik for Laravel Forge


## Features


## Usage


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
