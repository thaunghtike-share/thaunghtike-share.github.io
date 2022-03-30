---
layout:     post
title:   "Introduction To Terraoform Provisioners"
date:       2022-02-14 15:13:18 +0200
image: 36.png
tags:
    - Terraform
    - IAC
categories: Terraform
---

<h1> Introduction To Terraforn Provisioners </h1>

As you continue learning about Terraform, you will start hearing about provisioners. Terraform provisioners can be created on any resource and provide a way to execute actions on local or remote machines. Provisioners can be used to model specific actions on the local machine or on a remote machine in order to prepare servers or other infrastructure objects for service.

<h2> Types of provisioners </h2>

There are two categories for the terraform provisioners. Those categories are 

- generic provisioner ( file, local-exec and remote-exec )
- vendor provisioner - configuration provisioner ( chef, puppet, ansible etc )

Let’s get started discussing each of these.

<h2> File Provisioners </h2>

The provisioner was used to collect or create content that needs uploading to a remote host. This use could be gathering information from other Terraform resources and generating a config file that gets uploaded to a server. It can also move files to different locations as part of the Terraform execution. It comes down to your needs as to how you would use it. Here is an example.

```yaml
resource "null_resource" remoteExecProvisionerWFolder {

  provisioner "file" {
    source      = "test.txt"
    destination = "/tmp/test.txt"
  }

  connection {
    host     = "${azurerm_virtual_machine.vm.ip_address}"
    type     = "ssh"
    user     = "${var.admin_username}"
    password = "${var.admin_password}"
    agent    = "false"
  }
}
```
<h2> Local Exec </h2>

The local exec provisioner executes code locally on the machine that is running the Terraform. This provisioner is useful when you need steps to occur with other tools you have installed. Here is an example.

```bash
resource "null_resource" "azure-cli" {
  
  provisioner "local-exec" {
    # Call Azure CLI Script here
    command = "ssl-script.sh"

    # We are going to pass in terraform derived values to the script
    environment {
      webappname = "${azurerm_app_service.demo.name}"
      resourceGroup = ${azurerm_resource_group.demo.name}
    }
  }

  depends_on = ["azurerm_app_service_custom_hostname_binding.demo"]
}
```
<h2> Remote Exec </h2>

The remote exec provisioner executes tools and scripts on a remote target. This ability comes in handy if you need to run a command or script on a VM that doesn’t allow some other way. Let’s look at how to do that for an AWS instance.

```bash
resource "aws_instance" "web" {
  # ...

  # Establishes connection to be used by all
  # generic remote provisioners (i.e. file/remote-exec)
  connection {
    type     = "ssh"
    user     = "root"
    password = var.root_password
    host     = self.public_ip
  }

  provisioner "remote-exec" {
    inline = [
      "puppet apply",
      "consul join ${aws_instance.web.private_ip}",
    ]
  }
}
```
<h2> Destroy-Time Provisioners </h2>

By default, provisioners run when the resource they are defined within is created. Creation-time provisioners are only run during creation, not during updating or any other lifecycle.

If when = destroy is specified, the provisioner will run when the resource it is defined within is destroyed.

```bash
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Destroy-time provisioner'"
  }
}
```

Destroy provisioners are run before the resource is destroyed. If they fail, Terraform will error and rerun the provisioners again on the next terraform apply

<h2> Multiple Provisioners </h2>

Multiple provisioners can be specified within a resource. Multiple provisioners are executed in the order they're defined in the configuration file.

You may also mix and match creation and destruction provisioners. Only the provisioners that are valid for a given operation will be run. Those valid provisioners will be run in the order they're defined in the configuration file.

Example of multiple provisioners:

```bash
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command = "echo first"
  }

  provisioner "local-exec" {
    command = "echo second"
  }
}
```
<h2> Failure Behavior </h2>

By default, provisioners that fail will also cause the Terraform apply itself to fail. The on_failure setting can be used to change this. The allowed values are:

    continue - Ignore the error and continue with creation or destruction.

    fail - Raise an error and stop applying (the default behavior). If this is a creation provisioner, taint the resource.

Example:

```bash
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command    = "echo The server's IP address is ${self.private_ip}"
    on_failure = continue
  }
}
```

<h2> References </h2>

- [https://www.phillipsj.net/posts/introduction-to-terraform-provisioners](https://www.phillipsj.net/posts/introduction-to-terraform-provisioners)
- [https://www.terraform.io/language/resources/provisioners/syntax](https://www.terraform.io/language/resources/provisioners/syntax)
