# Exercise - Using Collections

As of Ansible 2.9, we now have [collections](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html)  built in by default.

There are different ways to interact with Ansible Collections:

* Install into your runtime or virtual environment
* Provide as part of your SCM tree, or,
* Using a requirements file

Regardless of the method chosen, first you need to find, identity and obtain the Ansible Collections you want to use.

Ansible when installed, will come with some **core** collection content. This quick exercise will show you how to use that in your playbooks/roles, as a quick example.

If you want to learn more about developing collections then go [here](https://docs.ansible.com/ansible/devel/dev_guide/developing_collections.html)

## Just Ping It

Using the simple **ping** module we can quickly demonstrate how collections are used within a playbook.

Create this simple playbook, by running:

```bash
cat >collection_example.yml <<EOF
---
- hosts: localhost
  connection: local

  tasks:

  - ansible.builtin.ping:
    when: which_ping == "builtin"

  - ansible.legacy.ping:
    when: which_ping == "legacy"

  - ping: # still works = ansible.legacy.ping
    when:
      - which_ping != "builtin"
      - which_ping != "legacy"
EOF
```

## Hold your horses!

Before running it, let's first explain what we have here :)

Collections 'live' in a namespace, have a name, and then contain content, like modules within them. So it's like this:
my_namespace.my_collection.my_module

In our example, 'ansible' is the namespace, the collections are 'builtin' and 'legacy' and the module is 'ping'

Other examples might be something like:
f5.bigip.provisioning_role (made up), or,
azure.azcollection
google.cloud (real ones!)

Now run the playbook passing in a **which_ping** variable (using -e extra_vars) to see what gets called:

```bash
ansible-playbook collection_example.yml -e which_ping="builtin"
ansible-playbook collection_example.yml -e which_ping="legacy"
ansible-playbook collection_example.yml -e which_ping="somethingelse"
```

You'll see the appropriate ping module gets called when the condition is true, and that all work.

As with everything Ansible, order is important. The first found will get used, as we run sequentially through the playbook.


## Too Verbose?

We can make life easier for ourselves by defining collections paths with our playbook:

```bash
cat >collection_terse.yml <<EOF
---
- hosts: localhost
  connection: local

  collections:
   - ansible.builtin
   - myns.mycollection
   - otherns.othercollection

  tasks:

  - ping: # first found in the list of collections
  #- otherns.othercollection.mymodule: # fully-qualified is also fine
EOF
```

Run it of you like :)

```bash
ansible-playbook collection_terse.yml
```

Using paths, should cut down on typing!

---

[Click Here to return to the Ansible Linklight - Ansible Engine Workshop](../README.md)
