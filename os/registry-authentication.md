# Using Authentication for a Registry

A json file `.dockercfg` is generated in your home directory on `docker login`. It holds authentication information for a public or private Docker registry. This `.dockercfg` can be reused in other home directories to authenticate. One way to do this is using Cloud-Config which is discussed more below. If you want to populate these values without running Docker login, the auth token is a base64 encoded string: `base64(<username>:<password>)`.

## The .dockercfg File

Here's what an example looks like with credentials for Docker's public index and a private index:


#### .dockercfg

```json
{
  "https://index.docker.io/v1/": {
    "auth": "xXxXxXxXxXx=",
    "email": "username@example.com"
  },
  "https://index.example.com": {
    "auth": "XxXxXxXxXxX=",
    "email": "username@example.com"
  }
}
```


The last step is to tell your systemd units to run as the `core` user in order for Docker to use the credentials we just set up. This is done in the service section of the unit:

```ini
[Unit]
Description=My Container
After=docker.service

[Service]
User=core
ExecStart=/usr/bin/docker run busybox /bin/sh -c "while true; do echo Hello World; sleep 1; done"

[Install]
WantedBy=multi-user.target
```

### Cloud-Config

Since each machine in your cluster is going to have to pull images, cloud-config is the easiest way to write the config file to disk.

```yaml
#cloud-config
write_files:
    - path: /home/core/.dockercfg
      owner: core:core
      permissions: '0644'
      content: |
        {
          "https://index.docker.io/v1/": {
            "auth": "xXxXxXxXxXx=",
            "email": "username@example.com"
          },
          "https://index.example.com": {
            "auth": "XxXxXxXxXxX=",
            "email": "username@example.com"
          }
        }
```

## Using a Registry Without SSL Configured

The default behavior of Docker is to prevent access to registries that aren't using SSL. If you're running a registry behind your firewall without SSL, you need to configure an additional parameter, which whitelists a CIDR range of allowed "insecure" registries.

The best way to do this is within your cloud-config:

```yaml
#cloud-config

write_files:
  - path: /etc/systemd/system/docker.service.d/50-insecure-registry.conf
    content: |
        [Service]
        Environment='DOCKER_OPTS=--insecure-registry="10.0.1.0/24"'
```
