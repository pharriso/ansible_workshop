# Exercise - Using Molecule To Test Your Roles

Molecule is designed to aid in the development and testing of Ansible roles. Molecule provides support for testing with multiple instances, operating systems and distributions, virtualization providers, test frameworks and testing scenarios.

Molecule uses Ansible playbooks to exercise the role and its associated tests. So we eat our own dog food :)

In this exercise, we'll use molecule in association with podman, a drop-in rootless replacement for docker, to spin up and test our role.

Note: molecule is an upstream open source project, very liable to change.

(This exercise was last tested against - Red Hat Enterprise Linux release 8.3 (Ootpa) - on 27 January 2021)


## Section 1: Installing Components

SSH into your control node.

### Step 1 - Docker

Now we can install podman and other dependencies for molecule.

```bash
sudo yum -y install podman
sudo systemctl enable podman.socket && sudo systemctl start podman
```

Podman should be running (loaded, active and running):
```bash
sudo systemctl status podman
● podman.service - Podman API Service
   Loaded: loaded (/usr/lib/systemd/system/podman.service; static; vendor preset: disabled)
   Active: active (running) since Wed 2021-01-27 12:15:17 UTC; 1s ago
     Docs: man:podman-system-service(1)
 Main PID: 167860 (podman)
    Tasks: 8 (limit: 23573)
   Memory: 31.8M
   CGroup: /system.slice/podman.service
           └─167860 /usr/bin/podman system service
```

### Step 2 - Molecule

We use pip inside a virtualenv to install molecule:

```bash
sudo yum -y install gcc python3-pip python3-devel openssl-devel libselinux-python3 libffi-devel git python3-virtualenv yamllint
virtualenv --system-site-packages ~/molecule
. ~/molecule/bin/activate
pip install --upgrade pip
pip install molecule molecule[podman] molecule[lint] 
```

```bash
$ molecule
Usage: molecule [OPTIONS] COMMAND [ARGS]...

   _____     _             _
  |     |___| |___ ___ _ _| |___
  | | | | . | | -_|  _| | | | -_|
  |_|_|_|___|_|___|___|___|_|___|

  Molecule aids in the development and testing of Ansible roles.

  Enable autocomplete issue:

    eval "$(_MOLECULE_COMPLETE=source molecule)"

Options:
  --debug / --no-debug    Enable or disable debug mode. Default is disabled.
  -c, --base-config TEXT  Path to a base config.  If provided Molecule will
                          load this config first, and deep merge each
                          scenario's molecule.yml on top.
                          (/home/student1/.config/molecule/config.yml)
  -e, --env-file TEXT     The file to read variables from when rendering
                          molecule.yml. (.env.yml)
  --version               Show the version and exit.
  --help                  Show this message and exit.

Commands:
  check        Use the provisioner to perform a Dry-Run...
  converge     Use the provisioner to configure instances...
  create       Use the provisioner to start the instances.
  dependency   Manage the role's dependencies.
  destroy      Use the provisioner to destroy the instances.
  idempotence  Use the provisioner to configure the...
  init         Initialize a new role or scenario.
  lint         Lint the role.
  list         Lists status of instances.
  login        Log in to one instance.
  matrix       List matrix of steps used to test instances.
  prepare      Use the provisioner to prepare the instances...
  side-effect  Use the provisioner to perform side-effects...
  syntax       Use the provisioner to syntax check the role.
  test         Test (lint, destroy, dependency, syntax,...
  verify       Run automated tests against instances.
  
$ molecule --version
molecule 3.2.2 using python 3.6
    ansible:2.9.16
    delegated:3.2.2 from molecule
    podman:0.3.0 from molecule_podman
```

## Section 2: Creating a New Role Framework

We'll use a simple apache role to test molecule.

### Step 1 - Initalise New Role

