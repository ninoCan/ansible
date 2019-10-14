# Ansible


This week's focus is going to be on Ansible. We are going to fire up the servers we created last
week on Google Cloud with Ansible to familiarize with automation and orchestrations techniques.

References: 
* [Ansible for
  Devops(book)](https://www.ansible.com/hubfs/-2016-ebooks/ansible-for-devops-first-four-chapters.pdf)
* [Official GCP documentation :github:](https://github.com/GoogleCloudPlatform/compute-video-demo-ansible)
## What is Ansible?

Ansible is a automation and orchestration tool and engine, created to automate server creation in a
simple way which makes it easy to reproduce the same server settings and scale it if/when needed
(for instance when the company grows). Ansible is "battery-included" meaning that it does not need
additional software to run. Ansible fundamental tools are:
- inventory
 - a list of the servers it is going to orchestate. This could be static or dynamics(for example
   AWS, Azure, GCP servers and so on)
 - ranges
 - customizations
-playbooks, the main file that we are going to pass. It contains: 
 - tasks (sequentially run commands) 
 - modules (automate nearly every part of the orchestrated environment)
 - handlers: triggered by tasks and running at the end of the play.
- API's
- plugins
What makes it particularly ideal is the fact that it does not need to be set up on the orchestrated
machines and can access them via ssh. In addition, Ansible playbooks function idempotentially,
meaning that, once we lauched the servers first time, settings will not be modified on eventual
later calls (that contains no new instructions).

## Using Ansible
Ansible can be used in three ways:

- **command line**:
  Commands and modules can be called directly from command line with the following structure:
  `ansible <host-patten> [options]`
In the host pattern we need to specify which machine are we using Ansible on. One of the most common
  ways to do it is by parsing the inventory.
Inventories can be directly saved in the folder `/etc/ansible/` or on the local folder and called
  with the `-i` flag. Eg.
  
      $ansible -i <inventory-name> [server-name] ...
  The inventory file can be given an arbitrary name and its most basic form is:
  
      [name-to-refer-the-server]
      ip.ad.dr.ess
      
      [name-to-refer-other-server]
      www.your-domain.name
  The bracketed name is the one you could use in the server-name field.
  After we specified an host, we can either run a command by calling it after the `-a` flag, e.g.:
  
      $ ansible web -a /bin/date
  or we can call one of the ansible modules with the `-m` flag: eg.
  
      $ ansible -i inventory -m ping <server-name>
## Ansible and GC

In order to create gce instances with Ansible on GCP we need to create [credentials](https://docs.ansible.com/ansible/latest/scenario_guides/guide_gce.html#credentials) for ansible. 
This can be easily done throught the GCP gui:
- open the Navigation Menu
- click IAM & admin
  - select Service accounts
- click the Create service account on top of the screen.
- fill in the service account name: e.g. n5602-ansible
- fill in a description on the service account: e.g. manage servers with ansible
- click create 
- click continue at the bottom
- click create key
- save the key to your project folder
Having done so, there are 2 two possible ways to provide the credentials to Ansible:
    * 
Once the instances are created they could be accessed via ssh, upon enabling it with:

    $ gcloud  compute config-ssh
