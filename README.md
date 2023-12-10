- [Description](#description)
- [Requirements](#requirements)
- [Usage](#usage)
- [Variables](#variables)

## Description

Ansible role for installing [zot registy](https://zotregistry.io/)

## Requirements

This role should work on any linux distro with systemd

## Usage

TBD

## Variables

| Variable                 | Format | Default value     | Description                         |
| ------------------------ | ------ | ----------------- | ----------------------------------- |
| zot_registry_version     | string | 2.0.0-rc7         | version of zot registry to install  |
| zot_registry_config      | dict   | see usage         | dict with zot main config file      |
| zot_user                 | string | zot               | system user for running zot         |
| zot_group                | string | alias to zot_user | system group for running zot        |
| zot_registry_credentials | dict   | empty             | credentials for accessing upstreams |
