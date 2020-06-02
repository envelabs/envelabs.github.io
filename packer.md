# Packer

Packer is a tool for creating machine and container images for multiple platforms from a single source configuration. Out of the box Packer comes with support to build images for Amazon EC2, DigitalOcean, Docker, Google Compute Engine, QEMU, VirtualBox, VMware, Microsoft Azure, and more.

## Resources

[https://www.packer.io/](https://www.packer.io/)

[Packer Documentation - Packer by HashiCorp](https://www.packer.io/docs/)

[Building Docker image with Packer and provisioning with Ansible · GitHub](https://gist.github.com/maxivak/2d014f591fc8b7c39d484ac8d17f2a55)

[Advanced Provisioning With Packer For Docker And Vagrant - Matthew McKeen](http://mmckeen.net/blog/2013/12/27/advanced-docker-provisioning-with-packer/)

##### Packer install

1. Download from [Downloads - Packer by HashiCorp](https://www.packer.io/downloads.html)
2. unzip it and move it to your */usr/local/bin* or *\~/bin* directory (be sure that your *bin* is in your $PATH)
3. check if *packer* is properly installed

```
packer

usage: packer [--version] [--help] <command> [<args>]

Available commands are:
    build        build image(s) from template
    fix          fixes templates from old versions of packer
    inspect      see components of a template
    validate     check that a template is valid
```

#### Packer config

```
#todo
```

#### Build

```
PACKER_LOG=1 packer build build.json
```

#### Building Docker image with Packer and provisioning with Ansible

**Overview**

Packer

* Packer is used to build image from a base image, perform provisions and store (commit) the final image.
* We use provisioners and Packer templates to do the actual work to create the final image.
* We use Ansible for provisioning.

Ansible

* Ansible runs playbooks on localhost (inside Docker container).
* Ansible must be installed on the image you're provisioning.
* Packer uploads the contents of the Ansible playbook to the Docker container instance, and runs it locally.

## Build workflow

### Build the build.json configuration file

Variable definition

```
{
  "variables": {
    "tmp_dir": "/tmp",
    "playbook_dir": "/opt/devops",
    "start_services": "bin/start_services.sh",
    "stop_services": "bin/stop_services.sh"
  },
```

Builder definition

```
  "builders":[
    {
      "type": "docker",
      "image": "ubuntu:16.04",
      "pull": "false",
      "commit": true
    }
  ],
```

Provisioners definition

```
  "provisioners":[
    {
      "type": "shell",
      "inline": [
        "apt-get -y update",
        "apt-get install -y software-properties-common",
        "apt-add-repository ppa:ansible/ansible",
        "apt-get -y update",
	"apt-get install -y sudo wget curl iputils-ping net-tools less vim --allow-unauthenticated",
        "apt-get install -y python --allow-unauthenticated",
        "apt-get install -y ansible --allow-unauthenticated"
      ]
    },
    {
      "type": "shell",
      "inline": [
        "mkdir -p {{user `tmp_dir`}}"
      ]
    }
    ,
    {
      "type": "file",
      "source": "{{user `start_services`}}",
      "destination": "{{user `tmp_dir`}}/start_services.sh"
    }
    ,
    {
      "type": "file",
      "source": "{{user `stop_services`}}",
      "destination": "{{user `tmp_dir`}}/stop_services.sh"
    }
    ,
    {
      "type": "shell",
      "inline": [
        "chmod u+x /{{user `tmp_dir`}}/*_services.sh",
        "./{{user `tmp_dir`}}/start_services.sh"
      ]
    }
    ,
    {
      "type": "ansible-local",
      "staging_directory": "{{user `tmp_dir`}}/ansible",
      "playbook_dir": "{{user `playbook_dir`}}",
      "playbook_file": "play.packer.yml"
    }
  ],

  "post-processors": [
    [
      {
        "type": "docker-tag",
        "repository": "enve/tradehelm.dsc",
        "tag": "0.4"
      }
    ]
  ]
}
```

#### Build Image

```
sudo packer build build.json
```

#### Run Docker container

```
sudo docker run --name <container_name> -p 8080:8080 -it -d <image_name> bash
```

#### Building a Docker image using a local repo

Building a template with the docker builder, you can have an error when I try to specify an image which is local to the docker instance.

In order to avoid the issue, you need to add an specific value to the `pull:false `parameter

```
 "builders": [
        {
            "commit": true,
            "pull": false,
            "image": "increment-image",
            "type": "docker"
        }
    ]
```
