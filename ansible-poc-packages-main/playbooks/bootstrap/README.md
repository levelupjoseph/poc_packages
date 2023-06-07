# ansible-poc-packages

# POC Prep Steps

NOTE: This `poccheck.sh` script checks a non-exhaustive list of configurations which may prevent a successful AAP POC install. It is intended to be downloaded, reviewed and then run by the customer in order to validate that the POC Install Session can proceed as scheduled. The Level Up architect reviews the script output. It can be run either independently by the customer prior to the POC Install Session, or at the beginning of the POC Install Session. The script output offers fix-me suggestions where feasible. Level Up's strong preference is to move forward with the POC Install Session whenever possible, but it is up to the Level Up architect leading the POC to decide on a go/no go for the POC Install Session.

1. Ask the customer to download and review the POC check script, then run it and show or send us the results: `wget https://levelupworkshops.s3.us-west-2.amazonaws.com/ansible-pocs/poccheck.sh`.
1. Alternatively, this script can be run directly via `bash <(curl -s https://levelupworkshops.s3.us-west-2.amazonaws.com/ansible-pocs/poccheck.sh)`.

# POC Bootstrap Steps

## Getting Started

Note: This assumes that the Level Up Architect is driving most of the hands-on keyboard parts of the install. This should be clearly negotiated in advance with the customer.

1. Install AAP as usual.
1. Login to https://<AAP_HOSTNAME_OR_IP_ADDRESS>/.
1. Add AAP subscription as usual.
1. Guide the customer through obtaining a GitHub (or equivalent) PAT: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token
1. *During an actual POC install:* Ask the customer to enter Machine credentials for AAP to "POC AAP Controller".
1. *During an actual POC install:* Create a Vault credential named "Bootstrap".
1. Add a "Bootstrap" inventory.
1. Add <AAP_HOSTNAME_OR_IP_ADDRESS> to the "Bootstrap" inventory.
1. Confirm access to <AAP_HOSTNAME_OR_IP_ADDRESS> via ad hoc ping.
1. Access > Users > admin > Tokens > Add.
    - Leave other fields blank.
    - Scope: Write.
    - Save.
    - Copy token to clipboard.
    - *During an actual POC install:* Go back to the "POC AAP Controller" Credential and paste the token into the Description so that you don't have to worry about saving it somewhere on the customer's computer. Then Save again.
1. Execution Environments: Add: Name: "Bootstrap" Image: quay.io/danielgoosen/controller-ee Pull: Always.
1. Add a Source Control credential named "Ansible POC Packages Source Control". Enter the customer's PAT.
1. Create a Source Control credential named "Bootstrap".
1. Create a project called "Bootstrap" and pointed at the repo created for this POC (e.g., https://github.com/1eve1up/xyz-ansible-bootstrap). Select the "Bootstrap" EE.
1. *During an actual POC install:* We will actually create add 2 projects and 2 separate source control credentials, one for Bootstrap and another for the customer's POC package code. We WILL chat the customer the Bootstrap PAT. But we will delete the Bootstrap project's PAT immediately AFTER the end of ANY POC install, whether considered successful or not!
1. Create a job template: 
    - Name: "Bootstrap".
    - Inventory: "Bootstrap".
    - Project: "Bootstrap".
    - Execution Environment: "Bootstrap".
    - Playbook: "playbooks/bootstrap/main.yml".
    - Credentials: Machine: "POC AAP Controller".
    - *During an actual POC install:* Credentials: Vault: "Bootstrap".
    - Limit (check Prompt on launch): <AAP_HOSTNAME_OR_IP_ADDRESS>.
    - Save.
    - Add a Survey:
      - github_password: password. Default answer: the PAT.
      - controller_oauthtoken: password. Default answer: the token you created earlier.
      - poc_package: poc_package. Default answer: the package you're installing (e.g., "rhel_patching").
      - scm_url: Default answer: the customer's POC code repo.
    - Save.
    - Toggle "Survey Enabled"
1. *During an actual POC install:* Now delete the token from the "POC AAP Controller" Description field.
1. Launch the "Bootstrap" job template.
1. Verify success. Verify that job templates have appeared.
1. Add host(s) to the "POC" inventory.
1. Run some job templates.
1. *During an actual POC install:* Delete EVERYTHING you named "Bootstrap" in the previous steps BEFORE the end of ANY POC install, whether considered successful or not. 

# TODOS:
1. Source control credential add is not working yet
1. Machine creds not populating username and password yet
1. ansible.controller collection won't install on local AAP server
1. playbook so far won't run against itself on vagrant (probably localhost related)
1. need to map out how to inject the config into the AAP environment and then have it clean itself up leaving only the config behind (a PAT that we remove immediately upon success, while providing zip file of poc package's role? or a separate repo we build per customer for 60 days?)
1. Defining organization: "Default" does NOT work for Project or Job Template tasks and breaks both.
1. Put Controller in its own inventory
1. Create a dict for the template vars to pick privilege execution or no; ignore_errors
1. poc network automation, all playbooks, and tutorials: builder, nav, and something solid for windows and networking
1. All linux-discovery.yml debug tasks need to be named

# FIXED:

1. Creds not applying to templates
1. Update revision on launch

## How to get all of the tags for each poc package:

```
$ ansible-playbook rhel_patching.yml --list-tags
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

playbook: rhel_patching.yml

  play #1 (all): all    TAGS: []
      TASK TAGS: [add_repository, configure_automatic_updates, configure_insights_client, disable_repository, disable_services, downgrade_package, enable_repository, enable_services, firewalld, install_insights_client, install_packages, install_satellite_client, install_security_updates, install_specific_version, kernel_parameters, list_installed_packages, list_updates, register_insights_client, register_insights_client_status, register_satellite_client, remove_insights_client, remove_packages, remove_repository, run_insights_client, selinux_state, unregister_insights_client, unregister_satellite_client, update_cache]
(ender14) daniel@Daniels-MBP (08:56:55) 47: rhel_patching$ 
```
