[![PyPI version](https://img.shields.io/pypi/v/ansible.svg)](https://pypi.python.org/pypi/ansible)
[![PyPI downloads](https://img.shields.io/pypi/dm/ansible.svg)](https://pypi.python.org/pypi/ansible)
[![Build Status](https://api.shippable.com/projects/573f79d02a8192902e20e34b/badge?branch=stable-2.1)](https://app.shippable.com/projects/573f79d02a8192902e20e34b)

Changing the SSH port with Ansible?
===================================

This fork of https://github.com/ansible/ansible provides a patch to the "stable-1.9" and "stable-2.0.0.1" (also fits the "devel" branch, circa 2016-01-28) to test a non-standard ssh port specification and revert to the default ssh port (normally 22) if the non-standard port is not open. Essentially, Ansible uses the following logic to employ ssh:

<pre><code>
      if self.port is not None:
         ssh -p {{ self.port }} ...
      else:
         ssh ...
</code></pre>

This patch changes the logic to:

<pre><code>
      if self.port is not None and self.port is OPEN:
         ssh -p {{ self.port }} ...
      else:
         ssh ...
</code></pre>

Currently the recommended way to handle ssh port changes with an un-patched Ansible is to use wait_for/when tasks in a playbook as follows:

<pre><code>
---
# file: demo.yaml
# ansible_ssh_port is used in preference to ansible_port because it works across Ansible v1 and v2.

- hosts: all
  gather_facts: false
  vars:
    ansible_ssh_port: 2222

  pre_tasks:
    - name: Test connection to port 2222
      local_action:
        module: wait_for
        port: "{{ ansible_ssh_port }}"
        timeout: 5
      register: test_2222
      ignore_errors: true

    - name: Set ansible_ssh_port to 22 if cannot connect to 2222
      set_fact:
        ansible_ssh_port: 22
      when: test_2222.elapsed >= 5

    - setup:

  roles:
    - set_non-std_port
</code></pre>

In a multi-host environment, this method is unreliable because it tests a single host but applies the result to all hosts. In contrast, the patch tests the port on each host as the connection is required, applies the result only to the tested host and requires no special specification in the playbook:

<pre><code>
---
# file: demo.yaml
# The non-standard ssh ports are specified in the host inventory and could be different for
# each host; see "sample_ssh_port_setting" directory on the "stable-1.9","stable-2.0.0.1" and
# "stable-2.1" branches.

- hosts: all
  roles:
    - set_non-std_port
</code></pre>

Installation of the patch
=========================

Two methods of installation are available:

   1. If you have Ansible installed via the distro's packet manager, the following procedure is recommended:
      * Locate and switch to the connection plugin directory in your Ansible installation. For Ansible version 1.9 on Centos 7.2, this directory is located at "/usr/lib/python2.7/site-packages/ansible/runner/connection_plugins/"
      * Retrieve the appropriate patch into this directory using wget or curl. For Ansible version 1, the patch is located at "https://raw.githubusercontent.com/crlb/ansible/stable-1.9/lib/ansible/runner/connection_plugins/ssh-revert_to_default_port.patch". For version 2, the location is "https://raw.githubusercontent.com/crlb/ansible/stable-2.0.0.1/lib/ansible/plugins/connection/ssh-revert_to_default_port.patch".
      * Apply the patch with "patch < ssh-revert_to_default_port.patch".
      * Recompile ssh.py with "python -m compileall .".
      * It is also recommended that you exclude Ansible from automatic updates to avoid the patch being inadvertently removed.
   1. If you do not have Ansible installed and wish to install it from the git hub repository, use the following procedure:
      * Clone the Ansible fork with "git clone git@github.com:crlb/ansible.git".
      * Switch to the cloned repository (ie. "cd ansible") and  choose the version to install by using the the "git checkout {{branch}}" command. For version 1, the patch is supplied in the "stable-1.9" branch and for version 2, in the "stable-2.0.0.1" branch (the patches may fit other releases, see above).
      * Install the Ansible core modules with "git submodule update --init --recursive".
      * Install the patched ansible with "python setup.py install".

And now, over to ...

Changing the SSH port with Ansible?
===================================

This fork of https://github.com/ansible/ansible provides a patch to the "stable-1.9" and "stable-2.0.0.1" (also fits the "devel" branch, circa 2016-01-28) to test a non-standard ssh port specification and revert to the default ssh port (normally 22) if the non-standard port is not open. Essentially, Ansible uses the following logic to employ ssh:

<pre><code>
      if self.port is not None:
         ssh -p {{ self.port }} ...
      else:
         ssh ...
</code></pre>

This patch changes the logic to:

<pre><code>
      if self.port is not None and self.port is OPEN:
         ssh -p {{ self.port }} ...
      else:
         ssh ...
</code></pre>

Currently the recommended way to handle ssh port changes with an un-patched Ansible is to use wait_for/when tasks in a playbook as follows:

<pre><code>
---
# file: demo.yaml
# ansible_ssh_port is used in preference to ansible_port because it works across Ansible v1 and v2.

- hosts: all
  gather_facts: false
  vars:
    ansible_ssh_port: 2222

  pre_tasks:
    - name: Test connection to port 2222
      local_action:
        module: wait_for
        port: "{{ ansible_ssh_port }}"
        timeout: 5
      register: test_2222
      ignore_errors: true

    - name: Set ansible_ssh_port to 22 if cannot connect to 2222
      set_fact:
        ansible_ssh_port: 22
      when: test_2222.elapsed >= 5

    - setup:

  roles:
    - set_non-std_port
</code></pre>

In a multi-host environment, this method is unreliable because it tests a single host but applies the result to all hosts. In contrast, the patch tests the port on each host as the connection is required, applies the result only to the tested host and requires no special specification in the playbook:

<pre><code>
---
# file: demo.yaml
# The non-standard ssh ports are specified in the host inventory and could be different for
# each host; see "sample_ssh_port_setting" directory on the "stable-1.9","stable-2.0.0.1" and
# "stable-2.1" branches.

- hosts: all
  roles:
    - set_non-std_port
</code></pre>

Installation of the patch
=========================

Two methods of installation are available:

   1. If you have Ansible installed via the distro's packet manager, the following procedure is recommended:
      * Locate and switch to the connection plugin directory in your Ansible installation. For Ansible version 1.9 on Centos 7.2, this directory is located at "/usr/lib/python2.7/site-packages/ansible/runner/connection_plugins/"
      * Retrieve the appropriate patch into this directory using wget or curl. For Ansible version 1, the patch is located at "https://raw.githubusercontent.com/crlb/ansible/stable-1.9/lib/ansible/runner/connection_plugins/ssh-revert_to_default_port.patch". For version 2, the location is "https://raw.githubusercontent.com/crlb/ansible/stable-2.0.0.1/lib/ansible/plugins/connection/ssh-revert_to_default_port.patch".
      * Apply the patch with "patch < ssh-revert_to_default_port.patch".
      * Recompile ssh.py with "python -m compileall .".
      * It is also recommended that you exclude Ansible from automatic updates to avoid the patch being inadvertently removed.
   1. If you do not have Ansible installed and wish to install it from the git hub repository, use the following procedure:
      * Clone the Ansible fork with "git clone git@github.com:crlb/ansible.git".
      * Switch to the cloned repository (ie. "cd ansible") and  choose the version to install by using the the "git checkout {{branch}}" command. For version 1, the patch is supplied in the "stable-1.9" branch and for version 2, in the "stable-2.0.0.1" branch (the patches may fit other releases, see above).
      * Install the Ansible core modules with "git submodule update --init --recursive".
      * Install the patched ansible with "python setup.py install".

And now, over to ...

Ansible
=======

Ansible is a radically simple IT automation system.  It handles configuration-management, application deployment, cloud provisioning, ad-hoc task-execution, and multinode orchestration - including trivializing things like zero downtime rolling updates with load balancers.

Read the documentation and more at http://ansible.com/

Many users run straight from the development branch (it's generally fine to do so), but you might also wish to consume a release.  

You can find instructions [here](http://docs.ansible.com/intro_getting_started.html) for a variety of platforms.  If you decide to go with the development branch, be sure to run `git submodule update --init --recursive` after doing a checkout. 

If you want to download a tarball of a release, go to [releases.ansible.com](http://releases.ansible.com/ansible), though most users use `yum` (using the EPEL instructions linked above), `apt` (using the PPA instructions linked above), or `pip install ansible`.

Design Principles
=================

   * Have a dead simple setup process and a minimal learning curve
   * Manage machines very quickly and in parallel
   * Avoid custom-agents and additional open ports, be agentless by leveraging the existing SSH daemon
   * Describe infrastructure in a language that is both machine and human friendly
   * Focus on security and easy auditability/review/rewriting of content
   * Manage new remote machines instantly, without bootstrapping any software
   * Allow module development in any dynamic language, not just Python
   * Be usable as non-root
   * Be the easiest IT automation system to use, ever.
  
Get Involved
============

   * Read [Community Information](http://docs.ansible.com/community.html) for all kinds of ways to contribute to and interact with the project, including mailing list information and how to submit bug reports and code to Ansible.  
   * All code submissions are done through pull requests.  Take care to make sure no merge commits are in the submission, and use `git rebase` vs `git merge` for this reason.  If submitting a large code change (other than modules), it's probably a good idea to join ansible-devel and talk about what you would like to do or add first and to avoid duplicate efforts.  This not only helps everyone know what's going on, it also helps save time and effort if we decide some changes are needed.
   * Users list: [ansible-project](http://groups.google.com/group/ansible-project)
   * Development list: [ansible-devel](http://groups.google.com/group/ansible-devel)
   * Announcement list: [ansible-announce](http://groups.google.com/group/ansible-announce) - read only
   * irc.freenode.net: #ansible

Branch Info
===========

   * Releases are named after Led Zeppelin songs. (Releases prior to 2.0 were named after Van Halen songs.)
   * The devel branch corresponds to the release actively under development.
   * As of 1.8, modules are kept in different repos, you'll want to follow [core](https://github.com/ansible/ansible-modules-core) and [extras](https://github.com/ansible/ansible-modules-extras)
   * Various release-X.Y branches exist for previous releases.
   * We'd love to have your contributions, read [Community Information](http://docs.ansible.com/community.html) for notes on how to get started.

Authors
=======

Ansible was created by [Michael DeHaan](https://github.com/mpdehaan) (michael.dehaan/gmail/com) and has contributions from over 1000 users (and growing).  Thanks everyone!

Ansible is sponsored by [Ansible, Inc](http://ansible.com)


