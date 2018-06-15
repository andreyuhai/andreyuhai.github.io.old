---
layout: post
title: "Ansible force remote user with Vagrant 1.8"
excerpt: My Ansible playbook stopped working when using Vagrant 1.8 or above. The problem seems to be with changes made in Vagrant.
modified: 2016-06-08 18:09:12 +0300
categories: articles
tags: [ansible, vagrant, provisioning]
image:
  feature: 2016-05-28-ansible-skipping-localhost-when-using-vagrant/cover.jpg
  credit: https://www.vagrantup.com
  creditlink: https://www.vagrantup.com/assets/images/vagrant_header_background-482a12a7.png
comments: true
share: true
published: true
aging: true
---

I have an [Ansible playbook](https://docs.ansible.com/ansible/playbooks.html "Ansible Playbooks documentation") that I use to provision my virtual machine. Using the Ansible provisioner for [Vagrant](https://www.vagrantup.com/ "Vagrant homepage") worked fine for me. But when I upgraded to Vagrant 1.8, Ansible failed because it could not SSH into the VM. On the other hand, running the playbook with `anisble-playbook` command seemed to work fine.

I was able to solve the problem by adding the `force_remote_user` setting to my `Vagrantfile`.

{% highlight ruby %}
a.vm.provision :ansible do |ansible|
  ansible.playbook = 'site.yml'
  ansible.inventory_path = 'inventory/dev/hosts'
  ansible.force_remote_user = false
end
{% endhighlight %}

## How did that solve the problem?

First of all, in my inventory file, I had specified which user Ansible should use when connecting via SSH. That's all good, but Vagrant seemed to ignore that. Instead of the user I explicitly configured, the `vagrant` user was used. After going through [Vagrant docs](https://www.vagrantup.com/docs/provisioning/ansible.html "Ansible provisioner documentation") I found the following:

>`force_remote_user` (boolean) - require Vagrant to set the `ansible_ssh_user` setting in the generated inventory, or as an extra variable when a static inventory is used. All the Ansible `remote_user` parameters will then be overridden by the value of `config.ssh.username` of the Vagrant SSH Settings.
If this option is set to false Vagrant will set the Vagrant SSH username as a default Ansible remote user, but `remote_user` parameters of your Ansible plays or tasks will still be taken into account and thus override the Vagrant configuration.

The default value for this option is true and it was introduced in Vagrant 1.8. Versions prior to 1.8 acted as if this setting was set to false. So the obvious solution for me was to explicitly set this option to false.
