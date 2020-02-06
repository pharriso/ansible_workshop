# Exercise - Tower External Secret Management

Tower has it's own built in secret management which we have used to store credentials. But, we can also integrate Tower with external secret management systems - namely Hashicorp Vault, CyberArk and Azure Key Manager.

In this lab we will use Azure Key Manager to show this integration.

Here is a link to the Ansible documentation on [Secret Management](https://docs.ansible.com/ansible-tower/latest/html/userguide/credential_plugins.html)


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

  - For the **Secret Name** Enter password. You should see a message appear in the top right corner saying **Test Passed**.

  - Click **Close** to close the Test dialog box.

  - Click **Save**

## Create a credential that will use a secret from Azure Vault

In the Tower UI, create a credential that will use an existing secret in Azure Vault.

In the **RESOURCES** menu choose **Credentials**. Now:

Click the green **+** button to add a new credential:
    
  - **NAME:** Vault Machine Credential 

  - **ORGANIZATION:** Default

  - **CREDENTIAL TYPE:** Click on the magnifying glass, pick **Machine** 

  - **USERNAME:** azure-user

  - **PASSWORD:** Press the magnifying glass in the password box. You should see **azure vault** in the list. Select **azure vault** and then click on the **METADAT** button. In the **SECRET NAME** enter **password**

  - Click **OK**

  - Click **SAVE**

## Test the Integration

To test the integration we will run an ad-hoc job against node1. In the Tower UI navigate to **Inventories**, **Workshop Inventory** and then click the **Hosts** button. Select **node1** from the list and then press the **Run commands** button.

For the module select **setup**. For the Machine Credential use **Vault Machine Credential**. Press **Launch**

Tower will now go to Azure Vault to retrieve the relevant secret and you should see a succesful job completion. 

## Break the Integration

As a final test, change the password for azure-user on node1 so that it no longer matches the value in Azure Vault. From your Ansible control node run the following:

```bash
ansible node1 -m user -a "name=azure-user password={{ 'IBM123' | password_hash('sha512', 'mysecretsalt') }}" -b
```

Now re-launch the ad-hoc job against node1 (navigate to **Jobs** and hit the launch icon next to the setup job you just ran). You should see a failed login attempt.

---

[Click Here to return to the Ansible Tower Workshop](../README.md)
