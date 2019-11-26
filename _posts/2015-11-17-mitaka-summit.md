---
layout: post
title:  "OpenStack-Ansible Mitaka Summit Summary"
date:   2015-11-17 11:35:00
tags: [openstack, ansible, mitaka, summit]
---
The OpenStack Design Summit provides an opportunity for OpenStack
collaborators to meet face-to-face, discuss lessons learned in the last cycle
and figure out the plans for the next cycle.

As a new PTL for a deployment project in the OpenStack Big Tent it provided me
an opportunity to attend design sessions/discussions in the summit which I
felt would benefit the project as a whole and to look for opportunities where
the project can add value to Operators, to Developers and to the whole
OpenStack ecosystem.

## Upgrading OpenStack

The Operator sessions I managed to attend were the
[Major Liberty Issues](https://etherpad.openstack.org/p/TYO-ops-major-liberty-issues)
and
[Upgrades](https://etherpad.openstack.org/p/TYO-ops-upgrades) sessions. Both
these sessions were of keen interest to me as upgrading OpenStack was a
subject matter which  we, as the
[OpenStack-Ansible](https://github.com/openstack/openstack-ansible)
project, had determined was a difficulty area for our downstream consumers and
for the OpenStack community as a whole.

### The Problem

The problem has a number of facets which make it difficult:

1. Upgrades require configuration review to spot any deprecations.

    Current configurations need to be reviewed to determine whether anything
    being used has been deprecated upstream. If so then it needs to be
    identified whether there is a replacement configuration or whether it has
    been entirely removed. The release notes do help, and they have gotten
    better, but they are not always complete.

    While this action often ends up being considered a luxury (as deprecation
    cycles are often quite long), not doing this proactively can result in you
    being caught by surprise several upgrades later when the reference material
    needed is lost in the haystack of more up to date information. Also, the
    best opportunity to improve the reference information is usually at the
    point of initial deprecation - as that is when the developers have the
    information you need fresh in their minds.

2. Upgrades require API down time.

    For some this is more of a problem than for others. Reports are that Nova
    online upgrades work well, but require careful orchestration of
    configuration changes if you care about staying online and not losing any
    API transactions that enter the environment during the course of the
    upgrades. Other projects in the ecosystem are not nearly as mature in
    terms of doing online upgrades and therefore require API down time.

3. Database offline migration times are unpredictable.

    This essentially means that you have no idea how long your API down time
    will be, making your minimal change window time hard to determine. The
    only solution recommended by Operators to improve the success of this
    step, and to improve time estimates, is to ensure that database migrations
    are tested using a backup of the live database before the change window as
    part of the change preparation.

4. Component version compatibility is important to understand up front.

    It appears to be fairly common for Operators using Horizon to be using a
    later version of Horizon than is used for the rest of the stack. Many of
    the larger Operators even run the latest version of Horizon from the head
    of the master branch. It is well known that this is an option, but whether
    you can run mixed versions of other components is not very well known. The
    general consensus is to test what you are hoping to do in a staging
    environment before executing it in production.

5. Upgrading is more than just about upgrading OpenStack.

    As was
    [expressed recently on the Operators mailing list](http://lists.openstack.org/pipermail/openstack-operators/2015-November/008705.html),
    upgrading OpenStack is only one part of the upgrade process. It also
    involves certification of all integrations, hardware and other bits that
    interact with OpenStack. It will also often involve updating training
    materials, Operations run books, automation scripts, knowledgebase
    materials, marketing information and many other bits that relate to the
    environment.

    If there is any data plane down time then there is also a possibility that
    co-ordinating that down time with the stakeholders will be necessary, which
    introduces the additional complexity of having to schedule in-between
    change black-out periods or having to jump through additional hoops which
    the stakeholders require as part of their policies and procedures.

### How can we help?

As OpenStack-Ansible is a deployment project which deploys from source, we are
well positioned to play a role in the broader community to help improve the
Operator experience with regards to upgrades:

1. We can play a part in validating and improving release documentation.

    Our community is able to test the next major version of OpenStack as it
    develops. This means that the community can play a part in improving
    OpenStack
    [installation](http://docs.openstack.org/liberty/install-guide-ubuntu/),
    [security](http://docs.openstack.org/security-guide/),
    [architecture](http://docs.openstack.org/arch-design/content/) and
    [developer](http://docs.openstack.org/developer/)
    documentation for current and future releases.

2. We can play a part in improving OpenStack code quality before it merges.

    As reviews are submitted in the OpenStack projects, the OpenStack-Ansible
    community is in a position to test those reviews using the
    [All-in-One (AIO)](http://docs.openstack.org/developer/openstack-ansible/developer-docs/quickstart-aio.html)
    or in multinode lab test environments and to provide feedback to the
    developers well before the code is merged. This allows the Operator
    community to play a larger part in proactive bug prevention!

    [Miguel Grinberg](http://blog.miguelgrinberg.com/) did
    [a presentation](https://www.openstack.org/summit/tokyo-2015/videos/presentation/life-without-devstack-upstream-development-with-osad)
    at the Tokyo summit about his experience using OpenStack-Ansible instead
    of DevStack for OpenStack development. There is also
    [a blog post](https://developer.rackspace.com/blog/life-without-devstack-openstack-development-with-osa/)
    on the same subject.

3. We can work out how best to orchestrate upgrades and codify the methods.

    While I am a firm believer that it is not possible to implement a
    one-size-fits-all upgrade process, I do believe that there is value in
    taking the time to work out how to execute an upgrade from one major
    version to the next with as little down time as possible, then to express
    that process in Ansible. Ansible is relatively easy to read, so it not
    only adds value to OpenStack-Ansible, but also adds value to the broader
    OpenStack community which can learn from what we put together.

### OpenStack-Ansible Upgrade Framework

We held an open workshop to further discuss the difficulties with regards to
upgrades and to try and determine whether we could identify ways in which we
could providing tooling in the project which could improve the upgrade
experience for Operators.

[The discussion](https://etherpad.openstack.org/p/openstack-ansible-mitaka-upgrades)
got a little stuck on some of the complexities and experiences and we did not
have enough time to come to any specific conclusions. However this input did
inform a further discussion in the
[Ansible Collaboration Day](https://etherpad.openstack.org/p/ansible-collabday-mitaka)
where [Blue Box](https://www.blueboxcloud.com/) OpenStack Release Lead
[Jesse Keating](https://www.linkedin.com/in/jkeating) agreed to work with
OpenStack-Ansible to develop an Ansible module to help manage database
migrations. This work will learn from the
[Neutron Database Migration module](https://github.com/openstack/openstack-ansible/blob/master/playbooks/roles/os_neutron/library/neutron_migrations_facts)
which currently only handles Neutron database migrations (both online and
offline). The end goal will be to have a module that handles database
migrations for all projects which is available outside of OpenStack-Ansible,
possibly as an Ansible extra module.

Beyond that work we hope to implement a generalised framework in the Mitaka
development cycle in order to cater for major upgrades of OpenStack,
supporting services (MariaDB, RabbitMQ, etc) and OpenStack-Ansible.

## Image-based Deployment

We held an open workshop to discuss the value and plausibility of developing
an image-based deployment mechanism within OpenStack-Ansible.

[The discussion](https://etherpad.openstack.org/p/openstack-ansible-mitaka-image-based-deployment)
had some passionate points of view, not all of which agreed. There were those
who were passionate about the implementation of microservices containers and
using image-based deployment. There were others who were passionate about
image-based deployment being a problem at scale due to network saturation.
There were still others who raised the important point that implementing
the containers through images does not cover the whole picture as it leaves
out the hosts.

No real conclusions were reached in the workshop. It does seem that we should
discuss this again another time, but perhaps it should be done after the
downstream consumers of OpenStack-Ansible have had an opportunity to make use
of the shippable venvs which were introduced in the Liberty release. The
combination of using shippable venvs and a managed apt repository do solve
the problem of repeatable deployments quite well already.

## Community Day

The OpenStack-Ansible community day was held on Friday and provided an
opportunity for us to coalesce our summit experience into
[an etherpad](https://etherpad.openstack.org/p/openstack-ansible-mitaka-meetup)
and just generally hang out, discussing anything that came to mind.

I think that while we were already quite frazzled by then, it was great to
share experiences and have open discussions.

## Other Mitaka Development Cycle Work

As is evident in the list of
[OpenStack-Ansible Mitaka Specifications](http://specs.openstack.org/openstack/openstack-ansible-specs/#mitaka-specifications)
there is also a lot of other work which we hope to achieve in the Mitaka
development cycle.

Highlights include:

1. [Gate Split](http://specs.openstack.org/openstack/openstack-ansible-specs/specs/mitaka/gate-split.html)

    The current integration gate check relies on an All-In-One (AIO) build
    which is running low on resources and does not adequately test all code
    paths that matter for the primary use-cases of the project. This work
    aims to resolve this.

2. [Independent Role Repositories](http://specs.openstack.org/openstack/openstack-ansible-specs/specs/mitaka/independent-role-repositories.html)

    In order to improve the ability to independently consume the roles
    produced by OpenStack-Ansible in different reference use-case deployments
    and allow independent development of each role by different projects, the
    existing roles are being split into their own repositories. The roles will
    also be registered in [Ansible Galaxy](https://galaxy.ansible.com/) for
    the broader community to consume.

## Call For Contributors: Multi-OS Enablement

There has also been interest in implementing OpenStack-Ansible on platforms
other than Ubuntu. eg: CentOS, Fedora, Gentoo

While there has been interest, no-one has stepped forward to own the work so
if you are interested in seeing this become a reality then please engage with
us on the [openstack-dev](http://lists.openstack.org/pipermail/openstack-dev/)
mailing list with the subject line [openstack-ansible].
