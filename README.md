# Ansible Certbot Role

An Ansible role for installing Certbot and automatically requesting Let's Encrypt SSL certificates with support for multiple DNS providers and authentication methods.

## Description

This role automates the installation and configuration of Certbot, the official Let's Encrypt client. It supports multiple certificate acquisition methods including webroot validation and DNS challenges through various providers like DigitalOcean, GoDaddy, and AWS Route53.

## Features

- Installs Certbot and required plugins
- Supports multiple DNS providers for domain validation
- Automatic certificate renewal via systemd timer
- Certificate expansion for adding new domains
- Flexible plugin installation (package manager or PyPI)
- Automatic FQDN certificate generation option

## Requirements

- Ansible 2.9+
- Target systems: Linux distributions with systemd support
- For DNS challenges: appropriate DNS provider credentials

## Role Variables

### Required Variables

| Variable              | Description                                  | Default                        |
| --------------------- | -------------------------------------------- | ------------------------------ |
| `certbot_certs_email` | Email address for Let's Encrypt registration | `root@{{ ansible_inventory }}` |

### Certificate Configuration

| Variable                    | Description                              | Default |
| --------------------------- | ---------------------------------------- | ------- |
| `certbot_certs`             | List of certificates to request          | `[]`    |
| `certbot_request_fqdn_cert` | Automatically request cert for host FQDN | `true`  |

### Plugin Configuration

| Variable                         | Description                                              | Default        |
| -------------------------------- | -------------------------------------------------------- | -------------- |
| `certbot_plugins_source`         | Plugin installation source (`package_manager` or `pypi`) | `pypi`         |
| `certbot_plugins_package_prefix` | Prefix for plugin package names                          | `certbot-dns-` |

### DNS Provider Credentials

| Variable                     | Description                               | Default     |
| ---------------------------- | ----------------------------------------- | ----------- |
| `certbot_digitalocean_token` | DigitalOcean API token for DNS challenges | `undefined` |

### System Configuration

| Variable                | Description                       | Default         |
| ----------------------- | --------------------------------- | --------------- |
| `certbot_packages`      | Base certbot packages to install  | `[certbot]`     |
| `certbot_timer_service` | Systemd timer service for renewal | `certbot.timer` |

### Plugin Arguments

The `certbot_plugin_arguments` dictionary defines command-line arguments for different authentication methods:

```yaml
certbot_plugin_arguments:
  digitalocean: --dns-digitalocean --dns-digitalocean-credentials /root/do_secrets.ini
  godaddy: --authenticator dns-godaddy --dns-godaddy-credentials /root/gd_secrets.ini
  route53: --dns-route53
  default: "--webroot -w /var/www/acme-challenge"
```

## Usage Examples

### Basic Webroot Validation

```yaml
- hosts: webservers
  roles:
    - ansible-certbot
  vars:
    certbot_certs_email: admin@example.com
    certbot_certs:
      - hostname: example.com
        sans:
          - www.example.com
```

### DigitalOcean DNS Challenge

```yaml
- hosts: servers
  roles:
    - ansible-certbot
  vars:
    certbot_certs_email: admin@example.com
    certbot_digitalocean_token: "your_do_token_here"
    certbot_certs:
      - hostname: example.com
        plugin: digitalocean
        sans:
          - "*.example.com"
          - www.example.com
```

### Multiple Certificates with Different Plugins

```yaml
- hosts: servers
  roles:
    - ansible-certbot
  vars:
    certbot_certs_email: admin@example.com
    certbot_digitalocean_token: "your_do_token_here"
    certbot_certs:
      - hostname: api.example.com
        plugin: digitalocean
      - hostname: blog.example.com
        plugin: default  # Uses webroot
      - hostname: shop.example.com
        plugin: route53
        extra_arguments: "--dns-route53-propagation-seconds 60"
```

### Custom Plugin Installation

```yaml
- hosts: servers
  roles:
    - ansible-certbot
  vars:
    certbot_plugins_source: package_manager
    certbot_plugins_package_prefix: python3-certbot-dns-
    certbot_certs:
      - hostname: example.com
        plugin: cloudflare
```

## Certificate Configuration Format

Each certificate in the `certbot_certs` list should follow this format:

```yaml
certbot_certs:
  - hostname: primary.domain.com        # Required: Primary domain name
    plugin: digitalocean                # Optional: Authentication plugin
    sans:                              # Optional: Subject Alternative Names
      - www.primary.domain.com
      - alt.domain.com
    extra_arguments: "--key-type rsa"   # Optional: Additional certbot arguments
```

## Dependencies

- `community.crypto` collection (for certificate information parsing)

## License

MIT
