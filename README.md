# CASC-DEMO


## Description

This repository give us a place to maintain a control versions of the every object in an Ansible Automation Controller.

CasC (Configuration as Code)  means the possibility of define every object of Ansible Automation Controller as code in a git repository. In this lab, we have defined two environments (dev and pro) to do the CasC and interact with a gitops approach between them.

## Steps to test the CasC approach
### Day-Zero from CLI

   > **_NOTE:_** Execution Environment called ee-casc in the following playbooks has six needed collections (You can use a EE with this collections inside or install each collection in your workspace. If you will not use a EE, you can skip "podman login" steps.):

    - name: ansible.controller
    - name: ansible.utils
    - name: ansible.posix
    - name: community.general
    - name: infra.controller_configuration
    - name: automationiberia.casc_setup

Before using CasC as a GitOps approach, it is needed to launch an initialization from CLI which it is called Day-Zero.

   > **_NOTE:_**  Copy this repo to your own gitlab organization and play with it instead of the example we use during following steps.

1. Clone the repository and create a new day-zero branch

   ```
   git clone git@gitlab.com:casc/CASC-ORG.git
   cd CASC-ORG
   git checkout -b casc-dev-day0
   ```

2. Edit credentials to connect to the controller for day zero.

   ```
   vi group_vars/dev/configure_connection_controller_credentials.yml
   vi group_vars/pro/configure_connection_controller_credentials.yml
   ansible-vault encrypt group_vars/dev/configure_connection_controller_credentials.yml group_vars/pro/configure_connection_controller_credentials.yml
   ```

3. Edit credentials for day zero

   ```
   vi orgs_vars/CASC-ORG/env/dev/controller_credentials.d/controller_credentials.yml
   vi orgs_vars/CASC-ORG/env/pro/controller_credentials.d/controller_credentials.yml
   ansible-vault encrypt orgs_vars/CASC-ORG/env/pro/controller_credentials.d/* orgs_vars/CASC-ORG/env/dev/controller_credentials.d/*
   ```

4. Check the inventory file. Example:

   ```
   [dev]
   controller.domain.com

   [pro]
   controller.domain.com
   ```

5. Setting vault credential file

   ```
   echo "my_vault_pass" > ~/.vault_password
   ln ~/.vault_password .
   ```

6. Launch ansible-navigator from CLI to setup day-zero of CasC. Example:

   ```
   ansible-navigator run casc_ctrl_config.yml -i inventory -l dev -e '{orgs: CASC-ORG, dir_orgs_vars: orgs_vars, env: dev}' -m stdout --eei quay.io/automationiberia/aap/ee-casc --vault-password-file .vault_password
   ansible-navigator run casc_ctrl_config.yml -i inventory -l pro -e '{orgs: CASC-ORG, dir_orgs_vars: orgs_vars, env: pro}' -m stdout --eei quay.io/automationiberia/aap/ee-casc --vault-password-file .vault_password
   ```

7. Push the changes

   ```
   git status -s
   git add -A
   git commit -m "CasC day zero"
   git push origin casc-dev-day0
   ```

8. Pomote the `casc-dev-day0` branch to dev (`dev` branch)

   * Select the source branch as `casc-dev-day0` and `dev` as the destination one]
     ![New Merge Request Dev](images/newmrtodev-0.png "New Merge Request Dev")

   * Fill in the merge request information
     ![Fill in the merge request information](images/newmrtodev-0-step2.png "Fill in the merge request information")

   * Approve the Merge Request.
     ![New Merge Request Pro](images/newmrtodev-0-step3.png "Merge the merge request")

9. Promote the `dev` branch to pro (`pro` branch)

   * Select the source branch as `dev` and `pro` as the destination one]
     ![New Merge Request Pro](images/newmrtopro.png "New Merge Request Pro")

   * Fill in the merge request information
     ![Fill in the merge request information](images/newmrtoprostep2.png "Fill in the merge request information")

     > :warning: **Be sure to write a title that have sense for the Merge Request**: The default value here is `dev`, that is not usefull at all!

   * Approve the Merge Request.
     ![New Merge Request Pro](images/newmrtoprostep3.png "Merge the merge request")

