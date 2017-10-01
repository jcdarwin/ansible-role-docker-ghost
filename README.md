mebooks.ansible-role-docker-ghost
=================================

Based on [rgarrigue.docker-ghost-blog](https://github.com/rgarrigue/ansible-role-docker-ghost-blog).

This role installs the [dockerised ghost blog](https://github.com/jcdarwin/docker-ghost-alpine) on an Ubuntu server.

We presume the use of [Traefik](traefik.io) as our reverse proxy, and use git to back our blog content up to a repository.

We assume that we've probably run:

* [ansible-role-users](https://github.com/jcdarwin/ansible-role-users)
* [ansible-role-common](https://github.com/jcdarwin/ansible-role-common)
* [ansible-role-docker](https://github.com/jcdarwin/ansible-role-docker)

The above all chain as dependencies of our main dependency:

* [ansible-role-docker-traefik](https://github.com/jcdarwin/ansible-role-docker-traefik)

It will be available at https://blog.domain.tld, and administration interface is at https://blog.domain.tld/admin

Blog content is under `/etc/ghost`

Requirements
------------

Developed and tested on *Ubuntu Server 16.10 Yakkety*, but should work on other OSes.

It's running on Docker.

Role Variables
--------------

For let's encrypt certificate, and automatic reverse proxy

- `owner`  defaults to *admin*
- `ansible_role_docker_ghost.domain` defaults to *domain.tld*
- `ansible_role_docker_ghost.install_dir` defaults to */etc/ghost*
- `ansible_role_docker_ghost.remote` defaults to *git@bitbucket.org:whever/blog.git*
- `ansible_role_docker_ghost.mail.transport` defaults to *SMTP*
- `ansible_role_docker_ghost.mail.smtp_service` defaults to *Mailgun*
- `ansible_role_docker_ghost.mail.user` defaults to *postmaster@blog.domain.tld*
- `ansible_role_docker_ghost.mail.pass` defaults to *password*

Dependencies
------------

- `mebooks.ansible-role-docker-traefik`

Example Playbook
----------------

```yml
- hosts: remote
  remote_user: deploy
  become: true
  become_user: root
  become_method: sudo

  vars:
    traefik_testing: true
    owner: administrator
    domain: mebooks.co.nz
    email: mebooks.support@gmail.com
    users:
     # owner_password / owner_password_encrypted are defined in the unversioned group_vars/remote
    - username: "{{ owner }}"
      password: "{{ owner_password }}"
      acl:
      - traefik

    ghost:
      domain: blog.mebooks.co.nz
      install_dir: /etc/ghost
      remote: git@bitbucket.org:jcdarwin/blog.git
      # theme_url: https://jcdarwin@bitbucket.org/jcdarwin/ghost-theme-goblin.git
      # theme_name: goblin
      mail:
        transport: SMTP # or SES
        smtp_service: Mailgun # or ~
        user: postmaster@blog.mebooks.co.nz
		# pass is defined in the unversioned group_vars/remote
        # pass: WHATEVER

  roles:
    # We presume we've already run ansible-role-users and ansible-role-common
    # The following role is listed as dependencies in ansible-role-docker-traefik/meta/main.yml:
    # - role: ansible-role-docker
    # The following role is listed as dependencies in ansible-role-docker-ghost/meta/main.yml:
    # - role: ansible-role-docker-traefik
    - role: ansible-role-docker-ghost
```

License
-------

GPLv3

Author Information
------------------

Jason Darwin