```bash
cd ~/ansible-files/roles
molecule init role apache_install --driver-name podman
INFO     Initializing new role apache_install...
Using /home/student1/.ansible.cfg as config file
- Role apache_install was created successfully
INFO     Initialized role in /home/student1/ansible-files/roles/apache_install successfully.
```

Let's have a look at what was created:

```bash
$ tree
.
└── apache_install
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── molecule
    │   └── default
    │       ├── converge.yml
    │       ├── molecule.yml
    │       └── verify.yml
    ├── README.md
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml
```

This command uses ansible-galaxy behind the scenes to generate a new Ansible role. It then injects a molecule directory in the role, and sets it up to run builds and test runs in a containerised environment.


## Section 3: Testing

### Step 1 - First Tests

Straight out the box, we should be able to do things:

```bash
cd apache_install
molecule create (check out 'docker images' and 'docker ps' output)
molecule verify
```

Hopefully that works, so you now have a test framework to work with.

### Step 2 - Further Testing

A typical dev cycle is : write some plays/roles -> molecule converge -> rinse and repeat...
Once you're happy you can commit your code to SCM.

```bash
molecule converge
--> Test matrix

└── default
    ├── dependency
    ├── create
    ├── prepare
    └── converge

--> Scenario: 'default'
--> Action: 'dependency'
Skipping, missing the requirements file.
Skipping, missing the requirements file.
--> Scenario: 'default'
--> Action: 'create'
Skipping, instances already created.
--> Scenario: 'default'
--> Action: 'prepare'
Skipping, prepare playbook not configured.
--> Scenario: 'default'
--> Action: 'converge'
--> Sanity checks: 'docker'

    PLAY [Converge] ****************************************************************

    TASK [Gathering Facts] *********************************************************
    ok: [instance]

    TASK [Include apache_install] **************************************************

    PLAY RECAP *********************************************************************
    instance                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

### Step 3 - Configuring Molecule

You can see from above that a few things have been skipped. You can tune a lot of the molecule config and what you want it to do by changing the molecule.yml file.

Let's define the test sequence we want. We'll add the test_sequence block under the default scenario.

Change molecule.yml to reflect this:

```bash
cat molecule/default/molecule.yml
---
dependency:
  name: galaxy
driver:
  name: docker
lint: yamllint .
platforms:
  - name: instance
    image: centos:7
provisioner:
  name: ansible
  lint:
    name: ansible-lint
scenario:
  name: default
  test_sequence:
    - lint
    - destroy
    - syntax
    - create
    - converge
#    - verify
    - destroy
```

Now do a test run. You should see something like this:

```bash
molecule test
--> Test matrix

└── default
    ├── lint
    ├── destroy
    ├── syntax
    ├── create
    ├── converge
    └── destroy

--> Scenario: 'default'
--> Action: 'lint'
--> Executing: yamllint .
--> Scenario: 'default'
--> Action: 'destroy'
--> Sanity checks: 'docker'

    PLAY [Destroy] *****************************************************************

    TASK [Destroy molecule instance(s)] ********************************************
    changed: [localhost] => (item=instance)

    TASK [Wait for instance(s) deletion to complete] *******************************
    ok: [localhost] => (item=None)
    ok: [localhost]

    TASK [Delete docker network(s)] ************************************************

    PLAY RECAP *********************************************************************
    localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

--> Scenario: 'default'
--> Action: 'syntax'

    playbook: /home/student1/ansible-files/roles/apache_install/molecule/default/converge.yml
