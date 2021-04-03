![Flight Deck](https://raw.githubusercontent.com/ten7/flight-deck/main/flightdeck-logo.png)

# Flight Deck Varnish

Flight Deck Varnish is a minimalist Varnish container for Drupal sites on Kubernetes and Docker. You can use it both for local development and production.

Features:
* ConfigMap-friendly YAML configuration

## Tags and versions

There are several tags available for this container, each with different Solr and module support:

| Varnish version | Tags |
| --------------- | ---- |
| 6.1 | 6, latest |

## Configuration

This container does not use environment variables for configuration. Instead, the `flight-deck-varnish.yml` file is used to handle all configuration.

```yaml
---
flightdeck_varnish:
  secret: "secretish"
  hostPort: "6081"
  controlPort: "6082"
  memSize: "256m"
  storageFile: "/var/lib/varnish/storage.bin"
  storageSize: "1024m"
```

Where:
* **secret** is the Varnish secret used to access the control terminal. Optional.
* **hostPort** is the port on which the Varnish proxy is available. Optional, defaults to `6081`.
* **controlPort** is the port for the Varnish control terminal. Optional, defaults to `6082`.
* **memSize** is the size of memory to dedicate to caching. Optional, defaults to `256m`.
* **storageFile** is the full path in the container to the file to use for longer term varnish caching. Optional, defaults to `/var/lib/varnish/storage.bin`.
* **storageSize** is the size of the storage file. Optional, defaults to `1024m`.

## Describing backends

Varnish acts as a caching reverse HTTP proxy. You can specify one or more backends by defining the `flightdeck_varnish.backends` list:

```yaml
---
flightdeck_varnish:
  backends:
   - name: "default"
     host: "web"
     port: "80"
```

For each item:

* **name** is the a name of the backend. Required.
* **host** is the IP address, hostname, or fully qualified DNS name with which to access the backend. Required.
* **port** is the port with which to access the backend. Optional, defaults to `80`.

### Using separate files for credentials

Sometimes, you may wish to keep certain credentials in separate files from the rest of the configuration. One such case is if you want to keep `flight-deck-varnish.yml` in a Kubernetes configmap, but keep the database passwords in secrets instead.

```yaml
---
flightdeck_varnish:
  secretFile: "/config/varnish-secret/secret.txt"
```

Where:

* **secretFile** is the path in the container to the file containing the varnish secret.

### Excluding paths from caching

Some paths, such administration paths, key user interaction paths should not be cached by varnish. If the user login page is cached, for example, you may not be able to login, as Varnish would return the un-logged in HTML each time.

You can configure which items always bypass the cache in the configuration file:

```yaml
---
flightdeck_varnish:
  skipCache:
    - "^/status\\.php$"
    - "^/update\\.php"
    - "^/install\\.php"
    - "^/wp-login\\.php$"
    - "^/apc\\.php$"
    - "^/admin"
    - "^/admin/.*$"
    - "^/user"
    - "^/user/.*$"
    - "^/users/.*$"
    - "^/info/.*$"
    - "^/flag/.*$"
    - "^/wp-admin/.*$"
    - "^.*/ahah/.*$"
    - "^/system/files/.*$"
```

Where:

* **skipCache** is a list of regular expressions which match URL paths to skip caching. Note, YAML needs backslashes to be escaped with another backslash. Optional, the container will skip best practice Drupal and Wordpress paths.

## Deployment on Kubernetes

Use the [`ten7.flightdeck_cluster`](https://galaxy.ansible.com/ten7/flightdeck_cluster) role on Ansible Galaxy to deploy DB as a statefulset:

```yaml
flightdeck_cluster:
  namespace: "example-com"
  configMaps:
    - name: "flight-deck-varnish"
      files:
        - name: "flight-deck-varnish.yml"
          content: |
          flightdeck_varnish:
            secret: "secretish"
            hostPort: "6081"
            controlPort: "6082"
            memSize: "256m"
            storageFile: "/var/lib/varnish/storage.bin"
            storageSize: "1024m"
            backends:
             - name: "default"
               host: "web"
               port: "80"
  web:
    replicas: 1
    configMaps:
      - name: "flight-deck-varnish"
        path: "/config/varnish"
```

## Using with Docker Compose

Create the `flight-deck-varnish.yml` file relative to your `docker-compose.yml`. Define the `varnish` service mounting the file as a volume:

```yaml
version: '3'
services:
  varnish:
    image: ten7/flight-deck-varnish:6
    ports:
      - 6081:6081
      - 6082:6082
    volumes:
      - ./flight-deck-varnish.yml:/config/varnish/flight-deck-varnish.yml
```

## Part of Flight Deck

This container is part of the [Flight Deck library](https://github.com/ten7/flight-deck) of containers for Drupal local development and production workloads on Docker, Swarm, and Kubernetes.

Flight Deck is used and supported by [TEN7](https://ten7.com/).


## Debugging

If you need to get verbose output from the entrypoint, set `flightdeck_debug` to `true` or `yes` in the config file.

```yaml
---
flightdeck_debug: yes
```

This container uses [Ansible](https://www.ansible.com/) to perform start-up tasks. To get even more verbose output from the start up scripts, set the `ANSIBLE_VERBOSITY` environment variable to `4`.

If the container will not start due to a failure of the entrypoint, set the `FLIGHTDECK_SKIP_ENTRYPOINT` environment variable to `true` or `1`, then restart the container.

## License

Flight Deck is licensed under GPLv3. See [LICENSE](https://raw.githubusercontent.com/ten7/flight-deck/master/LICENSE) for the complete language.
