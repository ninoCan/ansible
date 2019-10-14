# Ansible


This week's focus is going to be on Ansible. We are going to fire up the servers we created last
week on Google Cloud with Ansible to familiarize with automation and orchestrations techniques.

## What is Ansible?


## Ansible and GC

In order to create gce instances with Ansible on GCP we need to create credentials for ansible. 
This can be easily done throught the GCP gui:
- open the Navigation Menu
- click IAM & admin
  - select Service accounts
- click the Create service account on top of the screen.
- fill in the service account name: e.g. n5602-ansible
- fill in a description on the service account: e.g. manage servers with ansible
-


    $ gcloud  compute config-ssh