--> Scenario: 'default'
--> Action: 'create'

    PLAY [Create] ******************************************************************

    TASK [Log into a Docker registry] **********************************************
    skipping: [localhost] => (item=None)

    TASK [Check presence of custom Dockerfiles] ************************************
    ok: [localhost] => (item=None)
    ok: [localhost]

    TASK [Create Dockerfiles from image names] *************************************
    changed: [localhost] => (item=None)
    changed: [localhost]

    TASK [Discover local Docker images] ********************************************
    ok: [localhost] => (item=None)
    ok: [localhost]

    TASK [Build an Ansible compatible image (new)] *********************************
    changed: [localhost] => (item=molecule_local/centos:8)

    TASK [Create docker network(s)] ************************************************

    TASK [Determine the CMD directives] ********************************************
    ok: [localhost] => (item=None)
    ok: [localhost]

    TASK [Create molecule instance(s)] *********************************************
    changed: [localhost] => (item=instance)

    TASK [Wait for instance(s) creation to complete] *******************************
    FAILED - RETRYING: Wait for instance(s) creation to complete (300 retries left).
    changed: [localhost] => (item=None)
    changed: [localhost]

    PLAY RECAP *********************************************************************
    localhost                  : ok=7    changed=4    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

--> Scenario: 'default'
--> Action: 'converge'

    PLAY [Converge] ****************************************************************

    TASK [Gathering Facts] *********************************************************
    ok: [instance]

    TASK [Include apache_install] **************************************************

    PLAY RECAP *********************************************************************
    instance                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

--> Scenario: 'default'
--> Action: 'destroy'

    PLAY [Destroy] *****************************************************************

    TASK [Destroy molecule instance(s)] ********************************************
    changed: [localhost] => (item=instance)

    TASK [Wait for instance(s) deletion to complete] *******************************
    changed: [localhost] => (item=None)
    changed: [localhost]

    TASK [Delete docker network(s)] ************************************************

    PLAY RECAP *********************************************************************
    localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

--> Pruning extra files from scenario ephemeral directory
```

### Step 4 - (Double) Testing Your Roles

testinfra, a python tool can be used as a verifier step for molecule. Testinfra uses pytest and makes it easy to test the system after the role is run to ensure our created role has the results that we expected. But as this requires python coding knowledge, this is beyond the scope of this exercise.

You would need to uncomment the verifier stage in molecule.yml (see above) for it to run any verification using this method.


### Step 5 - Dummy Full Test

Let's run molecule test to see the full cycle in action:

```bash
molecule test
--> Validating schema /home/student1/ansible-files/roles/apache_install/molecule/default/molecule.yml.
Validation completed successfully.
--> Test matrix

└── default
    ├── lint
    ├── destroy
    ├── syntax
    ├── create
    ├── converge
    ├── verify
    └── destroy

--> Scenario: 'default'
[output truncated...]
```

Let's not worry too much about this output as we've other things to do yet :)


## Section 4: Write The Role Tasks

So your role testing is useful, let's write the role contents!

```bash
vi ~/ansible-files/molecule_play.yml

---
- name: Main Playbook
  hosts: web
  become: "yes"

  roles:
    - apache_install
```

Not strictly necessary here, but I'm using the include_tasks directive to show that as well.

```bash
vi ~/ansible-files/roles/apache_install/tasks/main.yml

---
# tasks file for apache_install
- name: Include other playbooks
  include_tasks: install_apache.yml
```

```bash
vi ~/ansible-files/roles/apache_install/tasks/install_apache.yml

---
# tasks file for install_apache

- name: Install Apache
  yum:
    name: httpd
    state: present
  become: "yes"
```


## Section 5: Full Test Run

Let's first test the playbook to prove we've written something useful and workable:

```bash
ansible-playbook ~/ansible-files/molecule_play.yml

PLAY [Main Playbook] **********************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************
ok: [node1]
ok: [node2]
ok: [node3]

TASK [apache_install : Include other playbooks] *******************************************************************************
included: /home/student1/ansible-files/roles/apache_install/tasks/install_apache.yml for node1, node2, node3

TASK [apache_install : Install Apache] ****************************************************************************************
changed: [node1]
changed: [node2]
changed: [node3]

PLAY RECAP ********************************************************************************************************************
node1                      : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node2                      : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node3                      : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

So that worked :)

Now let's do a full on test using molecule:

