# Ansible
[gcd]:https://github.com/GoogleCloudPlatform/compute-video-demo-ansible/

This week's focus is going to be on Ansible. We are going to fire up the servers we created last
week on Google Cloud with Ansible to familiarize with automation and orchestrations techniques.

References: 

 * [Ansible for
  Devops(book)](https://www.ansible.com/hubfs/-2016-ebooks/ansible-for-devops-first-four-chapters.pdf
  )
  
 * [Official GCP documentation
   (github)](https://github.com/GoogleCloudPlatform/compute-video-demo-ansible/ )
 * [Programmatic pondering
   (blog)](https://programmaticponderings.com/2019/01/30/getting-started-with-red-hat-ansible-for-google-cloud-platform/
   ) 
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
 - **playbooks**:
   The true power of Ansible lies in the playbooks---scripts to automate the creation of server.
   These are `.yml` files that are convenient because of the ease of understaning for both computers
   (which interpret them and run the equivalent command) and from user (experience or
   less-experienced) being them quite verbose.
   Playbooks can be run with the following:
   
       $ ansible-playbook playbook-name.yml
   Again, if no change are made to the config file and rerun the playbook, nothing is affected.
   - **Ansible Towers**:
     I might be mistaken, but these are basically images for ansible, useful for the users that do
     not want/need to gets their hands dirty.
     
A very useful feature for ansible is the _dry-run_ or _check mode_ which validate the commands or
playbooks before making changes on the target systems.
<!-- blank line -->
## Ansible and gc

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
    *  **Module parameter specification**:
         In the the yml file we can specify the following variable as a subfield, eg.
```yaml 
- name: Create IP address
    hosts: localhost
           
    vars:
    #type of auth method used (machineaccount, serviceaccount, application)
        auth_kind: serviceaccount
        #email associated with the project
        service_account_email: myemail@domain.com
        #Path to JSON credentials
        service_account_file: ./my_account.json
        #id of the project:
        project: my-project
        #specific scope to use
        scope: -
```
 - **providing credentials as Environment Variables**:
   Just set the variable before running ansible:
```bash  
   GCP_AUTH_KIND
   GCP_SERVICE_ACCOUNT_EMAIL
   GCP_SERVICE_ACCOUNT_FILE
   GCP_SCOPES
```
Once the instances are created they could be accessed via ssh, upon enabling it with:

    $ gcloud  compute config-ssh

## Getting started with our project:

**READ** Ansible's documentation [best
practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#directory-layout
), [YAML syntax
introduction](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html#yaml-syntax
). and [intro to
playbooks](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html#about-playbooks
) and this [short tutorial on writing playbooks by DigitalOcean](https://www.digitalocean.com/community/tutorials/configuration-management-101-writing-ansible-playbooks )

## Launch the first instances

We are going to split the process in different yaml files, these can then be called at once by
regrouping them inside another yaml file. Let's touch this first:

    $ echo "---" > site.yml
    $ echo "# workflow to create instances" >> site.yml
    $ echo "- include: crete-instances.yml" >> site.yml
This `all.yml` file is going to be very simple and will be a mere collection of the ymls we are
going to parse to `ansible-playbook`. They could be launched singularly, but in this way, it is
going to be easy to reorchestrate the workflow even with no knowledge/memory of what has been done. Breaking down the meaning of these tree lines:
1. It just initialize the yml file `all.yml`, the pattern `---` is optional in yml files and just
   states the beginning of the file. We could add a `...` to state the ending of it, but it is more
   convenient to skip this process and echoing the next lines at the end.
2. A comment to express what we are doing
3. This tell ansible (and the user) that we are going to start with the `create-instances` file

*** 

To create our first yml file we are going to fetch (or copy-paste) the `gce-instances.yml` file from
the :fa-github: [Google Cloud Github documentation][gcd].
```yml
- name: Create Compute Engine instances
  hosts: local
  gather_facts: False
  vars_files:
    - gce_vars/auth
    - gce_vars/machines
    - gce_vars/zone
  tasks:
    - name: Create an IP address for first instance
      gcp_compute_address:
        name: "{{ name_zonea }}-ip"
        region: "{{ region }}"
        project: "{{ project }}"
        service_account_file: "{{ credentials_file }}"
        auth_kind: "{{ auth_kind }}"
      register: gcea_ip
    - name: Bring up the first instance in the first zone.
      gcp_compute_instance:
        name: "{{ name_zonea }}"
        machine_type: "{{ machine_type }}"
        disks:
          - auto_delete: true
            boot: true
            initialize_params:
              source_image: "{{ image }}"
        network_interfaces:
          - access_configs:
              - name: External NAT
                nat_ip: "{{ gcea_ip }}"
                type: ONE_TO_ONE_NAT
        tags:
          items:
            - http-server
            - https-server
        zone: "{{ zone }}"
        project: "{{ project }}"
        service_account_file: "{{ credentials_file }}"
        auth_kind: "{{ auth_kind }}"
      register: gcea
    - name: Create an IP address for second instance
      gcp_compute_address:
        name: "{{ name_zoneb }}-ip"
        region: "{{ region }}"
        project: "{{ project }}"
        service_account_file: "{{ credentials_file }}"
        auth_kind: "{{ auth_kind }}"
      register: gceb_ip
    - name: Bring up the instance in the second zone.
      gcp_compute_instance:
        name: "{{ name_zoneb }}"
        machine_type: "{{ machine_type }}"
        disks:
          - auto_delete: true
            boot: true
            initialize_params:
              source_image: "{{ image }}"
        network_interfaces:
          - access_configs:
              - name: External NAT
                nat_ip: "{{ gceb_ip }}"
                type: ONE_TO_ONE_NAT
        tags:
          items:
            - http-server
            - https-server
        zone: "{{ zone }}"
        project: "{{ project }}"
        service_account_file: "{{ credentials_file }}"
        auth_kind: "{{ auth_kind }}"
      register: gceb
  post_tasks:
    - name: Wait for SSH for instances in first zone
      wait_for: delay=1 host={{ gcea_ip.address }} port=22 state=started timeout=30
    - name: Save host data for first zone
      add_host: hostname={{ gcea_ip.address }} groupname=gce_instances_ips
    - name: Wait for SSH for instances in second zone
      wait_for: delay=1 host={{ gceb_ip.address }} port=22 state=started timeout=30
    - name: Save host data for second zone
      add_host: hostname={{ gceb_ip.address }} groupname=gce_instances_ips
```

Let's remove the `var_files` section and add a `vars:` section instead, in this section we can
define whatever variable we want to call later in this file. 
To understand what the rest of the file is doing, one approach (maybe the most effective) is to
perform a research 
on the [ansible documentation about gcp
](https://docs.ansible.com/ansible/latest/search.html?q=gcp&check_keywords=yes&area=default# )
and look up for the specific yaml keywords. The end result should be a file
looking like this
```yml
- name: Create Compute Engine instances #naming the process
  hosts: localhost
  gather_facts: False
  # vars_files:
  #   - gce_vars/auth
  #   - gce_vars/machines
  #   - gce_vars/zone

  vars:
    service_account_email: n5602-ansible@dev2ops.iam.gserviceaccount.com
    creds_file: ./<key_file>.json    
    project: dev2ops
    machine_type: f1-micro
    zone: europe-west1-b
    region: europe-west1
    backend: n5602-ansible-backend
    database: n5602-ansible-database
    auth_kind: serviceaccount
    image: projects/ubuntu-os-cloud/global/images/family/ubuntu-1804-lts
    
  tasks:
    - name: Create an IP address for first instance
      gcp_compute_address:
        name: "{{backend}}"
        region: "{{ region }}"
        project: "{{ project }}"
        service_account_file: "{{ creds_file }}"
        auth_kind: "{{ auth_kind }}"
      register: backend_ip #capture the result of this task to the variable
    - name: create instance for the backend.
      gcp_compute_instance:
        name: "{{ backend }}"
        machine_type: "{{ machine_type }}"
        disks:
          - auto_delete: true
            boot: true
            initialize_params:
              source_image: "{{ image }}"
        network_interfaces:
          - access_configs:
              - name: External NAT
                nat_ip: "{{ backend_ip }}"
                type: ONE_TO_ONE_NAT
        tags:
          items:
            - http-server
            - https-server
        zone: "{{ zone }}"
        project: "{{ project }}"
        service_account_file: "{{ creds_file }}"
        auth_kind: "{{ auth_kind }}"
      register: backend
      
    - name: Create an IP address for second instance
      gcp_compute_address:
        name: "{{ database }}"
        region: "{{ region }}"
        project: "{{ project }}"
        service_account_file: "{{ creds_file }}"
        auth_kind: "{{ auth_kind }}"
      register: database_ip
      
    - name: create the instance for database.
      gcp_compute_instance:
        name: "{{ database }}"
        machine_type: "{{ machine_type }}"
        disks:
          - auto_delete: true
            boot: true
            initialize_params:
              source_image: "{{ image }}"
        network_interfaces:
          - access_configs:
              - name: External NAT
                nat_ip: "{{ database_ip }}"
                type: ONE_TO_ONE_NAT
        tags:
          items:
            - http-server
            - https-server
        zone: "{{ zone }}"
        project: "{{ project }}"
        service_account_file: "{{ creds_file }}"
        auth_kind: "{{ auth_kind }}"
      register: database
      
  post_tasks:
    - name: Wait for SSH for instances in first zone
      wait_for: delay=1 host={{ backend_ip.address }} port=22 state=started timeout=30
    - name: Save host data for first zone
      add_host: hostname={{ backend_ip.address }} groupname=gce_instances_ips
    - name: Wait for SSH for instances in second zone
      wait_for: delay=1 host={{ database_ip.address }} port=22 state=started timeout=30
    - name: Save host data for second zone
      add_host: hostname={{ database_ip.address }} groupname=gce_instances_ips
```

## Dynamic inventory

To define an inventory that track the machine we are going to use this
[tutorial](http://matthieure.me/2018/12/31/ansible_inventory_plugin.html )
First of all, we must tell ansible to use the library plugin for GCP, this is
done by creating a `ansible.cfg` file with the following content:
```
[inventory]
enable_plugins = gcp_compute
```

Then create this file:
```yml
# inventory.gcp.yml
plugin: gcp_compute
projects:
  - dev2ops
account: serviceaccount
service_account_file: ./<key-file>.json
```
and add this task in the `create-instances.yml` file after having created all
the instances:
```yml
tasks:
  ...
  -name: Refresh the inventory
  -meta: refresh_inventory
```
## setting the  YAML file
By now, we have fired up our instances and now we want to go on and install
docker on them and then load the images for managing a webapp front/backend 
and a database. 
### Roles
The best way to automize task that are identical on different machines is
by the mean of
[Roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html
).
Roles are ways to automatically load certain vars_files, task or handlers;
but to work properly we need to set a prefix directory structure.
Therefore, let us create the folder roles:
  
  $ mkdir roles
Inside this folder we will create the name of the various role we are going
to use in the principal YAML file. For example, we want to install docker
so we shall create the following folders:
  
  $ mkdir -p roles/docker/tasks
Inside te tasks folder we create a `main.yml` file, which is going to
automate the installation of docker and its dependencies.

### Docker role

Inside the `roles/docker/tasks` we are going to create the following `main.yml`
```yml
- name: Install docker dependencies
  apt:
    name: ['apt-transport-https', 'ca-certificates', 'curl', 'gnupg-agent', 'software-properties-common']
    state: present

- name: Add docker's key to the repo 
  apt:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present

- name: Add Docker repo
  apt_repository:
    repo: deb https://download.docker.com/linux/ubuntu bionic stable
    state: present

- name: Update apt and install docker
  apt:
    update_cache: yes
    name: ['docker-ce', 'docker-ce-cli', 'containerd.io']
    state: latest

- name: Start docker
  service:
    name: docker
    state: started
```
This files mimicks the commands we used to install docker on the VM instances.

> In order to debug the following:
>    
>    **fatal: [35.195.192.79]: UNREACHABLE! => {"changed": false, "msg": "Failed to
>    connect to the host via ssh: Host key verification failed.", "unreachable":
>    true}**
> We are going to add, following this
>    [thread](https://stackoverflow.com/questions/46929624/failed-to-connect-to-the-host-via-ssh-host-key-verification-failed-r-n/46933307
>    ), we are going to add the next two lines to the `ansible.cnf` file:
>
>    [defaults]
>    host_key_checking = False

