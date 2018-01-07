# POC for ansible bug number 23609

This is a Proof of Concept for the ansible bug `include/include_role: maximum recursion depth exceeded` ([direct link to the bug](https://github.com/ansible/ansible/issues/23609))

## What this POC does

It simulates a useless set of role that are _chained_ to each other

* role `r1` prints a debug message, includes role `r2` using `include_role`, prints another debug message and leave
* role `r2` prints a debug message, includes role `r3` using `include_role`, prints another debug message and leave
* role `r3` prints a debug message, includes role `r4` using `include_role`, prints another debug message and leave
* ... and so on
* the last role (currently role `r41`) only prints a debug message and leave

## Environment

```
$ [ansible-include_role_poc](master)$ ansible --version
ansible 2.5.0
config file = None
configured module search path = [u'/home/user/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
ansible python module location = /tmp/ansible/ansible-devel/lib/ansible
executable location = /tmp/ansible/ansible-devel/bin/ansible
python version = 2.7.12 (default, Nov 19 2016, 06:48:10) [GCC 5.4.0 20160609]
```

## How to trigger the bug

```
$ ansible-playbook poc.yml
[...]
TASK [r27 : debug] *********************************************************************************************************************************************************************
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of r27"
}

TASK [r27 : include_role] **************************************************************************************************************************************************************
ERROR! Unexpected Exception, this is probably a bug: maximum recursion depth exceeded while calling a Python object
to see the full traceback, use -vvv
```
