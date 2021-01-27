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
    ansible:2.10.5
    delegated:3.2.2 from molecule
    podman:0.3.0 from molecule_podman
```

## Section 2: Creating a New Role Framework

We'll use a simple apache role to test molecule.

### Step 1 - Initalise New Role

```bash
mkdir -p ~/ansible-files/roles
cd ~/ansible-files/roles
molecule init role apache_install --driver-name podman
INFO     Initializing new role apache_install...
...
- Role apache_install was created successfully
INFO     Initialized role in /home/.../ansible-files/roles/apache_install successfully.
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

### Step 1 - Molecule Basics

Straight out the box, we should be able to do things:

```bash
cd apache_install
molecule create
INFO     default scenario test matrix: dependency, create, prepare
INFO     Running default > dependency
WARNING  Skipping, missing the requirements file.
WARNING  Skipping, missing the requirements file.
INFO     Running default > create
INFO     Sanity checks: 'podman'

PLAY [Create] ***********************************************************************************************************************

TASK [Log into a container registry] ************************************************************************************************
skipping: [localhost] => (item={'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True})

TASK [Check presence of custom Dockerfiles] *****************************************************************************************
ok: [localhost] => (item={'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True})

TASK [Create Dockerfiles from image names] ******************************************************************************************
skipping: [localhost] => (item={'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True})

TASK [Discover local Podman images] *************************************************************************************************
ok: [localhost] => (item={'changed': False, 'skipped': True, 'skip_reason': 'Conditional result was False', 'item': {'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True}, 'ansible_loop_var': 'item', 'i': 0, 'ansible_index_var': 'i'})

TASK [Build an Ansible compatible image] ********************************************************************************************
skipping: [localhost] => (item={'changed': False, 'skipped': True, 'skip_reason': 'Conditional result was False', 'item': {'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True}, 'ansible_loop_var': 'item', 'i': 0, 'ansible_index_var': 'i'})

TASK [Determine the CMD directives] *************************************************************************************************
ok: [localhost] => (item={'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True})

TASK [Create molecule instance(s)] **************************************************************************************************
changed: [localhost] => (item={'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True})

TASK [Wait for instance(s) creation to complete] ************************************************************************************
FAILED - RETRYING: Wait for instance(s) creation to complete (300 retries left).
changed: [localhost] => (item={'started': 1, 'finished': 0, 'ansible_job_id': '298323079824.101297', 'results_file': '/home/student2/.ansible_async/298323079824.101297', 'changed': True, 'failed': False, 'item': {'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True}, 'ansible_loop_var': 'item'})

PLAY RECAP **************************************************************************************************************************
localhost                  : ok=5    changed=2    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0

INFO     Running default > prepare
WARNING  Skipping, prepare playbook not configured.
```

Hopefully that works, so you now have a test framework to work with.

So what did this do and why is it useful?

The **_create_** directive effectively spins up some infra we can use to test our role without doing such else.

If we take a look at:

```bash
cat molecule/default/molecule.yml
---
dependency:
  name: galaxy
driver:
  name: podman
platforms:
  - name: instance
    image: docker.io/pycontribs/centos:8
    pre_build_image: true
provisioner:
  name: ansible
verifier:
  name: ansible
```

This helps us to see what's being used. 

The default in the path referes to a **scenario**, and there is always *default* out-the-box. There is no need to change that for our basic testing example.

We can see that molecule is going to use the podman driver and for a target platform spin up a container instance using a centos8 image.

We can see this in action, using:

```bash
podman images
REPOSITORY                   TAG     IMAGE ID      CREATED        SIZE
docker.io/pycontribs/centos  8       0e8bfa1c168c  10 months ago  752 MB

podman ps
CONTAINER ID  IMAGE                          COMMAND               CREATED         STATUS             PORTS   NAMES
f07640ca9996  docker.io/pycontribs/centos:8  bash -c while tru...  14 minutes ago  Up 14 minutes ago          instance
```

So we have a local centos container image and it's running from the **molecule create**

We can even login to the container:

```bash
 molecule login
INFO     Running default > login
[root@instance /]#
```

Remember to logout of the container before continuing!

We can use **_destroy_** to teardown that containerised infra:

