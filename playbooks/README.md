workshop_cockpit
========
This playbook can setup a cockpit server on one of the student workstations. This is useful to allow users access to the cockpit web terminal for ssh access to their lab environment. Pick a free lab environment as this playbook will stop Tower and configure cockpit on port 443.

To run the playbook (note you need the comma after server name):

ansible-playbook -i student.domain, workshop_cockpit.yml -u student -k

e.g.

ansible-playbook -i student1.c3ff.rhdemo.io, workshop_cockpit.yml -u student1 -k

When prompted enter the password. Then log into cockpit by browsing to:

https://student1.c3ff.rhdemo.io and log in as student1 with password.

Multiple users should be able to use the same cockpit server.


