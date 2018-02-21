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

```
$ ansible-playbook poc-import_role.yml # to create the required roles
[...]
```

and then to really use the `import_role` statement

```
$ ansible-playbook import_role.yml
[...]
PLAY [localhost] ***************************************************************************************************************************

TASK [import_r1 : debug] *******************************************************************************************************************
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of import_r1"
}

TASK [import_r2 : debug] *******************************************************************************************************************
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of import_r2"
}

TASK [import_r3 : debug] *******************************************************************************************************************
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of import_r3"
}

TASK [import_r4 : debug] *******************************************************************************************************************
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of import_r4"
}

TASK [import_r5 : debug] *******************************************************************************************************************
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of import_r5"
}

TASK [import_r6 : debug] *******************************************************************************************************************
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of import_r6"
}

TASK [import_r7 : debug] *******************************************************************************************************************
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of import_r7"
}
ERROR! Unexpected Exception, this is probably a bug: maximum recursion depth exceeded
to see the full traceback, use -vvv
```

The playbook fails even sooner than with `include_role`.

Here is the full traceback of the bug

```
ansible-playbook 2.5.0 (devel f2037bb629) last updated 2018/01/10 08:40:47 (GMT +200)
  config file = None
  configured module search path = [u'/home/user/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /home/user/ansible/lib/ansible
  executable location = /home/user/ansible/bin/ansible-playbook
  python version = 2.7.12 (default, Nov 19 2016, 06:48:10) [GCC 5.4.0 20160609]
No config file found; using defaults
setting up inventory plugins
 [WARNING]: Unable to parse /etc/ansible/hosts as an inventory source
 [WARNING]: No inventory was parsed, only implicit localhost is available
 [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
Loading callback plugin default of type stdout, v2.0 from /home/user/ansible/lib/ansible/plugins/callback/default.pyc

PLAYBOOK: import_role.yml ******************************************************************************************************************
1 plays in import_role.yml

PLAY [localhost] ***************************************************************************************************************************
META: ran handlers

TASK [import_r1 : debug] *******************************************************************************************************************
task path: /home/user/ansible/ansible_POC_include_role_max_recursion_depth_issue_23609/roles/import_r1/tasks/main.yml:2
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of import_r1"
}

TASK [import_r2 : debug] *******************************************************************************************************************
task path: /home/user/ansible/ansible_POC_include_role_max_recursion_depth_issue_23609/roles/import_r2/tasks/main.yml:2
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of import_r2"
}

TASK [import_r3 : debug] *******************************************************************************************************************
task path: /home/user/ansible/ansible_POC_include_role_max_recursion_depth_issue_23609/roles/import_r3/tasks/main.yml:2
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of import_r3"
}

TASK [import_r4 : debug] *******************************************************************************************************************
task path: /home/user/ansible/ansible_POC_include_role_max_recursion_depth_issue_23609/roles/import_r4/tasks/main.yml:2
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of import_r4"
}

TASK [import_r5 : debug] *******************************************************************************************************************
task path: /home/user/ansible/ansible_POC_include_role_max_recursion_depth_issue_23609/roles/import_r5/tasks/main.yml:2
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of import_r5"
}

TASK [import_r6 : debug] *******************************************************************************************************************
task path: /home/user/ansible/ansible_POC_include_role_max_recursion_depth_issue_23609/roles/import_r6/tasks/main.yml:2
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of import_r6"
}

TASK [import_r7 : debug] *******************************************************************************************************************
task path: /home/user/ansible/ansible_POC_include_role_max_recursion_depth_issue_23609/roles/import_r7/tasks/main.yml:2
ok: [localhost] => {
    "attempts": 1, 
    "msg": "Start of import_r7"
}
ERROR! Unexpected Exception, this is probably a bug: maximum recursion depth exceeded
the full traceback was:

Traceback (most recent call last):
  File "/home/user/ansible/bin/ansible-playbook", line 118, in <module>
    exit_code = cli.run()
  File "/home/user/ansible/lib/ansible/cli/playbook.py", line 122, in run
    results = pbex.run()
  File "/home/user/ansible/lib/ansible/executor/playbook_executor.py", line 159, in run
    result = self._tqm.run(play=play)
  File "/home/user/ansible/lib/ansible/executor/task_queue_manager.py", line 290, in run
    play_return = strategy.run(iterator, play_context)
  File "/home/user/ansible/lib/ansible/plugins/strategy/linear.py", line 248, in run
    task_vars = self._variable_manager.get_vars(play=iterator._play, host=host, task=task)
  File "/home/user/ansible/lib/ansible/vars/manager.py", line 404, in get_vars
    all_vars = combine_vars(all_vars, task.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 69, in get_vars
    all_vars = super(TaskInclude, self).get_vars()
  File "/home/user/ansible/lib/ansible/playbook/task.py", line 327, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/block.py", line 77, in get_vars
    all_vars.update(self._parent.get_vars())
  File "/home/user/ansible/lib/ansible/playbook/task_include.py", line 68, in get_vars
    if self.action != 'include':
RuntimeError: maximum recursion depth exceeded
```