```bash
(molecule) [vagrant@sudocve apache_install]$ molecule destroy
INFO     default scenario test matrix: dependency, cleanup, destroy
INFO     Running default > dependency
WARNING  Skipping, missing the requirements file.
WARNING  Skipping, missing the requirements file.
INFO     Running default > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running default > destroy
INFO     Sanity checks: 'podman'

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item={'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True})

TASK [Wait for instance(s) deletion to complete] *******************************
FAILED - RETRYING: Wait for instance(s) deletion to complete (300 retries left).
FAILED - RETRYING: Wait for instance(s) deletion to complete (299 retries left).
changed: [localhost] => (item={'started': 1, 'finished': 0, 'ansible_job_id': '193165890648.13046', 'results_file': '/home/vagrant/.ansible_async/193165890648.13046', 'changed': True, 'failed': False, 'item': {'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True}, 'ansible_loop_var': 'item'})

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

We can see that the container is no longer there:

```bash
podman ps
CONTAINER ID  IMAGE   COMMAND  CREATED  STATUS  PORTS   NAMES
```

We're using podman for the driver here, but there are lots of other options. We can see what's installed and being used:

```bash
molecule drivers
╶──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╴
  delegated
  podman
```

*delegated* can be used with vmware for instance - checkout [this](https://github.com/pharriso/ansible_molecule_vmware) for an example

```bash
$ molecule list
INFO     Running default > list
                ╷             ╷                  ╷               ╷         ╷
  Instance Name │ Driver Name │ Provisioner Name │ Scenario Name │ Created │ Converged
╶───────────────┼─────────────┼──────────────────┼───────────────┼─────────┼───────────╴
  instance      │ podman      │ ansible          │ default       │ false   │ false
```

### Step 2 - Testing Roles

A typical dev cycle is : write some plays/roles -> **molecule converge** -> rinse and repeat...
Once you're happy with the results, a **molecule test** does a full rinse cycle
Once you're happy you can commit your code to SCM.

Right out-the-box *converge* should work as it has a template *molecule/default/converge.yml*:

```bash
---
- name: Converge
  hosts: all
  tasks:
    - name: "Include apache_install"
      include_role:
        name: "apache_install"
```

As we haven't written anything in the role to test yet, it just falls through but it shows the principle:

```bash
molecule converge
INFO     default scenario test matrix: dependency, create, prepare, converge
INFO     Running default > dependency
WARNING  Skipping, missing the requirements file.
WARNING  Skipping, missing the requirements file.
INFO     Running default > create
WARNING  Skipping, instances already created.
INFO     Running default > prepare
WARNING  Skipping, prepare playbook not configured.
INFO     Running default > converge
INFO     Sanity checks: 'podman'

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************


ok: [instance]

TASK [Include apache_install] **************************************************

PLAY RECAP *********************************************************************
instance                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

Notice the first INFO line containing:
```bash
INFO     default scenario test matrix: dependency, create, prepare, converge
```

This shows us the things that are going to happen as part of the flow. All these parts can be customised, but for now we'll just use the defauls.



### Step 3: Write The Role Tasks

Time to write the role contents!

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

### Step 4 - Test The Role

Let's run *converge* again and see what happens:

