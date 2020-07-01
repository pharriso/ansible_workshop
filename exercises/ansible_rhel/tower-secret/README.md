# Exercise - Tower External Secret Management

Tower has it's own built in secret management which we have used to store credentials. But, we can also integrate Tower with external secret management systems like Hashicorp Vault, CyberArk and Azure Key Manager.

This exercise presents 2 examples of such integrations: Azure Key Manager and Hashicorp Vault.

Here is a link to the Ansible documentation on [Secret Management](https://docs.ansible.com/ansible-tower/latest/html/userguide/credential_plugins.html)


# Azure Key Manager

## Create user account on node1

From your Ansible control node, add a user called **azure-user** to node1 with the password **Redhat123**.

```bash
ansible node1 -m user -a "name=azure-user password={{ 'Redhat123' | password_hash('sha512', 'mysecretsalt') }}" -b
```

Verify you can log in with that user and password combination. First find the IP address of node1:

```bash
$ grep node1 ~/lab_inventory/hosts 
node1 ansible_host=18.195.124.139
```

Now login as azure-user and password Redhat123 (replace IP below with your IP for node1):

```bash
ssh azure-user@18.195.124.139
```

## Create an Azure Vault credential in Tower

In the Tower UI, create a credential that will allow Tower to query the Azure Vault.

In the **RESOURCES** menu choose **Credentials**. Now:

Click the green **+** button to add a new credential:
    
  - **NAME:** Azure Vault

  - **ORGANIZATION:** Default

  - **CREDENTIAL TYPE:** Click on the magnifying glass, pick **Microsoft Azure Key Vault** 

  - **Vault URL:** To be provided by instructor

  - **Client ID:** To be provided by instructor

  - **Client Secret:** To be provided by instructor

  - **Tenant ID:** To be provided by instructor

  - Click **TEST**

  - For the **Secret Name** Enter **password**. You should see a message appear in the top right corner saying **Test Passed**.

  - Click **Close** to close the Test dialog box.

  - Click **Save**

## Create a credential that will use a secret from Azure Vault

In the Tower UI, create a credential that will use an existing secret in Azure Vault.

In the **RESOURCES** menu choose **Credentials**. Now:

Click the green **+** button to add a new credential:
    
  - **NAME:** azure-user

  - **ORGANIZATION:** Default

  - **CREDENTIAL TYPE:** Click on the magnifying glass, pick **Machine** 

  - **USERNAME:** azure-user

  - **PASSWORD:** Press the magnifying glass in the password box. You should see **azure vault** in the list. Select **azure vault** and then click on the **METADATA** button. In the **SECRET NAME** enter **password**

  - Click **OK**

  - Click **SAVE**

## Test the Integration

To test the integration we will run an ad-hoc job against node1. In the Tower UI navigate to **Inventories**, **Workshop Inventory** and then click the **Hosts** button. Select **node1** from the list and then press the **Run commands** button.

For the module select **setup**. For the Machine Credential use **azure-user**. Press **Launch**

Tower will now go to Azure Vault to retrieve the relevant secret and you should see a succesful job completion. 

## Break the Integration

As a final test, change the password for azure-user on node1 so that it no longer matches the value in Azure Vault. From your Ansible control node run the following:

```bash
ansible node1 -m user -a "name=azure-user password={{ 'IBM123' | password_hash('sha512', 'mysecretsalt') }}" -b
```

Now re-launch the ad-hoc job against node1 (navigate to **Jobs** and hit the launch icon next to the setup job you just ran). You should see a failed login attempt.


# Hashicorp Vault

## Install Hashicorp Vault

We'll install Vault on your Ansible **control** node. 

Login to your student control node and run this to install and base configure the Vault:

```bash
cd && git clone https://github.com/ffirg/HashiVaultRole && cd HashiVaultRole
ansible-playbook hashivault.yml
```

It should now be running on the control node using HTTP port 8000. We can check using:

```bash
ss -nlp | grep 8000         [ check we're up and listening on port 8000 ]
curl localhost:8000/ui/     [check the UI, which we also install for convenience]
systemctl status vault      [ check our newly created systemd vault sercice is running]
```

## Adding a Secret

We can use the secrets.yml playbook also in this repo to add some credentials. But first we must authenticate with Vault.

```bash
export VAULT_ADDR="http://localhost:8000"
```

This sets where vault is. We can now login using a token. When we installed the system a **rootKey** was generated. We'll use that.

```bash
awk -F[ '{print $1}' rootKey/rootkey
s.83I8gWop4i79dFWWoiu3YuqC [yours will be different!]
```

Login using the above key:

```bash
vault login
Token (will be hidden):
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.
Key                  Value
---                  -----
token                s.83I8gWop4i79dFWWoiu3YuqC
token_accessor       cRlwkLm3UmiSwShvSsW9pJnE
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
```

tip: if you ever need to know what your token is again, you can also use this:
vault token lookup | grep '^id' | awk '{print $2}'

Before we can run our next playbook, we need to install some helpful modules:

```bash
sudo pip3 install ansible-modules-hashivault
```

We can now add some credentials. Run the following **verbose** playbook, supplying **your** assigned student/password details:

```bash
ansible-playbook secrets.yml -v
Using /home/student1/.ansible.cfg as config file
Enter your student number : student1
Enter your student password : redhat

PLAY [localhost] **********************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************
ok: [localhost]

TASK [hashivault_status] **************************************************************************************************
ok: [localhost] => changed=false
  rc: 0
  status:
    cluster_id: 5ab41459-68ea-7a58-a0f6-a8a3007f2ec5
    cluster_name: vault-cluster-be3d8d1f
    initialized: true
    migration: false
    n: 5
    nonce: ''
    progress: 0
    recovery_seal: false
    sealed: false
    storage_type: file
    t: 3
    type: shamir
    version: 1.3.2

TASK [hashivault_write] ***************************************************************************************************
changed: [localhost] => changed=true
  msg: Secret kv/******** written
  rc: 0

TASK [hashivault_read] ****************************************************************************************************
ok: [localhost] => (item=username) => changed=false
  ansible_loop_var: item
  item: username
  lease_duration: 2764800
  lease_id: ''
  rc: 0
  renewable: false
  value: student1
ok: [localhost] => (item=password) => changed=false
  ansible_loop_var: item
  item: password
  lease_duration: 2764800
  lease_id: ''
  rc: 0
  renewable: false
  value: Redhat123

PLAY RECAP ****************************************************************************************************************
localhost                  : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

You can see we've created a key-value 'kv' secret with the details you supplied, using the hashivault_write module.
We then use the hashivault_read module to show us what's been added. So we have **username** and **password** pairs stored.

## Create a Hashicorp Vault Credential in Tower

In the Tower UI, create a credential that will allow Tower to query the Hashicorp Vault.

In the **RESOURCES** menu choose **Credentials**. Now:

Click the green **+** button to add a new credential:
    
  - **NAME:** Hashicorp Vault

  - **ORGANIZATION:** Default

  - **CREDENTIAL TYPE:** Click on the magnifying glass, pick **HashiCorp Vault Secret Lookup** 

  - **Server URL:** http://localhost:8000

  - **Token:** your_rootKey

  - Click **TEST**

  - Leave **Name of Secret Backend** blank
  - **Path to Secret:** /kv/studentX
  - **Key Name:** password
  
  - Click **RUN**
  
  You should see a message appear in the top right corner saying **Lookup: Test passed**.

  - Click **Close** to close the Test dialog box.

  - Click **Save**

## Create a credential that will use a secret from Hashicorp Vault

In the Tower UI, create a credential that will use the secret we've created in Hashicorp Vault.

In the **RESOURCES** menu choose **Credentials**. Now:

Click the green **+** button to add a new credential:
    
  - **NAME:** hashivault-user

  - **ORGANIZATION:** Default

  - **CREDENTIAL TYPE:** Click on the magnifying glass, pick **Machine** 

  - **USERNAME:** studentX

  - **PASSWORD:** Press the magnifying glass in the password box. You should see **Hashicorp Vault** in the list. Select **Hashicorp Vault** and then click on the **NEXT**. 
  
  In the **PATH TO SECRET** enter **/kv/studentX**
  In the **KEY NAME** enter **password**

  - Click **TEST**
  
  You should see a message appear in the top right corner saying **Lookup: Test passed**.
  
  - Click **OK**

  - Click **SAVE**

## Test the Integration

To test the integration we will run an ad-hoc job against node1. In the Tower UI navigate to **Inventories**, **Workshop Inventory** and then click the **Hosts** button. Select **node1** from the list and then press the **Run commands** button.

For the module select **setup**. For the Machine Credential select **hashivault-user**. Press **Launch**

Tower will now go to Hashicorp Vault to retrieve the relevant password and you should see a succesful job completion.

You have now integrated Hashicorp Vault Secrets Engine with Ansible Tower :)

---

[Click Here to return to the Ansible Tower Workshop](../README.md)
