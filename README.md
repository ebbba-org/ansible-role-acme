# ACME.SH - Ansible Role for requesting SSL/TLS certs

- [ACME.SH - Ansible Role for requesting SSL/TLS certs](#acmesh---ansible-role-for-requesting-ssltls-certs)
  - [Default Variables](#default-variables)
  - [Example Usage of this role](#example-usage-of-this-role)
  - [Playbook example](#playbook-example)
    - [Variables for apache2 with just one domain](#variables-for-apache2-with-just-one-domain)
    - [Variables for apache2 with multiple domains](#variables-for-apache2-with-multiple-domains)
    - [Variables to use DigitalOcean dnsapi certificate generation](#variables-to-use-digitalocean-dnsapi-certificate-generation)

## Default Variables

| **Variable**                       | **Default**                                          | **Description**                                                                                                             |
| ---------------------------------- | ---------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `acme_email`           | `acme@domain.tld`                                    | The mail to receive mails to                                                                                                |
| `acme_hostname`                    | `{{ inventory_hostname }}`                           | The default host name to acquire a cert for                                                                                 |
| `acme_api_url`             | `https://acme-v02.api.letsencrypt.org/directory`     | By default production, you might want to use `https://acme-staging-v02.api.letsencrypt.org/directory` for staging dev certs |
| `acme_git_repo`            | `https://github.com/acmesh-official/acme.sh.git`     | The repository where to acquire acme.sh from, in case you run a mirror / fork                                               |
| `acme_install_dir`             | `/opt/acme`                                          | Where acme.sh should be installed to                                                                                        |
| `acme_install_version`         | `master`                                             | Which git version / branch to checkout                                                                                      |
| `acme_install_keep_updated`    | `true`                                               | Update the git repository when re-running this role?                                                                        |
| `acme_letsencrypt_install_command` | [defaults/default.yaml#13](./defaults/main.yaml#L13) | The acme.sh install command for the local configuration for cert creating                                                   |
| `acme_letsencrypt_create_command`  | [defaults/default.yaml#21](./defaults/main.yaml#L13) | The acme.sh create cert command, here you can add the `--force` option to force a renewal                                   |
| `DO_API_KEY` | `NONE` | A DigitalOcean API key which is by default empty |

## Example Usage of this role

## Playbook example

```yaml
---
- name: Run ACME.SH - We need more SSL/TLS!
  hosts: all
  roles:
    - role: acme.sh
```

### Variables for apache2 with just one domain

```yaml
acme_letsencrypt_create_command: >-
  {{ acme_letsencrypt_script }} --issue
  --domain {{ acme_hostname }}
  --standalone
  --server letsencrypt
  --keylength ec-256
  --pre-hook "systemctl stop apache2.service"
  --post-hook "systemctl start apache2.service"
  --server {{ acme_api_url }}
```

### Variables for apache2 with multiple domains

```yaml
acme_letsencrypt_create_command: >-
  {{ acme_letsencrypt_script }} --issue
  --domain {{ acme_hostname }}
  --domain foo.bar.DOMAIN.TLD
  --domain bar.foo.DOMAIN.TLD
  --standalone
  --server letsencrypt
  --keylength ec-256
  --pre-hook "systemctl stop apache2.service"
  --post-hook "systemctl start apache2.service"
  --server {{ acme_api_url }}
```

### Variables to use DigitalOcean dnsapi certificate generation

More about acme.sh dnsapi => <https://github.com/acmesh-official/acme.sh/wiki/dnsapi>

```yaml
DO_API_KEY: "The Secret DigitalOcean API Token"

acme_letsencrypt_create_command: >-
  {{ acme_letsencrypt_script }} --issue
  --domain *.{{ acme_hostname }}
  --domain {{ acme_hostname }}
  --dns dns_dgon
  --server letsencrypt
  --keylength ec-256
  --post-hook "systemctl reload nginx.service"
  --server {{ acme_api_url }}
  --force
```
