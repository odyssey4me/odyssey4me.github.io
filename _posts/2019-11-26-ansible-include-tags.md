---
layout: post
title: "Ansible 2.7+ tag inheritance with includes"
date: 2019-11-26 14:31:11
tags: [ansible, optimizing]
---
The [Ansible 2.8 documentation about tags] includes a section about tag
inheritance, explaining how it is that tags are inherited when using
``include_tasks`` vs ``import_tasks``. A little more explanation is also
given in [John Wadleigh's post].

Sometimes you may want to make use of a tag in a set of included tasks or
an included role and, for whatever reason, you want to avoid importing
the tasks/role. This is possible - see the illustration below.

```yml
---
#
# playbook.yml
#
- hosts: localhost
  connection: local
  gather_facts: no
  tasks:
    - include_tasks: tasks.yml
      tags: always
```

```yml
#
# tasks.yml
#
---
- debug:
    msg: "tag1"
  tags: tag1

- debug:
    msg: "tag2"
  tags: tag2
```

```
#
# output of: ansible-playbook playbook.yml --tags tag1
#
PLAY [localhost] **************************************************************

TASK [include_tasks] **********************************************************
included: /tmp/examples/tasks.yml for localhost

TASK [debug] ******************************************************************
ok: [localhost] => {}

MSG:

tag1

PLAY RECAP ********************************************************************
localhost : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

What we can see happening here is that when the pre-processing occurrs, the
``always`` tag ensures that the ``include_tasks`` task is included at run-time,
which then provides access to ``tag1``. Without the ``always`` tag, none of the
included tasks will be processed at all.

A side-effect of this method is that using the ``--list-tags`` argument will
not yield any of the tags from included tasks or roles.

[Ansible 2.8 documentation about tags]: https://docs.ansible.com/ansible/2.8/user_guide/playbooks_tags.html
[John Wadleigh's post]: https://www.ansiblejunky.com/blog/ansible-101-include-vs-import/
