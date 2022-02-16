# Role Assisted installer Podman

Ansible role is used for deploying Assisted installer locally.


### **Requirements** 
---
-  Ansible >= 2.9


### **How To Use**
---
setup galaxy requirement file

```yaml
# requirement.yaml
- src: https://github.com/eifrach/ansible-role-assisted-installer-pod
  version: main
  name: assisted-installer-pod
```
install the role locally
```shell
ansible-galaxy install -r requirement.yaml
```

next create a playbook and vars to your needs:
```yaml
# playbook.yaml
---
- hosts: localhost
  roles:
    - assisted-installer-pod
  vars:
    WORKDIR: /opt/assisted-podman
    CLEAN: true
    HW_VALIDATOR_REQUIREMENTS:
        - version: default
          sno:
            disk_size_gb: 13
```
finally run the playbook
```shell
ansible-playbook -i localhost, playbook.yaml
```

you can test your vars using `--tags tests`

```shell
ansible-playbook -i localhost, playbook.yaml --tags tests
```

Clean up environment
```shell
ansible-playbook -i localhost, playbook.yaml --tags cleanup
```


### **Vars and Defaults**
---
global configuations
```yaml
# name of the service and pod
SERVICE_NAME: assisted-installer

# location for configfiles and Database volume
WORKDIR: /home/assisted-podman

# create persistent volume for database
PRESERVE_DB: true
# if you need to clean up old database on the 
# next deployment
CLEAN: false

# IPv6 support default to true
IPV6: true

# set default NTP server
NTP_SERVER: ""

# enabled Dnsmasq single node
ENABLE_SINGLE_NODE_DNSMASQ: true


DUMMY_IGNITION: false

# default container registries 
PUBLIC_CONTAINER_REGISTRIES: "quay.io,registry-proxy.engineering.redhat.com"

# service mode http / https
ASSISTED_SERVICE_SCHEME: "http"
```

change image for assisted installer
```yaml
# Service container
SERVICE_IMAGE: quay.io/edge-infrastructure/assisted-service:latest

# UI image
ASSISTED_UI_IMAGE: quay.io/edge-infrastructure/assisted-installer-ui:latest

#image Service
IMAGE_SERVICE_IMAGE: quay.io/edge-infrastructure/assisted-image-service:latest
# postgres sql image 
PSQL_IMAGE_IMAGE: quay.io/centos7/postgresql-12-centos7:latest
```


basic configuration for database connection:
```yaml
# Set database config
POSTGRESQL_DATABASE: installer
POSTGRESQL_PASSWORD: password
POSTGRESQL_USER: admin

# set service connection to database
DB_HOST: 127.0.0.1
DB_PORT: 5432
```

hardware validation changes can be done be adding a list.

example to change the CPU limit on master and workers nodes
```yaml
---
- hosts: all
  roles:
    - assisted-installer-pod
  vars:
    HW_VALIDATOR_REQUIREMENTS:
        - version: default
          master:
            cpu_cores: 8 
          worker:
            cpu_cores: 4
```

the following is the default values for hardware validation
```yaml
# default values 
HW_VALIDATOR_REQUIREMENTS:
  - version: default
    master:
      cpu_cores: 4
      ram_mib: 16384
      disk_size_gb: 120
      installation_disk_speed_threshold_ms: 10
      network_latency_threshold_ms: 100
      packet_loss_percentage: 0
    worker:  
      cpu_cores: 2
      ram_mib: 8192
      disk_size_gb: 120
      installation_disk_speed_threshold_ms: 10
      network_latency_threshold_ms: 1000
      packet_loss_percentage: 10
    sno: 
      cpu_cores: 4
      ram_mib: 16384
      disk_size_gb: 120
      installation_disk_speed_threshold_ms: 10
```

Adding release images:
> to add release images please set a list with the following values:

```yaml
RELEASE_IMAGES:
    - openshift_version: "4.10"
      cpu_architecture: "x86_64"
      url: "quay.io/openshift-release-dev/ocp-release:4.10.0-fc.3-x86_64"
      version: "4.10.0-fc-3"
```

Adding os images: 
```yaml
OS_IMAGES:
    - openshift_version: "4.10"
      cpu_architecture: "x86_64"
      url: "https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/pre-release/4.10.0-fc.3/rhcos-4.10.0-fc.3-x86_64-live.x86_64.iso"
      rootfs_url: "https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/pre-release/4.10.0-fc.3/rhcos-4.10.0-fc.3-x86_64-live-rootfs.x86_64.img"
      version: "410.84.202201261816-0"
```

specifying container images:
```yaml
CONTROLLER_IMAGE: registry.redhat.io/rhai-tech-preview/assisted-installer-reporter-rhel8:v1.0.0-151
AGENT_DOCKER_IMAGE: registry.redhat.io/rhai-tech-preview/assisted-installer-agent-rhel8:v1.0.0-82
INSTALLER_IMAGE: registry.redhat.io/rhai-tech-preview/assisted-installer-rhel8:v1.0.0-116
```

