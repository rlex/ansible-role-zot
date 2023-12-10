- [Description](#description)
- [Requirements](#requirements)
- [Usage](#usage)
- [Default config](#default-config)
- [Variables](#variables)

## Description

Ansible role for installing [zot registy](https://zotregistry.io/).

## Requirements

This role should work on any linux distro with systemd.

## Usage

This role is pretty simple - basic variables control version of zot to install, user and group under which zot runs and similar parameters.<br>
Zot itself is configured with zot_registry_config variable, in form of yaml dict, which will be converted to json for zot consumption.<br>
By default, zot will be configured to serve on 8080 port, on all IPs, and mirroring most popular repositories, such as docker.io, ghcr.io and so on. <br>
Also, it will serve metrics on /metrics and ui on root path.<br>
You can apply any zot-compatible config, i.e. [from zot examples](https://github.com/project-zot/zot/tree/main/examples).<br>
Rewriting json to yaml is pretty straightforward and can be done with online tools such as [this](https://jsonformatter.org/json-to-yaml).<br>

## Default config
Default config should provide you with working installation of zot which will be suitable for acting as registry mirror.<br>
Default zot config which will get installed with this role, with some comments (this is in no way all possible options for zot, just default config):
```yaml
  distSpecVersion: 1.1.0-dev
  storage:
    #use tmp for storage. This folder will be created automatically
    rootDirectory: /tmp/zot
    #use deduplication
    dedupe: true
  http:
    #listen on all interfaces
    address: 0.0.0.0
    #listen on port 8080
    port: 8080
  log:
    #minimum log level - info
    level: info
  extensions:
    metrics:
      #enable prometheus metrics
      enable: true
      #...on path /metrics
      prometheus:
        path: /metrics
    search:
      enable: true
      #enable CVE scanning with trivy
      cve:
        updateInterval: 2h
    scrub:
      enable: true
      interval: 24h
    ui:
      #enable web interface
      enable: true
    sync:
      #enable repository mirroring
      enable: true
      #list of registries to mirror
      registries:
        - urls: https://index.docker.io
          content:
            destination: /docker.io
            prefix: "**"
          onDemand: true
          tlsVerify: true
        - urls: https://registry.gitlab.com
          content:
            destination: /registry.gitlab.com
            prefix: "**"
          onDemand: true
          tlsVerify: true
        - urls: https://ghcr.io
          content:
            destination: /ghcr.io
            prefix: "**"
          onDemand: true
          tlsVerify: true
        - urls: https://quay.io
          content:
            destination: /quay.io
            prefix: "**"
          onDemand: true
          tlsVerify: true
        - urls: https://gcr.io
          content:
            destination: /gcr.io
            prefix: "**"
          onDemand: true
          tlsVerify: true
        - urls: https://registry.k8s.io
          content:
            destination: /registry.k8s.io
            prefix: "**"
          onDemand: true
          tlsVerify: true
```


## Variables

| Variable                 | Format | Default value     | Description                         |
| ------------------------ | ------ | ----------------- | ----------------------------------- |
| zot_registry_version     | string | 2.0.0-rc7         | version of zot registry to install  |
| zot_registry_config      | dict   | see usage         | dict with zot main config file      |
| zot_user                 | string | zot               | system user for running zot         |
| zot_group                | string | alias to zot_user | system group for running zot        |
| zot_registry_credentials | dict   | empty             | credentials for accessing upstreams |
