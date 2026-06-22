# Ansible Role: Apache Tomcat

| Source                                                                                                                              | Version                                                                                                                                                                | CI                                                                                                                                                                                              | License                                                                                          |
| ----------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| [![Source Code](https://img.shields.io/badge/source-github-blue.svg)](https://github.com/grzegorzfranus/ansible-role-apache-tomcat) | [![Version](https://img.shields.io/github/v/release/grzegorzfranus/ansible-role-apache-tomcat)](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/releases) | [![CI](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/actions/workflows/ci.yml/badge.svg)](https://github.com/grzegorzfranus/ansible-role-apache-tomcat/actions/workflows/ci.yml) | [![Repository License](https://img.shields.io/badge/license-apache2.0-brightgreen.svg)](LICENSE) |

This Ansible role deploys Apache Tomcat using the **Enterprise Split Tomcat** architecture, separating binary files (`CATALINA_HOME`) from instance data (`CATALINA_BASE`). It provides production-grade security hardening, HTTPS with PKCS12 keystore, optional CI/CD deployer user, and a hardened systemd service with kernel-level isolation.

## ✨ Features

- 🏗️ **Split Tomcat Architecture**: Clean separation of `CATALINA_HOME` (binaries, root-owned) and `CATALINA_BASE` (instance data, mixed ownership) with symlink for easy upgrades
- 🔐 **HTTPS/SSL by Default**: PKCS12 keystore with TLSv1.2+1.3, strong cipher suite, and automatic self-signed certificate generation via `community.crypto` (no Java keytool dependency)
- 🛡️ **Security Hardening**: Removal of default apps, disabled autoDeploy/unpackWARs, hidden server version, custom error pages to prevent stack trace leaks
- 🔒 **Security Headers**: `HttpHeaderSecurityFilter` providing HSTS, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff` (when hardening enabled)
- ↗️ **Forced HTTPS Redirect**: Configurable `security-constraint` with `transport-guarantee CONFIDENTIAL`
- 🔑 **AJP Secret Support**: Ghostcat CVE-2020-1938 protection with configurable AJP secret requirement
- 🔀 **Reverse Proxy Ready**: Optional `RemoteIpValve` with configurable `X-Forwarded-For`/`X-Forwarded-Proto` headers and trusted proxy IP regex
- 👤 **Least Privilege**: Dedicated system user (`/sbin/nologin`), root-owned binaries and config, differentiated file permissions (dirs 755, files 644, scripts 755)
- 🚀 **Optional CI/CD Deployer**: Configurable deployment user with SSH keys, restricted sudoers (only `systemctl restart`), and webapps ownership with setgid
- 📊 **JMX Monitoring**: Secured JMX remote access with `.access` and `.password` files (chmod 400)
- ⚙️ **Hardened Systemd**: `Type=exec` foreground mode, `ProtectSystem=strict`, `PrivateTmp`, `NoNewPrivileges`, journal logging
- 📁 **Dedicated Logging**: Separate log directory (`/var/log/tomcat`) with systemd-managed logrotate and optional JVM GC logging
- 🔄 **Flexible Downloads**: Support for custom internal repositories (Nexus/Artifactory) or automatic Apache Archive URLs
- ☕ **Upgrade-Safe JAVA_HOME**: Runtime auto-detection via `readlink` — survives OpenJDK package upgrades without re-running Ansible
- ✅ **Full Parameterization**: Every value configurable through variables — zero hardcoded paths, ports, or credentials
- 🧹 **Optional Old Version Cleanup**: Automatic removal of outdated CATALINA_HOME directories with configurable retention count (`tomcat_keep_old_versions`) for controlled disk usage after upgrades
- 🏛️ **CIS Benchmark Compliance**: JVM security properties (RECYCLE_FACADES, STRICT_SERVLET_COMPLIANCE), configurable LockOutRealm, and managed `logging.properties` aligned with CIS Apache Tomcat 9 Benchmark v1.2

## 🎯 Architecture

```
/opt/tomcat/
├── apache-tomcat-9.0.118/     ← CATALINA_HOME (root:root, 755)
│   ├── bin/
│   ├── lib/
│   └── ...
├── current -> apache-tomcat-9.0.118  ← Symlink (easy upgrades)
└── instance/                  ← CATALINA_BASE (mixed ownership)
    ├── bin/                   ← setenv.sh (root:tomcat, 750)
    ├── conf/                  ← server.xml, web.xml, keystore.p12/.key/.crt (root:tomcat, 640)
    ├── lib/                   ← Additional JARs for this instance
    ├── logs -> /var/log/tomcat ← Symlink to dedicated log directory
    ├── temp/                  ← (tomcat:tomcat, 750)
    ├── webapps/               ← (root:tomcat or deployer:tomcat, 750)
    └── work/                  ← (tomcat:tomcat, 750)

/var/log/tomcat/               ← Dedicated log directory (tomcat:tomcat, 750)
```

### Ownership Model

| Directory             | Owner (default) | Owner (CI/CD)     | Permissions             |
| --------------------- | --------------- | ----------------- | ----------------------- |
| CATALINA_HOME (dirs)  | `root:root`     | `root:root`       | `755`                   |
| CATALINA_HOME (files) | `root:root`     | `root:root`       | `644` (scripts: `755`)  |
| `conf/`               | `root:tomcat`   | `root:tomcat`     | `640` (files)           |
| `webapps/`            | `root:tomcat`   | `deployer:tomcat` | `750` / `2770` (setgid) |
| `temp/`, `work/`      | `tomcat:tomcat` | `tomcat:tomcat`   | `750`                   |
| `/var/log/tomcat/`    | `tomcat:tomcat` | `tomcat:tomcat`   | `750`                   |

## 📋 Requirements

- **Ansible**: 2.16 or higher
- **Python**: 3.6 or higher on target hosts (see compatibility note below)
- **Java**: JDK/JRE 17+ must be pre-installed on target hosts (managed by a separate role)
- **Privileges**: sudo/root access on target hosts
- **Network**: Internet access for downloading Tomcat archive (or internal repository)
- **Collections**:
  - `community.crypto` >= 2.0.0 (for self-signed keystore generation via `openssl_privatekey`, `x509_certificate`, `openssl_pkcs12`)
  - `community.general` >= 6.0.0 (for `sefcontext` SELinux file context management on EL8/EL9)
  - `ansible.posix` (for `authorized_key` module when CI/CD is enabled)
- **Target host packages**: `python3-cryptography` (installed automatically by the role when self-signed keystore is generated)

### Supported operating systems

| OS Family                      | Version | Status                                               |
| ------------------------------ | ------- | ---------------------------------------------------- |
| EL (RHEL, Rocky, Alma, Oracle) | 9       | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| EL (RHEL, Rocky, Alma, Oracle) | 8       | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) * |

> **Note**: Tomcat is installed from the Apache tarball (not RPM packages), so the OS dependency is limited to systemd, user management, and Java availability.
>
> \* **EL8 Compatibility Constraints**:
> - **Ansible & Python compatibility**: EL8 defaults to Python 3.6. Support for target Python 3.6 was dropped in `ansible-core` >= 2.17.
>   - To run on EL8 with the system's default Python 3.6, you must use `ansible-core` <= 2.16.
>   - If running `ansible-core` >= 2.17, EL8 targets require Python >= 3.7. However, the system `python3-dnf` package manager bindings on EL8 are compiled exclusively for Python 3.6 and will not be available on newer Python interpreters. This will cause tasks using the `dnf` module (such as installing SELinux utilities or package updates) to fail.
> - **Molecule Testing**: Due to these Python version compatibility constraints, EL8 is not officially tested in the role's Molecule test suite (which runs a newer Ansible version in CI).

### Ansible version

Ansible >= 2.16

### Python version

Python >= 3.6 (with compatibility constraints on EL8)

### Setup module

The role uses facts gathered by Ansible on the remote host. If you disable the Setup module in your playbook, the role will not work properly.

### Root access

This role requires root access for directory creation, user management, systemd configuration, and file permission management. Tasks use `become: true` only where required (Least Privilege Execution).

## 🚀 Quick Start

### 1. Basic Installation

```yaml
---
- name: Deploy Apache Tomcat
  hosts: tomcat_servers
  roles:
    - role: grzegorzfranus.apache_tomcat
```

### 2. Run the playbook

```bash
ansible-playbook -i inventory tomcat-deploy.yml
```

This will install Tomcat 9.0.118 with:

- Self-signed HTTPS keystore on port 8443
- Hardening enabled (default apps removed, autoDeploy/unpackWARs disabled)
- HTTP (8080) and AJP (8009) connectors disabled
- Dedicated logging to `/var/log/tomcat/`

## ⚙️ Configuration

### Default Configuration

The role comes with secure, production-ready defaults:

```yaml
# Tomcat version
tomcat_version: "9.0.118"

# HTTPS on port 8443 with self-signed certificate
tomcat_https_port: 8443
tomcat_ssl_generate_self_signed: true

# Hardening enabled
tomcat_enable_hardening: true

# HTTP and AJP disabled
tomcat_enable_http: false
tomcat_enable_ajp: false
```

### Advanced Configuration — CI/CD with Custom Repository

```yaml
---
- name: Deploy Tomcat with CI/CD deployer
  hosts: tomcat_servers
  vars:
    # Download from internal Nexus
    tomcat_custom_download_url: "https://nexus.example.com/repository/apache/tomcat/apache-tomcat-9.0.118.tar.gz"

    # Custom keystore from PKI
    tomcat_ssl_keystore_src: "files/production-keystore.p12"
    tomcat_ssl_keystore_password: "{{ vault_tomcat_keystore_password }}"
    tomcat_ssl_generate_self_signed: false

    # Enable CI/CD deployer
    tomcat_enable_cicd: true
    tomcat_deployer_user: "deploy-svc"
    tomcat_deployer_uid: 910
    tomcat_deployer_ssh_keys:
      - "ssh-rsa AAAAB3... jenkins@ci-server"

    # JVM tuning
    tomcat_jvm_heap_min: "1g"
    tomcat_jvm_heap_max: "2g"
    tomcat_jvm_metaspace_max: "512m"

    # Enable JMX monitoring
    tomcat_enable_jmx: true
    tomcat_jmx_password: "{{ vault_tomcat_jmx_password }}"

  roles:
    - role: grzegorzfranus.apache_tomcat
```

### Minimal Configuration — Hardening Disabled

```yaml
---
- name: Deploy Tomcat (development)
  hosts: dev_servers
  vars:
    tomcat_enable_hardening: false
    tomcat_unpack_wars: true
    tomcat_auto_deploy: true
    tomcat_enable_http: true
  roles:
    - role: grzegorzfranus.apache_tomcat
```

## 📊 Variables

### General Settings

| Variable               | Description                                                            | Default     |
| ---------------------- | ---------------------------------------------------------------------- | ----------- |
| `tomcat_version`       | Version of Apache Tomcat to download and install                       | `"9.0.118"` |
| `tomcat_major_version` | Major version number used to construct the Apache Archive download URL | `"9"`       |

### Users & Groups

| Variable       | Description                                                                   | Default    |
| -------------- | ----------------------------------------------------------------------------- | ---------- |
| `tomcat_user`  | Name of the system user that runs the Tomcat process (shell: `/sbin/nologin`) | `"tomcat"` |
| `tomcat_group` | Name of the system group for the Tomcat process                               | `"tomcat"` |
| `tomcat_uid`   | UID for the Tomcat system user                                                | `900`      |
| `tomcat_gid`   | GID for the Tomcat system group                                               | `900`      |

### CI/CD Deployer

| Variable                           | Description                                                              | Default             |
| ---------------------------------- | ------------------------------------------------------------------------ | ------------------- |
| `tomcat_enable_cicd`               | Enable creation of a dedicated deployment user with SSH keys and sudoers | `false`             |
| `tomcat_deployer_user`             | Name of the deployment user                                              | `"deployer"`        |
| `tomcat_deployer_uid`              | UID for the deployment user                                              | `901`               |
| `tomcat_deployer_ssh_keys`         | List of public SSH keys for the deployer's `authorized_keys`             | `[]`                |
| `tomcat_deployer_sudoers_filename` | Filename for the sudoers drop-in in `/etc/sudoers.d/`                    | `"tomcat-deployer"` |

### Paths

| Variable               | Description                                                | Default                                                         |
| ---------------------- | ---------------------------------------------------------- | --------------------------------------------------------------- |
| `tomcat_install_dir`   | Parent directory for all Tomcat files                      | `"/opt/tomcat"`                                                 |
| `tomcat_catalina_home` | Full path to CATALINA_HOME (binary distribution)           | `"{{ tomcat_install_dir }}/apache-tomcat-{{ tomcat_version }}"` |
| `tomcat_symlink_path`  | Path to the symlink pointing to the current Tomcat version | `"{{ tomcat_install_dir }}/current"`                            |
| `tomcat_catalina_base` | Full path to CATALINA_BASE (instance data)                 | `"{{ tomcat_install_dir }}/instance"`                           |
| `tomcat_log_dir`       | Dedicated log directory for Tomcat                         | `"/var/log/tomcat"`                                             |

### Download

| Variable                     | Description                                                                        | Default  |
| ---------------------------- | ---------------------------------------------------------------------------------- | -------- |
| `tomcat_custom_download_url` | Full URL to a custom `.tar.gz` archive (Nexus/Artifactory). Empty = Apache Archive | `""`     |
| `tomcat_download_checksum`   | Optional checksum for the archive (format: `sha512:HASH`)                          | `""`     |
| `tomcat_download_timeout`    | Download timeout in seconds                                                        | `60`     |
| `tomcat_download_tmp_dir`    | Temporary directory for the downloaded archive                                     | `"/tmp"` |

### Java

| Variable           | Description                                                                       | Default |
| ------------------ | --------------------------------------------------------------------------------- | ------- |
| `tomcat_java_home` | Path to JAVA_HOME. Leave empty for auto-detection via `readlink -f $(which java)` | `""`    |

### JVM & CATALINA_OPTS

| Variable                   | Description                                                                           | Default    |
| -------------------------- | ------------------------------------------------------------------------------------- | ---------- |
| `tomcat_jvm_heap_min`      | Minimum JVM heap size (`-Xms`)                                                        | `"512m"`   |
| `tomcat_jvm_heap_max`      | Maximum JVM heap size (`-Xmx`)                                                        | `"1024m"`  |
| `tomcat_jvm_metaspace_max` | Maximum JVM Metaspace size (`-XX:MaxMetaspaceSize`)                                   | `"256m"`   |
| `tomcat_java_opts_extra`   | Additional JVM flags appended to CATALINA_OPTS (e.g. `-Dspring.profiles.active=prod`) | `""`       |
| `tomcat_enable_gc_logging` | Enable JVM Garbage Collection logging (JVM 11+ Unified Logging)                       | `false`    |
| `tomcat_gc_log_opts`       | GC logging JVM flags appended to CATALINA_OPTS when enabled                           | (computed) |

### Network & SSL

| Variable                              | Description                                                                                          | Default       |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------- | ------------- |
| `tomcat_enable_ssl`                   | Enable SSL/HTTPS connector and keystore management                                                   | `true`        |
| `tomcat_https_port`                   | HTTPS connector listening port                                                                       | `8443`        |
| `tomcat_https_address`                | HTTPS connector listen address                                                                       | `"0.0.0.0"`   |
| `tomcat_shutdown_port`                | Shutdown port (`-1` disables for security)                                                           | `-1`          |
| `tomcat_enable_http`                  | Enable the HTTP connector                                                                            | `false`       |
| `tomcat_http_port`                    | HTTP connector port (when enabled)                                                                   | `8080`        |
| `tomcat_http_address`                 | HTTP connector listen address                                                                        | `"0.0.0.0"`   |
| `tomcat_enable_ajp`                   | Enable the AJP connector                                                                             | `false`       |
| `tomcat_ajp_port`                     | AJP connector port (when enabled)                                                                    | `8009`        |
| `tomcat_ajp_address`                  | AJP connector listen address                                                                         | `"127.0.0.1"` |
| `tomcat_server_header`                | Value of the `server` attribute in connectors (hides Tomcat version)                                 | `"AppServer"` |
| `tomcat_connector_max_threads`        | Maximum worker threads per connector (controls concurrency)                                          | `200`         |
| `tomcat_connector_connection_timeout` | Connection timeout in ms (mitigates slowloris)                                                       | `20000`       |
| `tomcat_connector_enable_lookups`     | Enable DNS reverse lookups on requests                                                               | `false`       |
| `tomcat_connector_uri_encoding`       | URI encoding for request URLs                                                                        | `"UTF-8"`     |
| `tomcat_deploy_on_startup`            | Deploy webapps found in `webapps/` on startup                                                        | `false`       |
| `tomcat_ajp_secret_required`          | Whether AJP connector requires a secret (CVE-2020-1938)                                              | `true`        |
| `tomcat_ajp_secret`                   | AJP connector secret (override with Ansible Vault!)                                                  | `""`          |
| `tomcat_force_https`                  | Force HTTP→HTTPS redirect via `RewriteValve` (requires `tomcat_enable_ssl` and `tomcat_enable_http`) | `true`        |

### Reverse Proxy

| Variable                        | Description                                                     | Default                  |
| ------------------------------- | --------------------------------------------------------------- | ------------------------ |
| `tomcat_behind_proxy`           | Enable `RemoteIpValve` for Tomcat behind a reverse proxy        | `false`                  |
| `tomcat_proxy_internal_proxies` | Java regex of trusted proxy IP addresses (dots must be escaped) | (RFC1918 private ranges) |
| `tomcat_proxy_remote_ip_header` | HTTP header containing the original client IP                   | `"X-Forwarded-For"`      |
| `tomcat_proxy_protocol_header`  | HTTP header containing the original protocol (http/https)       | `"X-Forwarded-Proto"`    |
| `tomcat_proxy_log_proxy_ip`     | Whether to log the immediate proxy IP alongside the client IP   | `false`                  |

### SSL Keystore

| Variable                          | Description                                                           | Default                                                    |
| --------------------------------- | --------------------------------------------------------------------- | ---------------------------------------------------------- |
| `tomcat_ssl_keystore_file`        | Destination path for the keystore on the target server                | `"{{ tomcat_catalina_base }}/conf/keystore.p12"`           |
| `tomcat_ssl_keystore_password`    | Keystore password (override with Ansible Vault!)                      | `"changeit"`                                               |
| `tomcat_ssl_keystore_type`        | Keystore type (`PKCS12` or `JKS`)                                     | `"PKCS12"`                                                 |
| `tomcat_ssl_key_alias`            | Key alias within the keystore                                         | `"tomcat"`                                                 |
| `tomcat_ssl_protocol`             | SSL protocol                                                          | `"TLS"`                                                    |
| `tomcat_ssl_enabled_protocols`    | Allowed TLS protocol versions                                         | `"TLSv1.2+TLSv1.3"`                                        |
| `tomcat_ssl_ciphers`              | Strong cipher suite for TLS connections                               | (see defaults)                                             |
| `tomcat_ssl_keystore_src`         | Path on control node to a custom `.p12` file. Empty = use self-signed | `""`                                                       |
| `tomcat_ssl_generate_self_signed` | Generate a self-signed keystore when no custom keystore is provided   | `true`                                                     |
| `tomcat_ssl_self_signed_dname`    | Distinguished Name for the self-signed certificate                    | `"CN=localhost,OU=IT,O=Organization,L=City,ST=State,C=US"` |
| `tomcat_ssl_self_signed_validity` | Validity period of the self-signed certificate in days                | `365`                                                      |

### JMX

| Variable                   | Description                                 | Default                                                |
| -------------------------- | ------------------------------------------- | ------------------------------------------------------ |
| `tomcat_enable_jmx`        | Enable secured JMX remote monitoring        | `false`                                                |
| `tomcat_jmx_port`          | JMX registry port                           | `9090`                                                 |
| `tomcat_jmx_rmi_port`      | JMX RMI server port                         | `9091`                                                 |
| `tomcat_jmx_address`       | JMX RMI server hostname / listen address    | `"localhost"`                                          |
| `tomcat_jmx_role`          | JMX role name (`readonly` or `readwrite`)   | `"readonly"`                                           |
| `tomcat_jmx_password`      | JMX password (override with Ansible Vault!) | `"changeit"`                                           |
| `tomcat_jmx_access_file`   | Path to the JMX access file                 | `"{{ tomcat_catalina_base }}/conf/jmxremote.access"`   |
| `tomcat_jmx_password_file` | Path to the JMX password file               | `"{{ tomcat_catalina_base }}/conf/jmxremote.password"` |

### Hardening

| Variable                        | Description                                                | Default                                                   |
| ------------------------------- | ---------------------------------------------------------- | --------------------------------------------------------- |
| `tomcat_enable_hardening`       | Enable security hardening                                              | `true`                                                    |
| `tomcat_default_apps_to_remove` | List of default apps to remove from CATALINA_HOME/webapps/             | `["ROOT", "manager", "host-manager", "docs", "examples"]` |
| `tomcat_unpack_wars`            | `unpackWARs` attribute in the Host element of server.xml               | `false`                                                   |
| `tomcat_auto_deploy`            | `autoDeploy` attribute in the Host element of server.xml               | `false`                                                   |
| `tomcat_lockout_failure_count`  | Max consecutive failed auth attempts before lockout (CIS 5.2)          | `5`                                                       |
| `tomcat_lockout_time`           | Lockout duration in seconds after exceeding failure count (CIS 5.2)    | `300`                                                     |

### Systemd

| Variable                      | Description                                       | Default    |
| ----------------------------- | ------------------------------------------------- | ---------- |
| `tomcat_service_name`         | Name of the systemd service unit                  | `"tomcat"` |
| `tomcat_service_enabled`      | Enable automatic service start on system boot     | `true`     |
| `tomcat_systemd_limit_nofile` | Maximum open file descriptors (`LimitNOFILE`)     | `65536`    |
| `tomcat_systemd_restart_sec`  | Delay before restart after failure (`RestartSec`) | `10`       |
| `tomcat_systemd_timeout_stop_sec` | Timeout before sending SIGKILL (`TimeoutStopSec`) | `90`       |
| `tomcat_systemd_standard_output` | Destination for systemd standard output (stdout) | `"append:{{ tomcat_log_dir }}/catalina.out"` |
| `tomcat_systemd_standard_error`  | Destination for systemd standard error (stderr) | `"append:{{ tomcat_log_dir }}/catalina.out"` |
| `tomcat_systemd_extra_read_write_paths` | List of additional custom paths to allow Tomcat write access (appended to `ReadWritePaths`) | `[]` |

> **Note**: On EL8/EL9 with SELinux enforcing, the role automatically installs a custom SELinux policy module (`tomcat_systemd`) that allows `init_t` to write to `var_log_t` directories and execute files under `/opt/tomcat/`. No manual SELinux configuration is required.

### Logrotate

| Variable                                          | Description                                                       | Default                     |
| ------------------------------------------------- | ----------------------------------------------------------------- | --------------------------- |
| `tomcat_configure_logrotate`                      | Whether to configure logrotate for Tomcat logs                    | `true`                      |
| `tomcat_logrotate_options.archive_directory_path` | Path to the archive directory for rotated logs (used as `olddir`) | `"/var/log/tomcat/archive"` |
| `tomcat_logrotate_options.frequency`              | Rotation frequency (`hourly`, `daily`, `weekly`, `monthly`)       | `"daily"`                   |
| `tomcat_logrotate_options.count`                  | Number of rotated log files to retain                             | `14`                        |
| `tomcat_logrotate_options.missingok`              | Do not issue an error if log file is missing                      | `true`                      |
| `tomcat_logrotate_options.compress`               | Compress rotated files using gzip                                 | `true`                      |
| `tomcat_logrotate_options.nocreate`               | Do not create new empty log files after rotation                  | `true`                      |
| `tomcat_logrotate_options.copytruncate`           | Use copytruncate (Tomcat holds open FDs)                          | `true`                      |
| `tomcat_logrotate_options.dateext`                | Use date extension in rotated file names                          | `true`                      |

> **Note**: All log files use fixed names without date suffixes. JULI and AccessLogValve have `rotatable=false`. Logrotate is the single rotation mechanism, rotating files to `archive/` with `dateext` and `compress`. This ensures stable file paths for log aggregators (Filebeat, Splunk).

### Upgrade

| Variable                      | Description                                                                                                                                         | Default |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| `tomcat_cleanup_old_versions` | Remove old Tomcat versions from installation directory after upgrade. Current version is always kept.                                               | `true`  |
| `tomcat_keep_old_versions`    | Number of previous Tomcat versions to keep in the installation directory for rollback. Only effective when `tomcat_cleanup_old_versions` is `true`. | `2`     |

### Logging

| Variable                   | Description                                                                          | Default     |
| -------------------------- | ------------------------------------------------------------------------------------ | ----------- |
| `tomcat_configure_logging`  | Template a managed `logging.properties` into `CATALINA_BASE/conf/` (CIS 7.2)       | `true`      |
| `tomcat_logging_level`      | Default log level for Tomcat internals (`SEVERE`/`WARNING`/`INFO`/`CONFIG`/`FINE`/`FINER`/`FINEST`) | `"INFO"` |

## 📌 Role Properties

| Property         | Value        | Description                                                                                                                                                |
| ---------------- | ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Idempotent**   | ✅ Yes       | Running the role multiple times produces the same result. Archive download is skipped if CATALINA_HOME exists. Self-signed keystore uses stat check guard. |
| **Atomic**       | ❌ No        | A failure mid-execution may leave the system in an intermediate state (e.g., binaries extracted but config not templated).                                 |
| **Check Mode**   | ✅ Supported | All tasks work in check mode. `community.crypto` modules support check mode natively.                                                                      |
| **Diff Mode**    | ✅ Supported | Template and `community.crypto` tasks support diff mode for change preview.                                                                                |
| **Upgrade-Safe** | ✅ Yes       | Changing `tomcat_version` and re-running the role upgrades Tomcat without affecting deployed applications.                                                 |

## 📤 Role Output

This role does not set any public output facts. All internal facts use the `__tomcat_` double-underscore prefix and are not part of the public interface.

## 🔍 Verification

After deployment, verify Tomcat is working correctly:

### Check Service Status

```bash
sudo systemctl status tomcat

# Expected output: active (running)
```

### Test HTTPS Connectivity

```bash
# Test with self-signed certificate (skip verification)
curl -k https://localhost:8443/

# Check certificate details
openssl s_client -connect localhost:8443 -showcerts </dev/null 2>/dev/null | openssl x509 -text -noout
```

### Verify Directory Structure

```bash
# Check symlink
ls -la /opt/tomcat/current

# Check CATALINA_BASE permissions
ls -la /opt/tomcat/instance/

# Check config permissions (should be 640)
ls -la /opt/tomcat/instance/conf/

# Check log directory
ls -la /var/log/tomcat/
```

### Verify Hardening

```bash
# Default apps should be removed
ls /opt/tomcat/current/webapps/
# Expected: empty or only your deployed applications

# Check server header (should show "AppServer", not "Apache-Coyote")
curl -kI https://localhost:8443/ | grep Server

# Check security headers (when hardening enabled)
curl -kI https://localhost:8443/ | grep -E 'Strict-Transport|X-Frame|X-Content-Type'
# Expected: Strict-Transport-Security, X-Frame-Options: DENY, X-Content-Type-Options: nosniff

# Verify reverse proxy valve (when tomcat_behind_proxy: true)
grep RemoteIpValve /opt/tomcat/instance/conf/server.xml
```

## 🔄 Upgrade Procedure

Upgrading Apache Tomcat requires only a version variable change — the Split Tomcat architecture ensures zero impact on deployed applications.

### Upgrade Steps

1. **Update version** in your inventory or playbook variables:

   ```yaml
   tomcat_version: "9.0.120" # was "9.0.118"
   ```

2. **(Optional)** Add checksum for the new archive:

   ```yaml
   tomcat_download_checksum: "sha512:abc123..."
   ```

3. **Run the playbook**:
   ```bash
   ansible-playbook -i inventory tomcat-deploy.yml
   ```

### What Happens During Upgrade

| Step | Action                                   | Impact on Applications |
| ---- | ---------------------------------------- | ---------------------- |
| 1    | Check if new CATALINA_HOME exists        | None                   |
| 2    | Download new Tomcat archive              | None                   |
| 3    | Extract to new versioned directory       | None                   |
| 4    | Set ownership and permissions            | None                   |
| 5    | Update symlink `current → new version`   | None                   |
| 6    | Re-render configuration in CATALINA_BASE | None                   |
| 7    | Clean up old versions (if enabled)       | None                   |
| 8    | Restart systemd service                  | Brief downtime         |

### Upgrade Safety Guarantees

- ✅ **Applications survive**: `webapps/` is in CATALINA_BASE, never touched during upgrade
- ✅ **Configuration preserved**: `conf/` is re-rendered from templates (always current)
- ✅ **Rollback ready**: Previous CATALINA_HOME directories remain on disk (configurable retention)
- ✅ **Keystore preserved**: SSL keystore in CATALINA_BASE is not regenerated if it exists

### Old Version Cleanup

By default, old Tomcat versions remain on disk for rollback. Enable automatic cleanup:

```yaml
tomcat_cleanup_old_versions: true
tomcat_keep_old_versions: 3 # keep 3 previous versions for rollback
```

The current version is **always preserved** — `tomcat_keep_old_versions` controls how many _previous_ versions remain.

### Manual Rollback

1. Stop the service: `sudo systemctl stop tomcat`
2. Update `tomcat_version` to the previous version
3. Re-run the role — it will update the symlink and restart

## 🛡️ Security Features

- ✅ **Split Architecture**: Binaries owned by root, Tomcat process has read-only access
- ✅ **HTTPS Only**: HTTP and AJP disabled by default, PKCS12 keystore with strong TLS
- ✅ **Keystore via `community.crypto`**: Self-signed keystore generated using native Ansible modules (no shell commands, full idempotency)
- ✅ **SSL Cleanup**: Temporary `.key`/`.crt`/`.csr` files removed after keystore export (defense-in-depth)
- ✅ **JAVA_HOME Auto-Detection**: Runtime detection via `readlink` — survives OpenJDK package upgrades
- ✅ **Server Version Hidden**: `server="AppServer"` attribute masks Tomcat identity
- ✅ **Custom Error Pages**: Global error pages prevent Java stack trace leaks
- ✅ **Security Headers**: `HttpHeaderSecurityFilter` providing HSTS, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`
- ✅ **Forced HTTPS Redirect**: `security-constraint` with `transport-guarantee CONFIDENTIAL`
- ✅ **AJP Secret**: Ghostcat CVE-2020-1938 protection — `secretRequired=true` by default
- ✅ **Reverse Proxy Awareness**: `RemoteIpValve` with configurable trusted proxies and `X-Forwarded-*` headers
- ✅ **Shutdown Port Disabled**: `shutdown_port=-1` prevents remote shutdown attacks
- ✅ **Systemd Hardening**: `Type=exec`, `ProtectSystem=strict`, `PrivateTmp`, `NoNewPrivileges`, journal logging
- ✅ **SELinux Support**: Automatic file context labeling (`tomcat_log_t`, `tomcat_exec_t`, `tomcat_var_lib_t`) on EL8/EL9 with SELinux enforcing — no manual `audit2allow` required
- ✅ **JMX Security**: Access and password files with `chmod 400`
- ✅ **Ansible Vault**: All passwords (`keystore`, `JMX`, `AJP secret`) designed for Vault override
- ✅ **no_log**: Sensitive tasks (keystore generation, JMX files, SSH keys, AJP secret) use `no_log: true`
- ✅ **Least Privilege Execution**: `become: true` only on tasks that require root
- ✅ **Old Version Cleanup**: Optional disk cleanup of outdated CATALINA_HOME directories with configurable retention (`tomcat_keep_old_versions`)
- ✅ **CIS 10.3 RECYCLE_FACADES**: Prevents request facade object reuse between requests (information leakage mitigation)
- ✅ **CIS 10.4 STRICT_SERVLET_COMPLIANCE**: Enforces strict Servlet specification compliance
- ✅ **CIS 10.5 Path Traversal Protection**: Explicitly blocks backslash and encoded slash path traversal
- ✅ **CIS 5.2 LockOutRealm**: Configurable brute-force protection with `failureCount` and `lockOutTime`
- ✅ **CIS 7.2 Managed Logging**: Production-hardened `logging.properties` with `AsyncFileHandler`, configurable log levels, and auto-cleanup via `maxDays`

### Uninstall

This role does not include a built-in uninstall mechanism. To remove Tomcat:

```bash
sudo systemctl stop tomcat
sudo systemctl disable tomcat
sudo rm /etc/systemd/system/tomcat.service
sudo systemctl daemon-reload
sudo userdel -r tomcat
sudo groupdel tomcat
sudo rm -rf /opt/tomcat /var/log/tomcat /etc/logrotate.d/tomcat /etc/sudoers.d/tomcat-deployer
```

### Roll-back Capabilities

This role **does not** support automatic rollback. To revert to a previous version:

1. Stop the service: `sudo systemctl stop tomcat`
2. Update `tomcat_version` to the previous version
3. Re-run the role — it will download the older tarball and update the symlink
4. The previous CATALINA_HOME directory remains intact for manual rollback

## Example Playbooks

### Basic Installation

```yaml
---
- name: Deploy Apache Tomcat (defaults)
  hosts: tomcat_servers
  roles:
    - role: grzegorzfranus.apache_tomcat
```

### Production with CI/CD and Custom Keystore

```yaml
---
- name: Deploy Tomcat for Production
  hosts: production_servers
  vars:
    # Custom keystore from PKI
    tomcat_ssl_keystore_src: "files/production.p12"
    tomcat_ssl_keystore_password: "{{ vault_keystore_password }}"
    tomcat_ssl_generate_self_signed: false

    # CI/CD deployer
    tomcat_enable_cicd: true
    tomcat_deployer_ssh_keys:
      - "ssh-rsa AAAAB3... jenkins@build-server"

    # JVM tuning for production
    tomcat_jvm_heap_min: "2g"
    tomcat_jvm_heap_max: "4g"

    # JMX monitoring
    tomcat_enable_jmx: true
    tomcat_jmx_password: "{{ vault_jmx_password }}"

  roles:
    - role: grzegorzfranus.apache_tomcat
```

### Development Environment

```yaml
---
- name: Deploy Tomcat for Development
  hosts: dev_servers
  vars:
    tomcat_enable_hardening: false
    tomcat_unpack_wars: true
    tomcat_auto_deploy: true
    tomcat_enable_http: true
    tomcat_force_https: false
    tomcat_jvm_heap_max: "512m"

  roles:
    - role: grzegorzfranus.apache_tomcat
```

### Behind Reverse Proxy (SSL Termination at Proxy)

```yaml
---
- name: Deploy Tomcat behind Nginx (SSL terminated at proxy)
  hosts: app_servers
  vars:
    # Proxy mode — Tomcat receives plain HTTP from Nginx
    tomcat_behind_proxy: true
    tomcat_proxy_internal_proxies: "192\\.0\\.2\\.1" # Nginx server IP
    tomcat_enable_http: true

    # SSL disabled — handled by proxy, not Tomcat
    tomcat_enable_ssl: false
    tomcat_force_https: false

  roles:
    - role: grzegorzfranus.apache_tomcat
```

### Behind Reverse Proxy (End-to-End SSL)

```yaml
---
- name: Deploy Tomcat behind Nginx (end-to-end SSL / re-encrypt)
  hosts: app_servers
  vars:
    # Proxy mode — Nginx forwards to Tomcat HTTPS
    tomcat_behind_proxy: true
    tomcat_proxy_internal_proxies: "192\\.0\\.2\\.1"

    # Tomcat terminates its own SSL
    tomcat_ssl_keystore_src: "files/app.p12"
    tomcat_ssl_keystore_password: "{{ vault_keystore_password }}"
    tomcat_ssl_generate_self_signed: false
    tomcat_force_https: true

  roles:
    - role: grzegorzfranus.apache_tomcat
```

### Custom Internal Repository (Nexus)

```yaml
---
- name: Deploy Tomcat from Nexus
  hosts: internal_servers
  vars:
    tomcat_custom_download_url: "https://nexus.internal.example.com/repository/apache-dist/tomcat/apache-tomcat-9.0.118.tar.gz"
    tomcat_download_checksum: "sha512:abc123..."

  roles:
    - role: grzegorzfranus.apache_tomcat
```

## 📁 File Structure

```
ansible-role-apache-tomcat/
├── .github/workflows/
│   ├── ci.yml                         # CI pipeline
│   └── release.yml                    # Release Please + Galaxy publish
├── .ansible-lint                      # Ansible lint configuration
├── .gitignore                         # Git ignore rules
├── .release-please-manifest.json      # Release Please version manifest
├── .yamllint                          # YAML lint configuration
├── CHANGELOG.md                       # Version history
├── LICENSE                            # Apache-2.0 license
├── README.md                          # This documentation
├── release-please-config.json         # Release Please configuration
├── defaults/
│   └── main.yml                       # Default variables (15 sections)
├── handlers/
│   └── main.yml                       # Daemon-reload and service restart
├── meta/
│   └── main.yml                       # Galaxy metadata
├── molecule/
│   ├── default/                       # Default test scenario
│   │   ├── molecule.yml
│   │   ├── converge.yml
│   │   ├── prepare.yml
│   │   └── verify.yml
│   ├── cicd/                          # CI/CD deployer test scenario
│   │   ├── molecule.yml
│   │   ├── converge.yml
│   │   ├── prepare.yml
│   │   └── verify.yml
│   └── upgrade/                       # Upgrade test scenario
│       ├── molecule.yml
│       ├── converge.yml
│       ├── prepare.yml
│       └── verify.yml
├── tasks/
│   ├── main.yml                       # Task orchestration (include_vars → assert → ... → cleanup)
│   ├── assert.yml                     # Variable validation (all defaults verified)
│   ├── java_detect.yml                # JAVA_HOME auto-detection
│   ├── users.yml                      # User and group creation
│   ├── install.yml                    # Download, extract, symlink
│   ├── directories.yml                # Split Tomcat directory structure
│   ├── config.yml                     # Configuration templates and keystore (community.crypto)
│   ├── selinux.yml                    # SELinux file context labeling (EL8/EL9)
│   ├── systemd.yml                    # Systemd service unit
│   ├── logrotate.yml                  # Log rotation configuration
│   ├── sudoers.yml                    # CI/CD deployer sudoers
│   └── cleanup.yml                    # Old version cleanup (optional)
├── templates/
│   ├── jmx/
│   │   ├── jmxremote.access.j2       # JMX access control
│   │   └── jmxremote.password.j2     # JMX password file
│   ├── logrotate/
│   │   └── tomcat.j2                 # Log rotation configuration
│   ├── systemd/
│   │   └── tomcat.service.j2         # Hardened systemd unit
│   └── tomcat/
│       ├── logging.properties.j2      # JULI logging configuration (CIS 7.2)
│       ├── rewrite.config.j2          # Rewrite rules (method blocking, HTTPS redirect)
│       ├── server.xml.j2              # Tomcat server configuration
│       ├── setenv.sh.j2               # JVM options and JMX settings
│       ├── tomcat-users.xml.j2        # User database (empty by default)
│       └── web.xml.j2                 # Global web.xml with error pages
└── vars/
    └── main.yml                       # Internal variables (__tomcat_ prefix)
```

## 🤝 Contributing

Contributions, bug reports, and feature requests are welcome!

- Fork the repository and create your branch from `main`
- Use [Conventional Commits](https://www.conventionalcommits.org/) for commit messages:
  - `feat:` — new features (minor version bump)
  - `fix:` — bug fixes (patch version bump)
  - `docs:` — documentation changes
  - `refactor:` — code refactoring
  - `test:` — test additions
  - `ci:` — CI/CD changes
  - `chore:` — maintenance tasks
- Use branch naming convention: `feature/`, `bugfix/`, `hotfix/`, `docs/`, `refactor/`, `test/`, `chore/`, `ci/`
- Ensure your code passes all CI checks (YAML lint, Ansible lint, Molecule tests)
- Submit a pull request describing your changes
- For major changes, please open an issue first to discuss what you would like to change

## 📝 License

This project is licensed under the Apache-2.0 License - see the LICENSE file for details.

## 👥 Author Information

This role was created by [Grzegorz Franus](https://github.com/grzegorzfranus).
