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
$ ansible-playbook -e max_nested_include_roles=100 include-poc.yml
[...]
```

Everything works fast and as expected.

### Using import_role

**Create nested import_roles**

This extra step is necessary, because some ansible bug sometimes trigger compilation or parsing error and avoid any real testing of 'max nested import_roles'

```
$ ansible-playbook -e max_nested_import_roles=10 import-create_roles.yml
[...]
```

**Call the nested import_roles**

```
$ ansible-playbook import-poc.yml
[...]
ERROR! A recursion loop was detected with the roles specified. Make sure child roles do not have dependencies on parent roles

The error appears to have been in '/home/user/ansible_POC_include_role_max_recursion_depth_issue_23609/import-poc.yml': line 8, column 15, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

    - import_role:
        name: 'import_r1'
              ^ here
```

## History

**2018-02-21**

* After [this ansible bug comment](https://github.com/ansible/ansible/issues/17852#issuecomment-331115849), update the test/poc suite.
  The `include_role` seems now fully functionnal and pretty fast, thanks to the dev' work !

* The `import_role` statement seems still hard to use
