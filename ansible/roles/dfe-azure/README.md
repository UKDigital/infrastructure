Azure deployment role
=====================

Infrastructure deployment into Microsoft Azure with the usage of Ansible templating of an ARM resource template

Requirements
------------

Ansible does not support the latest Azure CLI2.0 yet so it can't automatically pick up the login details
from the ~/.azure/ folder as the structure of those files seem to have changed and also the name of the parameters.

There are 2 ways one can achieve Azure auth with Ansible: configuration file in the .azure folder
or module parameters from inventory/passed in variables. This approach uses the
configuration file approach as it produces much less maintainable code.

Further on the Azure modules can use a service principal or a user account.

Example of ~/.azure/credentials file with service principal:

    [default]
    client_id=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    secret=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    tenant=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    subscription_id=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

A service principal is created as follows:

    az ad sp create-for-rbac --name AnsibleDeployment

This should result in an output similar to this:

    {
      "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "displayName": "AnsibleDeployment",
      "name": "http://AnsibleDeployment",
      "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }


NOTE: parameter names Ansible expects differ from the ones Azure returns in its response lately.
The credentials file need to be created manually and the `appId` and `password` parameters
need to be renamed to `client_id` and `secret` respectively

Role Variables
--------------

The role currently requires variables as follows:

        azure_machine_spec:
          aom2:
            size: Standard_DS1_v2
            os: rhel74
            instances:
              - hostname: "{{ azure_hostname_prefix }}aom21"
                interfaces:
                  - subnet: app2subnet
                    ipv4: "10.100.2.21"
                    public_ip: false
              - hostname: "{{ azure_hostname_prefix }}aom22"
                interfaces:
                  - subnet: app2subnet
                    ipv4: "10.100.2.22"
                    public_ip: false
              - hostname: "{{ azure_hostname_prefix }}aom23"
                interfaces:
                  - subnet: app2subnet
                    ipv4: "10.100.2.23"
                    public_ip: false
            disk_size_gb: 5
          aom3:
            size: Standard_E2s_v3
            ...
        azure_ssh_keys:
          - path: "/home/cloud-user/.ssh/authorized_keys"
            keyData: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB..."

Further default variables can be overridden. Please refer to default folder

Dependencies
------------

In order for this role to work ansible must be installed with the azure extras package

    pip install ansible[azure]

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: localhost
      roles:
        - dfe-azure

      vars_prompt:
        - name: azure_credential_profile
          prompt: Section in ~/.azure/credentials file to use
          default: "default"
