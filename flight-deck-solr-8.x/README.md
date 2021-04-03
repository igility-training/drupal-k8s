![Flight Deck](https://raw.githubusercontent.com/ten7/flight-deck/main/flightdeck-logo.png)

# Flight Deck Solr

Flight Deck Solr is a minimalist Solr container for Drupal sites on Kubernetes and Docker. You can use it both for local development and production.

Features:
* ConfigMap-friendly YAML configuration
* Supports multiple cores
* Supports custom cores via ConfigMaps or volumes

## Tags and versions

There are several tags available for this container, each with different Solr and module support:

| Solr version | Tags | Search API | Apachesolr | Custom cores |
| ------------ | ---- | ---------- | ---------- | ------------ |
| 8.6.0 | 8.6, 8 | yes | no | yes |
| 6.6.6 | 6.6, 6, latest | yes | no | yes |
| 5.5.5 | 5.5, 5 | 7.x-1.x | 7.x-1.x | yes |


## Configuration

This container does not use environment variables for configuration. Instead, the `flight-deck-solr.yml` file is used to handle all configuration.

```yaml
---
flightdeck_solr:
  port: 8983
  cores: []
```

Where:
* **flightdeck_solr.port** is the port on which to run Solr. Optional, defailts to `8983`.
* **flightdeck_solr.cores** is a list of cores to create.

### Defining cores

This container supports multiple cores by defining an item under the `flightdeck_solr.cores` item:

```yaml
---
flightdeck_solr:
  port: 8983
  cores:
    - name: "my_custom_core"
      type: "custom"
      confDir: "/solr-conf"
      dataDir: "/solr-data"
```

Each item has the following variables:

* **name** is the name of the solr core. Required.
* **type** is the type of Solr core to create. Required, must be `custom` or `empty`.
* **confDir** is the path inside the container at which the schema files are mounted. Required for `custom` cores.
* **dataDir** is the path inside the container at which the core data is to be stored. Optional, defaults to `/data/&lt;name_of_the_core&gt;`.

### What about Search API?

Flight Deck Solr supports Search API as fully custom cores. This is due to changes in how Search API Solr works, which requires you to generate a config.zip under Admin &gt; Config &gt; Search and metadata &gt; Search API.

1. First, create an `empty` core using this container.
2. Generate the config.zip in Drupal.
3. Update your docker-compose.yml to mount the extracted *.zip at the path specified by `confDir`.
4. Update your flight-deck-solr.yml to be a `custom` core `type`.
5. Restart the solr container.

## Deployment on Kubernetes

Use the [`ten7.flightdeck_cluster`](https://galaxy.ansible.com/ten7/flightdeck_cluster) role on Ansible Galaxy to deploy Solr as a statefulset:

```yaml
flightdeck_cluster:
  namespace: "search"
  configMaps:
    - name: "solr_config"
      files:
        - name: "flight-deck-solr.yml"
          content: |
            flightdeck_solr:
              cores:
                - name: "my_custom_core"
                  confDir: "/config/my_custom_core"
  solr:
    configMaps:
      - name: "solr_config"
        path: "/config/my_custom_core"
```

## Using with Docker Compose

Create the `flight-deck-solr.yml` file relative to your `docker-compose.yml`. Define the `solr` service mounting the file as a volume:

```yaml
version: '3'
services:
  solr:
    image: ten7/flight-deck-solr
    volumes:
      - ./flight-deck-solr.yml:/config/solr/flight-deck-solr.yml
    ports:
      - "8003:8983"
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
