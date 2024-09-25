# Docker Compose file for Standalone Jenkins

If you prefer to use a Docker compose instead of manually creating a resource using the __docker__ command, then just copy the following YAML into a `docker-compose.yml`

```yaml
volumes:
  home:

networks:
  ci-network:
    ipam:
      config:
        - subnet: 172.30.0.0/24

services:
  master:
    container_name: master
    image: jenkins/jenkins:lts
    restart: unless-stopped
    networks:
      - ci-network
    ports:
      - 8080:8080
      - 50000:50000
    volumes:
      - home:/var/jenkins_home
```

> [!TIP]
> For a more details on what is going on, checkout this [Guide](./standalone-docker.md). The compose file basically combines all the commands used that guide into a single script.

## Run the Compose

```bash
docker compose up -d
```

## Access the One Time Password

```bash
docker compose logs master
```
> [!IMPORTANT]
> The One Time Password looks like `c3e77f61e3214cfdab0d557f7ac25796`

> [!TIP]
> This completes the deployment process. [Click Here](../README.md#contents) to go back to Homepage.
