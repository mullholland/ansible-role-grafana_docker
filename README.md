# [Ansible role grafana_docker](#grafana_docker)

Installs and configures grafana container based on official grafana docker container

|GitHub|Downloads|Version|
|------|---------|-------|
|[![github](https://github.com/mullholland/ansible-role-grafana_docker/actions/workflows/molecule.yml/badge.svg)](https://github.com/mullholland/ansible-role-grafana_docker/actions/workflows/molecule.yml)|[![downloads](https://img.shields.io/ansible/role/d/mullholland/grafana_docker)](https://galaxy.ansible.com/mullholland/grafana_docker)|[![Version](https://img.shields.io/github/release/mullholland/ansible-role-grafana_docker.svg)](https://github.com/mullholland/ansible-role-grafana_docker/releases/)|
## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/mullholland/ansible-role-grafana_docker/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: true
  gather_facts: true
  vars:
    adguardhome_docker_config:
      tls:
        options:
          modern:
            minVersion: "VersionTLS13"
            sniStrict: true

  roles:
    - role: "mullholland.grafana_docker"
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/mullholland/ansible-role-grafana_docker/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: true
  gather_facts: true
  vars:
    pip_packages:
      - "docker"

  roles:
    - role: mullholland.docker
    - role: mullholland.repository_epel
    - role: mullholland.pip
```



## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/mullholland/ansible-role-grafana_docker/blob/master/defaults/main.yml):

```yaml
---
# General config
grafana_docker_network_name: "web"
grafana_docker_base_path: "/opt"
grafana_docker_timezone: "Europe/Berlin"

# User/Group of the stack. Everything is mapped to this, instead of root.
grafana_docker_user: "homelab"
grafana_docker_uid: "900"
grafana_docker_group: "homelab"
grafana_docker_gid: "900"
grafana_docker_user_system: true

# which container version to install
# https://hub.docker.com/r/grafana/grafana/tags
# can also be latest
grafana_docker_version: "grafana/grafana:latest"

# additional docker compose environment varaibles
# https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/#override-configuration-with-environment-variables
grafana_docker_environment_variables:
  - "GF_INSTALL_PLUGINS: 'grafana-clock-panel, grafana-simple-json-datasource'"
grafana_docker_volumes:
  - "/opt/grafana/data:/var/lib/grafana"
  - "/opt/grafana/provisioning:/etc/grafana/provisioning"

# which port to expose. can be empty
grafana_docker_ports:
  - "3000:3000/tcp"  # WebUI

grafana_docker_labels:
  - "traefik.enable=false"
# - "traefik.enable=true"
# - "traefik.http.routers.grafana.entryPoints=https"
# - "traefik.http.routers.grafana.rule=Host(`grafana.example.com`)"
# - "traefik.http.routers.grafana.tls.certResolver=letsEncrypt"
# - "traefik.http.routers.grafana.tls=true"
#  - "traefik.http.routers.grafana.service=grafana"
#  - "traefik.http.routers.grafana.loadbalancer.server.port=80"

grafana_docker_provision_datasources:
  - name: VictoriaMetrics
    type: prometheus
    access: proxy
    url: http://victoriametrics:8428
    isDefault: true
    jsonData:
      prometheusType: Prometheus
      prometheusVersion: 2.24.0
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    jsonData:
      timeout: 60
      maxLines: 10000
  - name: VMAlert
    type: alertmanager
    url: http://vmalert:8880
    access: proxy
    jsonData:
      # Valid options for implementation include mimir, cortex and prometheus
      implementation: prometheus
      # Whether or not Grafana should send alert instances to this Alertmanager
      handleGrafanaManagedAlerts: false

# Creates the provison config.
# You can put any dashboards json files into the provision folder outsie the container
# and grafnaa will automatically deploy it
grafana_docker_provision_dashboards:
  - name: dashboards
    type: file
    updateIntervalSeconds: 30
    options:
      path: /var/lib/provision/dashboards
      foldersFromFilesStructure: true
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/mullholland/ansible-role-grafana_docker/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | GitLab |
|-------------|--------|--------|
|[mullholland.repository_epel](https://galaxy.ansible.com/mullholland/repository_epel)|[![Build Status GitHub](https://github.com/mullholland/ansible-role-repository_epel/workflows/Ansible%20Molecule/badge.svg)](https://github.com/mullholland/ansible-role-repository_epel/actions)|[![Build Status GitLab](https://gitlab.com/opensourceunicorn/ansible-role-repository_epel/badges/master/pipeline.svg)](https://gitlab.com/opensourceunicorn/ansible-role-repository_epel)|
|[mullholland.docker](https://galaxy.ansible.com/mullholland/docker)|[![Build Status GitHub](https://github.com/mullholland/ansible-role-docker/workflows/Ansible%20Molecule/badge.svg)](https://github.com/mullholland/ansible-role-docker/actions)|[![Build Status GitLab](https://gitlab.com/opensourceunicorn/ansible-role-docker/badges/master/pipeline.svg)](https://gitlab.com/opensourceunicorn/ansible-role-docker)|
|[mullholland.pip](https://galaxy.ansible.com/mullholland/pip)|[![Build Status GitHub](https://github.com/mullholland/ansible-role-pip/workflows/Ansible%20Molecule/badge.svg)](https://github.com/mullholland/ansible-role-pip/actions)|[![Build Status GitLab](https://gitlab.com/opensourceunicorn/ansible-role-pip/badges/master/pipeline.svg)](https://gitlab.com/opensourceunicorn/ansible-role-pip)|

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://mullholland.net) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/mullholland/ansible-role-grafana_docker/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/mullholland):

|container|tags|
|---------|----|
|[EL](https://hub.docker.com/r/mullholland/enterpriselinux)|all|
|[Fedora](https://hub.docker.com/r/mullholland/fedora/)|38, 39|
|[Ubuntu](https://hub.docker.com/r/mullholland/ubuntu)|all|
|[Debian](https://hub.docker.com/r/mullholland/debian)|all|

The minimum version of Ansible required is 2.10, tests have been done to:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/mullholland/ansible-role-grafana_docker/issues).

## [License](#license)

[MIT](https://github.com/mullholland/ansible-role-grafana_docker/blob/master/LICENSE).

## [Author Information](#author-information)

[mullholland](https://mullholland.net)
