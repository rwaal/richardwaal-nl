---
title: "Use Ansible To Deploy ARM Templates From Local File System"
description: "A small tip about deploying Azure Resource Manager templates with Ansible."
date: 2019-05-28
draft: false
author: richard

tags: [
    "ansible",
    "arm templates"
]
---

I have been working with Ansible a lot lately.  I wanted to share a small trick that I learned in [this stackoverflow thread](https://stackoverflow.com/questions/40068563/ansible-how-to-use-an-azure-rm-template-in-azure-rm-deployment-module), regarding deploying Azure Resource Manager (ARM) templates from Ansible.

<!--more-->
## azure_rm_deployment
Deploying an ARM template is done through the azure_rm_deployment Ansible module. The [documentation](https://docs.ansible.com/ansible/latest/modules/azure_rm_deployment_module.html) describes two options for providing a template:

- **Inline** >> You write the ARM template in YAML format inside the Ansible Playbook.
- **Template link** >> You provide a link to a public accessible ARM template, such as on Github.

I learned in the stackoverflow thread it's possible to simply deploy a template from your local file system, using the method below.

```yaml
- name: Create Azure Resource Group deployment
  azure_rm_deployment:
    state: present
    resource_group_name: myResourceGroup
    name: myDeployment
    location: West Europe
    template: "{{ lookup('file', 'azuredeploy.json') }}"
    parameters:
      siteName:
        value: myWebApp
      hostingPlanName:
        value: myHostingPlan
      sku:
        value: Standard
```

As you can see, the trick is to perform a [file lookup](https://docs.ansible.com/ansible/latest/plugins/lookup/file.html), which basically reads the contents of the file and loads it as a dictionary. The 'azuredeploy.json' is the name of my template file.

**That's it!**

Now you don't have to convert the JSON to YAML and put it inside your playbook. Which really is not a very elegant method of deploying templates. Also, there's no need to put your template on a public git repository, storage account, etc.
