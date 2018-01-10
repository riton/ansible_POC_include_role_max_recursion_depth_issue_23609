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

### Using include_role

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

### Using import_role

```
$ ansible-playbook poc-import_role.yml
[...]

PLAY [localhost] ***************************************************************************************************************************

TASK [include_role] ************************************************************************************************************************
ERROR! A recursion loop was detected with the roles specified. Make sure child roles do not have dependencies on parent roles

The error appears to have been in '/home/user/ansible/ansible_POC_include_role_max_recursion_depth_issue_23609/roles/import_r31/tasks/main.yml': line 6, column 11, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

- import_role:
    name: "import_r32"
          ^ here

PLAY RECAP *********************************************************************************************************************************
localhost                  : ok=82   changed=0    unreachable=0    failed=0   
```

The playbook refuses to start and triggers something like a _parse error_. There is no recursion loop in those roles.
