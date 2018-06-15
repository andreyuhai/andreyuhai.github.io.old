---
layout: post
title: "Using Ansible to manage SugarCRM 7 environments and deployments"
excerpt: I've decided to use Ansible to manage SugarCRM environments and deployments. Here's an overview of Ansible playbooks I've created.
modified: 2016-04-12 21:44:00 +0300
categories: articles
tags: [ansible, sugarcrm, configuration management, deployment]
image:
  feature: 2016-04-12-using-ansible-with-sugarcrm-7/cover.png
  credit:
  creditlink:
comments: true
share: true
published: true
---

SugarCRM is a web based customer relationship management system written in PHP. [Due to how it is designed there are some difficulties getting it into source control.]({{ site.url }}/articles/version-control-and-deployment-practices-for-sugarcrm/) Additionally there are a few things to keep in mind when managing multiple environments and doing deployments. I've decided to use Ansible to manage SugarCRM environments and deployments.

## Getting started

Firstly you should have a look at [sugarcrm-ansible Github repository](https://github.com/indrekots/sugarcrm-ansible "sugarcrm-ansible Github repository") which contains Ansible playbooks for creating a new SugarCRM environment. The rest of this post will go over how to use these playbooks to set up a new environment and make deployments.

### Install Vagrant
Vagrant will be used to create a virtual machine for local development.

* Install [Vagrant](https://www.vagrantup.com/downloads.html "Vagrant download page")
* Install the latest version of [VirtualBox](https://www.virtualbox.org/ "VirtualBox homepage") (or any other [supported provider](https://www.vagrantup.com/docs/providers/ "Vagrant providers"))

### Set up folder structure

Vagrant will mount SugarCRM from your local filesystem. The following diagram shows where Vagrant expects you to have placed Sugar's source code relative to the `Vagrantfile`.

{% highlight java %}
.
├── sugarcrm-ansible/
│   └── Vagrantfile
└── sugarcrm/
{% endhighlight %}

You can change the location of your SugarCRM installation by changing the `local_site_path` variable in `inventory/development/group_vars/all/common.yml`.

### Required Vagrant plugins
* [vagrant-hostsupdater](https://github.com/cogitatio/vagrant-hostsupdater)

If required plugin is missing, vagrant commands will fail.

### Configure variables
To configure your install, make changes in `inventory/development/group_vars/all/common.yml` and `group_vars/all/common.yml`.

### vagrant up

After setting up your folder structure, you should be able to run `vagrant up`. A new VM is created and Ansible is used for provisioning.
You will be prompted for your sudo password since *vagrant-hostsupdater* will make changes in your `/etc/hosts` file and map a domain name to point to the newly created VM.

If Ansible finished successfully, then point your browser to the domain name you specified in your [inventory](https://github.com/indrekots/sugarcrm-ansible/blob/master/inventory/development/group_vars/all/common.yml "Inventory example file") and you should see a fresh install of SugarCRM.

## Deployment

### Create a deploy key
For deployments to work you should host your Sugar source code in a Git repository. Then [generate an SSH key pair](https://help.github.com/articles/generating-an-ssh-key/ "Generating an ssh key") which allows you to access the repository.
Look at `inventory/development/group_vars/deploy_key.yml` for example and replace it with your private key.

Configuration management should be kept in source control. You don't want to expose any secrets. Therefore you should encrypt your private key. Ansible provides a feature called ["Vault"](https://docs.ansible.com/ansible/playbooks_vault.html "Ansible Vault documentation") that allows keeping sensitive data such as passwords or keys in encrypted files.

{% highlight bash %}
$ ansible-vault encrypt inventory/development/group_vars/deploy_key.yml
{% endhighlight %}

When running a playbook, pass the `--ask-vault-pass` flag for Ansible to prompt you for your vault password.

### Run Ansible deploy playbook
Running `ansible-playbook -i inventory/development/hosts deploy.yml -k -K --ask-vault-pass` will start a deployment to your local environment. Meaning that the latest commit from your repository will be checked out and applied to your local environment. Additionally the [repair script](https://gist.github.com/chicks/6084088#file-repair-php "SugarCRM repair script") will be called.

### Create new inventory entry
Creating a new inventory entry for a production environment is easy. Create a new folder `inventory/prod` and copy the contents of `inventory/development` into the newly created directory.
Replace all variables with values for the new environment. Make sure the `hosts` file contains correct addresses.

## Contribute

Please feel free to contribute to the [repository](https://github.com/indrekots/sugarcrm-ansible "sugarcrm-ansible Github repository") to make the playbooks more versatile.
