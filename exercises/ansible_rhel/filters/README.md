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


---

[Click Here to return to the Ansible Linklight - Ansible Engine Workshop](../README.md)
