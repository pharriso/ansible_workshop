# Exercise - Working with Ansible Filters

Ansible comes with many [filters](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html) which allow you to transform data.

Let's go through a couple of common use case examples to showcase their power.


## Parsing JSON Input

Perhaps we've got another application which spits out some valid JSON formatted data, which we want to make use of inside some playbooks for automation.

Let's create some dummy JSON data that we'll use in this example. Run this:

```bash
cat >sample.json << EOF
{
  "apple_pie": "ice_cream",
  "rhubarb": "custard",
  "salt": "pepper",
  "_NULL_": ""
}
EOF
```

Now we have some data to work with, and it should look like this:

```bash
{
  "apple_pie": "ice_cream",
  "rhubarb": "custard",
  "salt": "pepper",
  "_NULL_": ""
}
```

Let's create a playbook to use this:

```bash
cat >json_sample.yml <<EOF
---
- hosts: localhost
  connection: local

  tasks:
    - shell: cat ./sample.json
      register: result

    - set_fact:
        myvars: "{{ result.stdout | from_json }}"

    - debug:
        msg: "{{ item.key }} loves a bit of {{ item.value }}"
      loop: "{{ myvars | dict2items }}"
      when: item.value != ""
EOF
```

Before running this, let's explain what's going on here.

- Using the shell module we read in the JSON data and store it in the 'result' variable
- We then create a new 'myvars' fact, getting the data from the shell stdout, and use the from_json filter for parsing
- The debug module shows us the data, using a loop with the dict2items filter, which breaks it down into key/value pairs
- We only care about keys with a value. This is the when clause.

Run the playbook:

```bash
ansible-playbook json_sample.yml
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not
match 'all'

PLAY [localhost] **************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************
ok: [localhost]

TASK [shell] ******************************************************************************************************
changed: [localhost]

TASK [set_fact] ***************************************************************************************************
ok: [localhost]

TASK [debug] ******************************************************************************************************
ok: [localhost] => (item={'key': 'apple_pie', 'value': 'ice_cream'}) => {
    "msg": "apple_pie loves a bit of ice_cream"
}
ok: [localhost] => (item={'key': 'rhubarb', 'value': 'custard'}) => {
    "msg": "rhubarb loves a bit of custard"
}
ok: [localhost] => (item={'key': 'salt', 'value': 'pepper'}) => {
    "msg": "salt loves a bit of pepper"
}
skipping: [localhost] => (item={'key': '_NULL_', 'value': ''})

PLAY RECAP ********************************************************************************************************
localhost                  : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

You'll see that we have successfully parsed the JSON data, and ignored the _NULL_ key as it has no data associated with it.

## Setting Defaults and Sanity Checks

We might often want to use some shell environment variables within our plays. These are often required these days for many cloud providers and mainstream applications. 

Let's look one up and set a default if it's not set. 

Let's create the first part of the playbook:

```bash
cat >defaults_sample.yml <<EOF
---
- hosts: localhost
  connection: local

  vars:
    password: redhat

  tasks:

    - set_fact:
        myuser: "{{ lookup('env', 'MY_USER') | default('admin', true) }}"

    - debug:
        msg: myuser is now set to {{ myuser | upper }}

EOF
```

Before running it, check that you don't already have $MY_USER set in your shell:

```bash
echo $MY_USER
```

It should come back blank. You can now run the playbook:

```bash
ansible-playbook defaults_sample.yml
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not
match 'all'

PLAY [localhost] **************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************
ok: [localhost]

TASK [set_fact] ***************************************************************************************************
ok: [localhost]

TASK [debug] ******************************************************************************************************
ok: [localhost] => {
    "msg": "myuser is now set to ADMIN"
}

PLAY RECAP ********************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

You can see that we've set the value to 'admin' as it was unset. In the debug statement, we've used the upper filter to highlight to the user what it's been set to.

Feel free to set MY_USER to something else, and re-run the playbook, but leave it set to 'root' on the last run:

```bash
export MY_USER=bob
export MY_USER=root
```

We'll now move onto our sanity checks, to stop dumb ass things being used or not set correctly!

Add this to the existing playbook, using:

```bash
cat >>defaults_sample.yml <<EOF
    - name: User check validation
      assert:
        that:
          - myuser != "root"
        fail_msg:
          - "You appear to be root"
          - "I'm not going to allow that!"

    - name: Check password complexity
      assert:
        that:
          - password | length > 7
          - password | regex_search('[A-Z]')
          - password | regex_search('[a-z]')
          - password | regex_search('[0-9]')
        fail_msg: "password does not need password complexity requirements (8+ Chars, Lower Case, Upper Case, Number)"
EOF
```

We're now checking that MY_USER isn't set to root and that the password var set is sensible (perhaps to met minimum security standards)

As we left $MY_USER set to 'root', if you run the playbook it should now fail:

```bash
ansible-playbook defaults_sample.yml
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [localhost] ***************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************
ok: [localhost]

TASK [set_fact] ****************************************************************************************************************
ok: [localhost]

TASK [debug] *******************************************************************************************************************
ok: [localhost] => {
    "msg": "myuser is now set to ROOT"
}

TASK [User check validation] ***************************************************************************************************
fatal: [localhost]: FAILED! => {
    "assertion": "myuser != \"root\"",
    "changed": false,
    "evaluated_to": false,
    "msg": [
        "You appear to be root",
        "I'm not going to allow that!"
    ]
}

PLAY RECAP *********************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
```

Bingo! We've caught which could be a silly and far reaching issue. The [assert](https://docs.ansible.com/ansible/latest/modules/assert_module.html) module is a nice and simple way to make some simple assumptions and checks. 

Set $MY_USER back to something other than root and re-run the playbook:

```bash
export MY_USER=fred
ansible-playbook defaults_sample.yml
```

---

[Click Here to return to the Ansible Linklight - Ansible Engine Workshop](../README.md)