10. Configure the webhooks for both environments: DEV and PRO.

    MANUALLY IN DEV (only if you didn't by the playbook):

    1. Go to Dev Controller and open CASC-ORG CasC_AAP_Workflow
    2. Copy the content of "Webhook URL" and "Webhook Key"
    3. Go to Gitlab -> Settings -> Webhooks
    4. Paste the content of "Webhook URL" and "Webhook Key" in the gaps "URL" and "Secret Content"
    5. Select Push events and fill the gap with dev

    ![pusheventdev](images/pusheventdev.png)

    MANUALLY IN PRO (only if you didn't by the playbook):

    1. Go to PRO Controller and open CASC-ORG CasC_AAP_Workflow
    2. Copy the content of "Webhook URL" and "Webhook Key"
    3. Go to Gitlab -> Settings -> Webhooks
    4. Paste the content of "Webhook URL" and "Webhook Key" in the gaps "URL" and "Secret Content"
    5. Select Tag events and fill the gap with dev

    ![pusheventdev](images/tageventpro.png)

### GitOps flow

1. Clone the given repository:

   ```
   git clone git@gitlab.com:casc/CASC-ORG.git

   cd CASC-ORG/
   ```

2. Create a new branch from `dev` to introduce the new items:

   ```
   git checkout dev
   git checkout -b add_info_job_template
   ```

3. Add a new Playbook and a new Job Template

   File: `new_playbook1.yaml`
   ```
   cat > new_playbook1.yaml <<EOF
   ---
   - name: "Play to show the hostname"
     hosts: all
     tasks:
       - name: "Show the hostname"
         debug:
           msg:
             - "This server is called (from Ansible inventory):     {{ inventory_hostname }}"
             - "This server is called (from Execution Environment): {{ lookup('pipe', 'cat /etc/hostname') }}"
             - "Running as user: {{ lookup('pipe', 'id') }}"
   ...
   EOF
   ```

   File: `new_playbook2.yaml`
   ```
   cat > new_playbook2.yaml <<EOF
   ---
   - name: "Play to show the hostname"
     hosts: all
     connection: local
     tasks:
       - name: "Show the hostname"
         debug:
           msg:
             - "This server is called (from Ansible inventory):     {{ inventory_hostname }}"
             - "This server is called (from Execution Environment): {{ lookup('pipe', 'cat /etc/hostname') }}"
             - "Running as user: {{ lookup('pipe', 'id') }}"
   ...
   EOF
   ```

   File: `orgs_vars/CASC-ORG/env/common/controller_job_templates.d/new_job_template.yaml`
   ```
   cat > orgs_vars/CASC-ORG/env/common/controller_job_templates.d/new_job_template.yaml <<EOF
   ---
   controller_templates:
     - name: "{{ orgs }} New Job Template"
       description: "Template to show how to add a new JT"
       organization: "{{ orgs }}"
       project: "{{ orgs }} CasC_Data"
       inventory: "{{ orgs }} Localhost"
       playbook: "new_playbook1.yaml"
       job_type: run
       fact_caching_enabled: false
       concurrent_jobs_enabled: true
       ask_scm_branch_on_launch: true
       extra_vars:
         ansible_python_interpreter: /usr/bin/python3
         ansible_async_dir: /home/runner/.ansible_async/
       execution_environment: "ee-casc"
   ...
   EOF
   ```

4. Commit the changes to the new branch

   ```
   git add -A .
   git commit -am "Add new playbook and job template to show server information"
   git push -u origin add_info_job_template
   ```

5. Create a Merge Request to `dev` branch <a name="mrtodev"></a>

   * Go to Merge Requests and create a new merge request
     ![New merge request](images/mrtodev.png "New merge request")

   * Select the source branch `add_info_job_template` and `dev` as the destination one
     ![New merge request to dev](images/newmrtodev.png "New merge request to dev")

   * Fill in the merge request information
     ![Fill in the merge request information](images/newmrtodevstep2.png "Fill in the merge request information")

   * Merge the merge request
     ![Merge the merge request](images/newmrtodevstep3.png "Merge the merge request")

   ---

   The following automated process has ben executed at the Ansible Automation Controller:

   ![Look at the Ansible Automation Controller's jobs](images/devworkflowjobs.png "Look at the Ansible Automation Controller's jobs")

   The following diagram shows the components of the workflow:

   ![Workflow Diagram](images/workflowdiagram.png "Workflow Diagram")

   Of course, the new Job Template has been created:

   ![New Job Template Dev](images/newjtdev.png "New Job Template Dev")

6. Pomote the `dev` branch to production (`pro` branch) <a name="mrdevtopro"></a>

   Similarly to the [step 5](#mrtodev), create a new Merge Request from the `dev` branch to the `pro` branch:

   * Select the source branch as `dev` and `pro` as the destination one]
     ![New Merge Request Pro](images/newmrtopro.png "New Merge Request Pro")

   * Fill in the merge request information
     ![Fill in the merge request information](images/newmrtoprostep2.png "Fill in the merge request information")

     > :warning: **Be sure to write a title that have sense for the Merge Request**: The default value here is `dev`, that is not usefull at all!

   ---

   When the Merge Request is already merged, the new Job Template is also created in the `pro` environment:

   ![New Job Template Pro](images/newjtpro.png "New Job Template Pro")

7. Run the new Job Template at PRO <a name="runjtpro"></a>

   Run the Job Template:
   ![Run the JT](images/launchjtpro.png "Run the JT")

   and check that it is failing:
   ![Job Template PRO failed](images/jtprofailed.png "Job Template PRO failed")

8. Rollback the PRO environment to previously working tag

   To rollback the status of the controller to a previous working version, it's only needed to run the following Job Templates:

   * Run the Job Template `casc-twitch-demo CasC_JobTemplates_AAP_Drop_Diff` with the previous working version:
     ![Run the JT casc-twitch-demo CasC_JobTemplates_AAP_Drop_Diff](images/rollbackddv0.2.png "Run the JT casc-twitch-demo CasC_JobTemplates_AAP_Drop_Diff")
   * Run the Job Template `casc-twitch-demo CasC_JobTemplates_AAP_CD_Config_Controller` with the previous working version:
     ![Run the JT casc-twitch-demo CasC_JobTemplates_AAP_CD_Config_Controller](images/rollbackccv0.2.png "Run the JT casc-twitch-demo CasC_JobTemplates_AAP_CD_Config_Controller")

9. Fix your playbook

   ```
   git checkout dev
   git pull
   git checkout -b fix_playbook
   ```

   Modify the Job Template to use the correct playbook:

   `playbook: "new_playbook2.yaml"`

   Updated file: `orgs_vars/CASC-ORG/env/common/controller_job_templates.d/new_job_template.yaml`
   ```yaml
   ---
   controller_templates:
     - name: "{{ orgs }} New Job Template"
       description: "Template to show how to add a new JT"
       organization: "{{ orgs }}"
       project: "{{ orgs }} CasC_Data"
       inventory: "{{ orgs }} Localhost"
       playbook: "new_playbook2.yaml"
       job_type: run
       fact_caching_enabled: false
       concurrent_jobs_enabled: true
       ask_scm_branch_on_launch: true
       extra_vars:
         ansible_python_interpreter: /usr/bin/python3
         ansible_async_dir: /home/runner/.ansible_async/
       execution_environment: "ee-casc"
   ...
   ```

   Commit and push your changes:

   ```
   git commit -am "Fix the connection method"
   git push -u origin fix_playbook
   ```

   Create a Merge Request to `dev` branch:

   * ![Create New Merge Request from `fix_playbook` to Dev](images/newmrtodev-fix_playbook.png "Create New Merge Request from `fix_playbook` to Dev")
   * ![New Merge Request from `fix_playbook` to Dev](images/newmrtodev-fix_playbook-step2.png "New Merge Request from `fix_playbook` to Dev")
   * ![Merge the Request](images/newmrtodev-fix_playbook-step3.png "Merge the Request")

   Repeat the steps to create a new Merge Request from `dev` to `pro`, as described at [step 6](#mrdevtopro)

10. Run again the new Job Template at PRO <a name="runjtpro"></a>

    Run again the Job Template:
    ![Run the JT](images/launchjtpro.png "Run the JT")

    and check that it is working fine now:
    ![Job Template PRO failed](images/jtprosuccess.png "Job Template PRO failed")
