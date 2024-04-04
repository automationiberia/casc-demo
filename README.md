# CASC-DEMO


## Description

This repository give us a place to maintain a control versions of the every object in an Ansible Automation Controller.

## Steps to test the CasC approach for Summit24

   > **_NOTE:_** Execution Environment called ee-casc in the following playbooks has six needed collections (You can use a EE with this collections inside or install each collection in your workspace. If you will not use a EE, you can skip "podman login" steps.):

    - name: ansible.controller
    - name: ansible.utils
    - name: ansible.posix
    - name: community.general
    - name: infra.controller_configuration

For this Demo, we assume Day-Zero is already applied.

1. Comment old playbook and uncomment new playbook in orgs_vars/CASC-ORG/env/common/controller_job_templates.d/controller_job_templates_demo.yml

2. Create several resources through GUI and launch Drop Diff JT.