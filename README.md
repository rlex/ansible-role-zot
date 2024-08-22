- [Description](#description)
- [Requirements](#requirements)
- [Usage](#usage)
- [Default config](#default-config)
- [Basic auth](#basic-auth)
- [Upstream repositories credentials](#upstream-repositories-credentials)
- [Variables](#variables)

## Description

Ansible role for installing [zot registy](https://zotregistry.dev/).

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
  distSpecVersion: 1.1.0
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

## Basic auth
Zot have support for [some auth methods](https://zotregistry.io/v1.4.3/articles/authn-authz/), including htpasswd (with basic auth)  
To generate zot-compatible htpasswd entry, run 
```shell
htpasswd -Bn username
```

And put generated line to zot_registry_htpasswd array: 
```yaml
zot_registry_htpasswd:
  - registry:$2y$05$SEz6.UxATCqZlx3K/fbkzubujuSoeaFYcCP5FTcqGkqOgaO0Kx/YO
```

It will put that entry (or entries) to /etc/zot/htpasswd file which you can then use for auth in config:
```yaml
http:
...
  auth:
    htpasswd:
      path: /etc/zot/htpasswd
}
```
Please note that auth is not enabled in default config, so you need to configure it manually

## Upstream repositories credentials
If you want to mirror upstream registries that require auth, you can use zot_registry_credentials to generate file for authorization:
```yaml
zot_registry_credentials:
  127.0.0.1:8008:
    username: user
    password: pass
  registry2:5000:
    username: user2
    password: pass2
```

It will be created at path /etc/zot/credentials.json, which you can then use in your config:
```yaml
extensions:
  sync:
    credentialsFile: /etc/zot/credentials.json
    registries: 
      urls: 
        - https://registy2:5000
...
```

It's probably good idea to encrypt this variable using ansible-vault or something similar.


## Variables

| Variable                 | Format | Default value     | Description                                        |
| ------------------------ | ------ | ----------------- | -------------------------------------------------- |
| zot_registry_version     | string | 2.0.0-rc7         | version of zot registry to install                 |
| zot_registry_config      | dict   | see usage         | dict with zot main config file                     |
| zot_user                 | string | zot               | system user for running zot                        |
| zot_group                | string | alias to zot_user | system group for running zot                       |
| zot_registry_htpasswd    | array  | `[]`              | users for zot basic auth                           |
| zot_registry_credentials | dict   | `{}`              | credentials for authorizing on upstream registries |