```bash

cd ~/ansible-files/roles/apache_install
molecule test
--> Test matrix

└── default
    ├── lint
    ├── destroy
    ├── syntax
    ├── create
    ├── converge
    └── destroy

--> Scenario: 'default'
--> Action: 'lint'
--> Executing: yamllint .
--> Scenario: 'default'
--> Action: 'destroy'
--> Sanity checks: 'docker'

    PLAY [Destroy] *****************************************************************

    TASK [Destroy molecule instance(s)] ********************************************
    changed: [localhost] => (item=instance)

    TASK [Wait for instance(s) deletion to complete] *******************************
    ok: [localhost] => (item=None)
    ok: [localhost]

    TASK [Delete docker network(s)] ************************************************

    PLAY RECAP *********************************************************************
    localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

--> Scenario: 'default'
--> Action: 'syntax'

    playbook: /home/student1/ansible-files/roles/apache_install/molecule/default/converge.yml
--> Scenario: 'default'
--> Action: 'create'

    PLAY [Create] ******************************************************************

    TASK [Log into a Docker registry] **********************************************
    skipping: [localhost] => (item=None)

    TASK [Check presence of custom Dockerfiles] ************************************
    ok: [localhost] => (item=None)
    ok: [localhost]

    TASK [Create Dockerfiles from image names] *************************************
    changed: [localhost] => (item=None)
    changed: [localhost]

    TASK [Discover local Docker images] ********************************************
    ok: [localhost] => (item=None)
    ok: [localhost]

    TASK [Build an Ansible compatible image (new)] *********************************
    ok: [localhost] => (item=molecule_local/centos:8)

    TASK [Create docker network(s)] ************************************************

    TASK [Determine the CMD directives] ********************************************
    ok: [localhost] => (item=None)
    ok: [localhost]

    TASK [Create molecule instance(s)] *********************************************
    changed: [localhost] => (item=instance)

    TASK [Wait for instance(s) creation to complete] *******************************
    FAILED - RETRYING: Wait for instance(s) creation to complete (300 retries left).
    changed: [localhost] => (item=None)
    changed: [localhost]

    PLAY RECAP *********************************************************************
    localhost                  : ok=7    changed=3    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

--> Scenario: 'default'
--> Action: 'converge'

    PLAY [Converge] ****************************************************************

    TASK [Gathering Facts] *********************************************************
    ok: [instance]

    TASK [Include apache_install] **************************************************

    TASK [apache_install : Include other playbooks] ********************************
    included: /home/student1/ansible-files/roles/apache_install/tasks/install_apache.yml for instance

    TASK [apache_install : Install Apache] *****************************************
    changed: [instance]

    PLAY RECAP *********************************************************************
    instance                   : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

--> Scenario: 'default'
--> Action: 'destroy'

    PLAY [Destroy] *****************************************************************

    TASK [Destroy molecule instance(s)] ********************************************
    changed: [localhost] => (item=instance)

    TASK [Wait for instance(s) deletion to complete] *******************************
    FAILED - RETRYING: Wait for instance(s) deletion to complete (300 retries left).
    changed: [localhost] => (item=None)
    changed: [localhost]

    TASK [Delete docker network(s)] ************************************************

    PLAY RECAP *********************************************************************
    localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

--> Pruning extra files from scenario ephemeral directory
```

Molecule will roll through the stages we've defined, doing the necessary syntax/lint checks, starting with a clean docker slate by removing any old running instances, creating a new running image, 'converging' the playbook/role into it, testing it and then finally removing everything we've done.

## Summary: The Finished Playbook

You've explored the basics around using molecule for tesing Ansible.

Much of this content was based around Jeff Geerling's most excellent blog:
https://www.jeffgeerling.com/blog/2018/testing-your-ansible-roles-molecule

---

[Click Here to return to the Ansible Linklight - Ansible Engine Workshop](../README.md)
