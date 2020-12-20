# ansible

ansible configuration for workstations and servers

The goal here is to have a top-level playbook which we can use with ansible-pull from workstations, and using ansible-playbook from any node with ansible installed and the correct ssh keys.

This allows for new workstations to be configured easily with ansible-pull, and
becoming a control node is simply a matter of having the correct ssh keys.