```bash
molecule converge
INFO     default scenario test matrix: dependency, create, prepare, converge
INFO     Running default > dependency
WARNING  Skipping, missing the requirements file.
WARNING  Skipping, missing the requirements file.
INFO     Running default > create
WARNING  Skipping, instances already created.
INFO     Running default > prepare
WARNING  Skipping, prepare playbook not configured.
INFO     Running default > converge
INFO     Sanity checks: 'podman'

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [instance]

TASK [Include apache_install] **************************************************

TASK [apache_install : Include other playbooks] ********************************
included: /home/vagrant/ansible-files/roles/apache_install/tasks/install_apache.yml for instance

TASK [apache_install : Install Apache] *****************************************
changed: [instance]

PLAY RECAP *********************************************************************
instance                   : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

You can see that molecule is now picking up and running our role!

(Try running it again to check that idempotency, if you want)

So now you can write and test your role by repeating the above cycle. Neat.

## Step 5: Full Test Run

Once you've finished your role, use *test* to put it through more extensive testing.

You'll notice, how the *test* flow is different from *converge*:

```bash
INFO     default scenario test matrix: dependency, lint, cleanup, destroy, syntax, create, prepare, converge, idempotence, side_effect, verify, cleanup, destroy
```

We're doing a lot more testing in and around the role now!

```bash
molecule test
INFO     default scenario test matrix: dependency, lint, cleanup, destroy, syntax, create, prepare, converge, idempotence, side_effect, verify, cleanup, destroy
INFO     Running default > dependency
WARNING  Skipping, missing the requirements file.
WARNING  Skipping, missing the requirements file.
INFO     Running default > lint
INFO     Lint is disabled.
INFO     Running default > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running default > destroy
INFO     Sanity checks: 'podman'

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item={'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True})

TASK [Wait for instance(s) deletion to complete] *******************************
FAILED - RETRYING: Wait for instance(s) deletion to complete (300 retries left).
FAILED - RETRYING: Wait for instance(s) deletion to complete (299 retries left).
changed: [localhost] => (item={'started': 1, 'finished': 0, 'ansible_job_id': '222214046033.15586', 'results_file': '/home/vagrant/.ansible_async/222214046033.15586', 'changed': True, 'failed': False, 'item': {'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True}, 'ansible_loop_var': 'item'})

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Running default > syntax

playbook: /home/vagrant/ansible-files/roles/apache_install/molecule/default/converge.yml
INFO     Running default > create

PLAY [Create] ******************************************************************

TASK [Log into a container registry] *******************************************
skipping: [localhost] => (item={'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True})

TASK [Check presence of custom Dockerfiles] ************************************
ok: [localhost] => (item={'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True})

TASK [Create Dockerfiles from image names] *************************************
skipping: [localhost] => (item={'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True})

TASK [Discover local Podman images] ********************************************
ok: [localhost] => (item={'changed': False, 'skipped': True, 'skip_reason': 'Conditional result was False', 'item': {'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True}, 'ansible_loop_var': 'item', 'i': 0, 'ansible_index_var': 'i'})

TASK [Build an Ansible compatible image] ***************************************
skipping: [localhost] => (item={'changed': False, 'skipped': True, 'skip_reason': 'Conditional result was False', 'item': {'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True}, 'ansible_loop_var': 'item', 'i': 0, 'ansible_index_var': 'i'})

TASK [Determine the CMD directives] ********************************************
ok: [localhost] => (item={'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True})

TASK [Create molecule instance(s)] *********************************************
changed: [localhost] => (item={'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True})

TASK [Wait for instance(s) creation to complete] *******************************
FAILED - RETRYING: Wait for instance(s) creation to complete (300 retries left).
changed: [localhost] => (item={'started': 1, 'finished': 0, 'ansible_job_id': '886186648632.15751', 'results_file': '/home/vagrant/.ansible_async/886186648632.15751', 'changed': True, 'failed': False, 'item': {'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True}, 'ansible_loop_var': 'item'})

PLAY RECAP *********************************************************************
localhost                  : ok=5    changed=2    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0

INFO     Running default > prepare
WARNING  Skipping, prepare playbook not configured.
INFO     Running default > converge

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [instance]

TASK [Include apache_install] **************************************************

TASK [apache_install : Include other playbooks] ********************************
included: /home/vagrant/ansible-files/roles/apache_install/tasks/install_apache.yml for instance

TASK [apache_install : Install Apache] *****************************************
changed: [instance]

PLAY RECAP *********************************************************************
instance                   : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Running default > idempotence

PLAY [Converge] ****************************************************************

TASK [Gathering Facts] *********************************************************
ok: [instance]

TASK [Include apache_install] **************************************************

TASK [apache_install : Include other playbooks] ********************************
included: /home/vagrant/ansible-files/roles/apache_install/tasks/install_apache.yml for instance

TASK [apache_install : Install Apache] *****************************************
ok: [instance]

PLAY RECAP *********************************************************************
instance                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Idempotence completed successfully.
INFO     Running default > side_effect
WARNING  Skipping, side effect playbook not configured.
INFO     Running default > verify
INFO     Running Ansible Verifier

PLAY [Verify] ******************************************************************

TASK [Example assertion] *******************************************************
ok: [instance] => {
    "changed": false,
    "msg": "All assertions passed"
}

PLAY RECAP *********************************************************************
instance                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Verifier completed successfully.
INFO     Running default > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running default > destroy

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item={'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True})

TASK [Wait for instance(s) deletion to complete] *******************************
FAILED - RETRYING: Wait for instance(s) deletion to complete (300 retries left).
FAILED - RETRYING: Wait for instance(s) deletion to complete (299 retries left).
changed: [localhost] => (item={'started': 1, 'finished': 0, 'ansible_job_id': '710465571993.16827', 'results_file': '/home/vagrant/.ansible_async/710465571993.16827', 'changed': True, 'failed': False, 'item': {'image': 'docker.io/pycontribs/centos:8', 'name': 'instance', 'pre_build_image': True}, 'ansible_loop_var': 'item'})

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory
```

## Section 4: Customising Molecule

Out-the-box molecule gives us something really useful to start with, but we can chop and change what we what to happen.

**_TBC_**

## Summary: The Finished Playbook

You've explored the basics around using molecule for tesing Ansible.

---

[Click Here to return to the Ansible Linklight - Ansible Engine Workshop](../README.md)
