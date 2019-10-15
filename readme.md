# Ansible


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
```yaml {.line-numbers}
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
```bash {.line-numbers} 
   GCP_AUTH_KIND
   GCP_SERVICE_ACCOUNT_EMAIL
   GCP_SERVICE_ACCOUNT_FILE
   GCP_SCOPES
```
Once the instances are created they could be accessed via ssh, upon enabling it with:

    $ gcloud  compute config-ssh


## Launch the first instances

We are going to split the process in different yml files, these can then be called at once by
regrouping them inside another yml file. Let's touch this first:

    $ echo "---" > all.yml
    $ echo "# workflow to create instances" >> all.yml
    $ echo "- include: crete-instances.yml"
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
```yml{.line-numbers}
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


[gcd]:https://github.com/GoogleCloudPlatform/compute-video-demo-ansible/
