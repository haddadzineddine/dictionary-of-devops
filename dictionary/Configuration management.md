---
description: Using tools to install software and keep settings consistent on existing servers, such as with Ansible, Puppet, or Chef.
---

Configuration management is the use of tools to install software, write config files, manage users, and keep services running on existing servers in a defined way. Instead of logging in to each machine, you describe the desired setup once and apply it across a fleet. Ansible, Puppet, Chef, and Salt are established examples.

```yaml
- name: Ensure nginx is installed
  ansible.builtin.apt:
    name: nginx
    state: present

- name: Ensure nginx is running
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

The important property is idempotence: running the same configuration many times gives the same result as running it once. The task above installs nginx only when it is missing. Compare a shell script containing `echo "option" >> app.conf`, which appends a duplicate line on every run until the service refuses to start or behaves strangely.

Tools differ in how changes reach a machine:

| Model | How it works                                 | Examples      | Trade-off                                  |
| ----- | -------------------------------------------- | ------------- | ------------------------------------------ |
| Push  | A control machine connects, often over SSH   | Ansible       | No agent, but runs only when triggered     |
| Pull  | An agent on each server fetches its config   | Puppet, Chef  | Corrects drift regularly, but needs agents |

Configuration management is often confused with [infrastructure as code](./Infrastructure%20as%20code.md) tools for provisioning. [Terraform](./Terraform.md) typically creates the server, network, and disk, and configuration management sets up what runs inside the machine. The two are commonly used together.

These tools only manage what they are told to manage. A package installed by hand stays on the machine even after the playbook is corrected, so long-lived servers still gather [configuration drift](./Configuration%20drift.md). A run can also fail halfway through a fleet, leaving some hosts updated and others not.

[Container](./Container.md)-based systems reduce the need for this work, because the software is baked into an [image](./Container%20image.md) and replaced under [immutable infrastructure](./Immutable%20infrastructure.md). Configuration management remains useful for virtual machines, network devices, and building base images.

_Usage:_

"Half the fleet has the new TLS settings and the other half doesn't."

"The Ansible run stopped at host 40. It's idempotent, so fix the failure and run the configuration management playbook again."
