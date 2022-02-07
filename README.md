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

### **Vars and Defaults**
---
global configuations
```yaml
# location for configfiles and Database volume
WORKDIR: /home/assisted-podman

# create persistent volume for database
PRESERVE_DB: true
# if you need to clean up old database on the 
# next deployment
CLEAN: false
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
DB_USER: admin
DB_PASS: password
DB_NAME: installer
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
      disk_size_gb: 20
      installation_disk_speed_threshold_ms: 10
      network_latency_threshold_ms: 100
      packet_loss_percentage: 0
    worker:  
      cpu_cores: 2
      ram_mib: 8192
      disk_size_gb: 20
      installation_disk_speed_threshold_ms: 10
      network_latency_threshold_ms: 1000
      packet_loss_percentage: 10
    sno:
      cpu_cores: 8
      ram_mib: 32768
      disk_size_gb: 20
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

Adding is images: 
```yaml
OS_IMAGES:
    - openshift_version: "4.10"
      cpu_architecture: "x86_64"
      url: "https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/pre-release/4.10.0-fc.3/rhcos-4.10.0-fc.3-x86_64-live.x86_64.iso"
      rootfs_url: "https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/pre-release/4.10.0-fc.3/rhcos-4.10.0-fc.3-x86_64-live-rootfs.x86_64.img"
      version: "410.84.202201261816-0"
```



