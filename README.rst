##################
How we use ansible
##################

This document aims to layout the conventions used in (and the problems with)
the Lost Property ansible configurations. There is an incredibly high chance
that there is much to be improved in how we use ansible. The configuration we
use is based on one started when ansible was fairly new, since then a lot has
changed in ansible and what was a good practice then is quite possibly not a
good practice now. Beyond that, we do not use ansible frequently enough to
spend much time thinking about how to improve our configuration conventions.


Playbooks
=========

All projects default to using a main playbook called ``site.yml``. The
advantage that this brings is that when moving from project to project any
member of the team can safely assume that running ``ansible-playbook
site.yml`` will provision that project.

Site.yml
^^^^^^^^

Ideally, the main ``site.yml`` playbook should do nothing but include the
other playbooks required for the given project but there are some projects
where ``site.yml`` does include top-level tasks. These should be extracted
whenever possible.

css.yml
^^^^^^^

We have some sites where the servers are initially configured by a 3rd party
who use conventions different to the Ubuntu conventions our configs are based
around such as not creating a home directory for the default user and using a
restrictive http proxy. In order to bring these servers into line the first
included playbook for the affected projects is ``css.yml``. Not all servers
for an affected project will be controlled by this 3rd party however and so
the ``css.yml`` playbook should be included conditionally as follows::

    - include: css.yml
      when: css_controlled

This relies on a ``css_controlled`` variable being set appropriately. We
default this variable to ``false`` but it should be overridden where
appropriate in the group_vars file for the affect servers.


Common.yml
^^^^^^^^^^

There are many tasks that are needed for any server we provision such as
configuring sudo access, setting up ssh keys, configuring ``bashrc`` and
these tasks are grouped into the ``common.yml`` playbook. Similarly to
groups, some predictions of what constitutes a common task have proved wrong
- one example is that ``common.yml`` calls the common role which in turn runs
a task "Install essentials". At one point all machines we provisioned ran
python webapps deployed via git and so it seemed reasonable to consider git,
python-pip and python-dev essentials. This has proved to not be the case and
so perhaps the entire "Install essentials" task should be removed.
