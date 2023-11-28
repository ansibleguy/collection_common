# Ansible Collection - ansibleguy.common

[![Functional Test Status](https://badges.ansibleguy.net/common.collection.test.svg)](https://github.com/ansibleguy/collection_common/blob/latest/scripts/test.sh)
[![Lint Test Status](https://badges.ansibleguy.net/common.collection.lint.svg)](https://github.com/ansibleguy/collection_common/blob/latest/scripts/lint.sh)
[![Docs](https://readthedocs.org/projects/common_ansible/badge/?version=latest&style=flat)](https://common.ansibleguy.net)
[![Ansible Galaxy](https://badges.ansibleguy.net/galaxy.badge.svg)](https://galaxy.ansible.com/ui/repo/published/ansibleguy/common)

----

## Setup

```bash
# latest version:
ansible-galaxy collection install git+https://github.com/ansibleguy/collection_common.git

# stable/tested version:
ansible-galaxy collection install ansibleguy.common

# install to specific directory for easier development
cd $PLAYBOOK_DIR
ansible-galaxy collection install git+https://github.com/ansibleguy/collection_common.git -p ./collections
```

----

## Usage

See: [Docs](https://common.ansibleguy.net)

----

## Modules


| Function | Module              | Usage                                                          |
|:---------|:--------------------|:---------------------------------------------------------------|
| **?**    | ansibleguy.common.? | [Docs](https://common.ansibleguy.net/en/latest/modules/?.html) |

----

## About

Long story short: This collection of Ansible Modules is about performance!

### Why?

#### Performance

Ansible has some core modules that are very stable and good to use - but slow.

That is understandable as the underlying code has to check for **multiple platforms, many edge-cases and has switch-cases for a whole feature-set**.

Some of these points are important aspects of some code that you may want to automate your IT-infrastructure and CI/CD workflows with.

But in some cases it just gets too slow.

**Examples**:

* Copying (_lots of_) files
* Executing a module like [ansible.builtin.stat](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/stat_module.html) that collects lots of information about the target filesystem-object - but 99% of the time you are actually using only one specific method of it (_stat.exists_)


Why might those operations be slow? And what could be improved?

* Many modules are optimized for single operations - mass operations do not scale well

  For those operations the handling can be optimized

* Modules have a logical scope of features they support

  If the user knows the exact method he needs beforehand - it can save a lot of time processing

* Some/many management connections utilize SSH as transport-protocol

  Initiating SSH connections can be pretty slow/expensive. And I don't even talk about when you have high latency on your HomeOffice LTE/5G (;

  There are some options to boost your SSH performance - but that won't solve the base problem of not using as few connections as somehow possible.

  Even if HTTP is uses as transport - the connection latency adds up over multiple requests.


#### Ease of use

Some of Ansible's modules are just a little pain to use.

**Examples**:

* [ansible.builtin.pause](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/pause_module.html) when used for interactive prompts.

  The input is only set to the first host in the run and the path `hostvars[play_hosts.0].user_input.results` speaks for itself..

* [ansible.buitin.systemd](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html) not being able to show `journalctl` logs

  Needing to catch the module failure and pulling the logs using 'ansible.builtin.command' is a little tedious for my taste

----

### Why don't contribute to Ansible-Core?

It does not really make sense.

Ansible core modules are built for stability. It is not that easily possible to 'just refactor its core behaviour'. (;

Even if that would be possible - I personally like to see results fast - not spend my precious time discussing.

Also, the argument of 'performance' will not be interesting to most Ansible users. (_as the tasks are ran - you guessed it - automated_)

----

### How?

I want to create single modules that perform exactly ONE specific task.

They will be optimized for their target platform and usability.

I will add a score and benchmarks to each module. Using as few resources as possible is the priority.

**Examples**:

* ansibleguy.common.debian_file_exists => {'msg': 'yes', 'exists': true}

  May also have an option to fail if the element does not exist

* ansibleguy.common.debian_copy_mass

  * Local: pulling hashes of local files
  * Request 1: fetching hashes of affected remote file
  * Local: compare
  * Request 2: copying changed files to target system
  * Request 3:
    - creating diff for files with content-type text
    - setting privileges
    - local copying

* ansibleguy.common.debian_systemd_restart => {'msg': 'done', 'failed': false, 'errors': []}
* ansibleguy.common.debian_systemd_start => {'msg': 'done', 'changed': true, 'failed': false, 'errors': []}

