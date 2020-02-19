# Exercise - Using Collections

As of Ansible 2.9, we now have [collections](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html)  built in by default.

They are the future, and there is no need to be scared of change!

This quick exercise will show you how to use them in your playbooks/roles.

If you want to learn more about developing collections then go [here](https://docs.ansible.com/ansible/devel/dev_guide/developing_collections.html)

## Just Ping It

Using the ping module we can quickly demonstrate how collections are called/used with a playbook.

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

Hold your horses! Before running it, let's first explain what we have here :)

TBA...

Now run the playbook passing in a which_ping variable (using -e extra_vars) to see what get's called:

```bash
ansible-playbook collection_example.yml -e which_ping="builtin"
ansible-playbook collection_example.yml -e which_ping="legacy"
ansible-playbook collection_example.yml -e which_ping="somethingelse"
```

---

[Click Here to return to the Ansible Linklight - Ansible Engine Workshop](../README.md)
