# How to boostrap CoreOS on bare metal

## System Pre-requisites

### Bare Metal Server

* 16GB+ RAM
* 4+ cores
* 100GB+ SSD
* Internet access

## Download CoreOS iso

Download the Bare Metal Live DVD ISO

https://www.fedoraproject.org/coreos/download?stream=stable#arches



https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/42.20250526.3.0/x86_64/fedora-coreos-42.20250526.3.0-live-iso.x86_64.iso



## Generate RSA Key Pair


```
ssh-keygen -t rsa -b 2048 -f ~/.ssh/core_id_rsa
```


## Create Butane yaml file

Create coreos directory where the config.bu file will be stored.

Create config.bu file in the coreos directory. Content is yaml

```
variant: fcos
version: 1.5.0
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - ssh-rsa <RSA-KEY-VALUE>
                
storage:
  files:
    - path: /etc/hostname
      mode: 0644
      contents:
        inline: coreos-node1

systemd:
  units:
    - name: docker.service
      enabled: true

    - name: automatic-updates.timer
      enabled: true
```

## Convert Butane butane.bu file to Ignition coreos.ign

This step converts the butane.bu file to Ignition format

```
docker run -i --rm quay.io/coreos/butane:release --strict < config.bu > coreos.ign
```

## Webhost Ignition coreos.ign file

Get machine ip address

```
ip addr
```

From within the coreos directory run a http server:

```
python3 -m http.server
```


## On CoreOS Live DVD Machine

Identify the block device where CoreOS will be installed

```
lsblk
```

Install CoreOS

```
sudo coreos-installer install /dev/sda1 --ignition-url http://192.168.0.???:8000/coreos.ign --insecure-ignition
```

