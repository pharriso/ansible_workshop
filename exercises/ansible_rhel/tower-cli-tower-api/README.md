# Exercise - Using Tower CLI

We provide an official CLI tool for interacting with Ansible Tower. As per the [docs](https://docs.ansible.com/ansible-tower/latest/html/towercli/index.html), use cases include:

* Configuring and launching jobs/playbooks

* Checking on the status and output of job runs

* Managing objects like organizations, users, teams, etc…

### Installing Tower CLI

Let's install Tower CLI on the Ansible control node. As your student user on ansible-1 run the following:

```bash
sudo dnf config-manager --add-repo https://releases.ansible.com/ansible-tower/cli/ansible-tower-cli-el8.repo
sudo dnf install ansible-tower-cli -y
```

### Authentication with Tower CLI

There are different ways of configuring authentication for Tower CLI. We will eventually be running our Tower configuration as part of a CI/CD pipeline so we'll use environment variables to set our authentication. As your student user on ansible-1, run the following commands to set the username and password. Replace **yourpassword** with the admin password.

```bash
export TOWER_USERNAME=admin && export TOWER_VERIFY_SSL=false && export TOWER_PASSWORD=yourpassword
```

### Exploring the CLI

First, let's explore all of the options available to the command line tool:

```bash
awx-cli --help
```

Next, let's list all of the job templates:

```bash
awx-cli job_template list
```

The output should look something like this

```bash
== ============================================ ========= ======= ==================================================== 
id                     name                     inventory project                       playbook                       
== ============================================ ========= ======= ==================================================== 
10 INFRASTRUCTURE / Turn off IBM Community Grid         2       9 playbooks/infrastructure/turn_off_community_grid.yml
12 Install Apache                                       2      11 rhel/apache/apache_install.yml
== ============================================ ========= ======= ====================================================
```

### Launching a job

Let's launch the **Install Apache** job again.

```bash
awx-cli job launch -J "Install Apache"
```

You can check the status of the job as follows (replace job ID with the one returned from the previous command):

```bash
awx-cli job status 8
```

Once the job shows a status of "successful", let's look at the standard output from the job:

```bash
awx-cli job stdout 8
```

You should see the familiar Ansible job output

```bash
PLAY [Apache server installed] *************************************************

TASK [Gathering Facts] *********************************************************
ok: [node2]
ok: [node1]
ok: [node3]

TASK [latest Apache version installed] *****************************************
ok: [node1]
ok: [node3]
ok: [node2]
```

### Launching a job and waiting

So far we can see that the CLI makes it very easy to interact with the Tower API. But we are still stringing together multiple commands to launch and then poll jobs. This time we are going to launch the job and wait for the return code:

```bash
awx-cli job launch -J "Install Apache" --wait
```

You should see something like this 

```bash
Resource changed.          
== ============ =========================== ========== ======= 
id job_template           created             status   elapsed 
== ============ =========================== ========== ======= 
10           12 2021-07-12T13:54:30.805366Z successful 14.714
== ============ =========================== ========== =======
```

## Summary

Tower CLI makes it very easy to automate the interaction with the Tower API and provides a number of short-cuts when compared with writing your own script.

---

[Click Here to return to the Ansible Tower Workshop](../README.md)
