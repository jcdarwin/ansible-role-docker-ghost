mebooks.ansible-role-docker-ghost
=================================

Based on [rgarrigue.docker-ghost-blog](https://github.com/rgarrigue/ansible-role-docker-ghost-blog).

This role installs Ghost 6 on an Ubuntu server, using the [official `ghost` Docker image](https://hub.docker.com/_/ghost) pinned to `ghost:6.65.0-alpine`, with SQLite as the database. Ghost is configured through environment variables on the container; there is no `config.production.json`.

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

Deploy Key
----------

`ansible-role-users` generates a single SSH keypair on the server at `/root/.ssh/id_rsa` and prints the public key — it does **not** register it with any git remote for you. Before this role can clone (and before the daily cron job can force-push) `ghost.remote`, a public key must be manually added as a **deploy key with write access** on the remote repo:

* GitHub: repo → Settings → Deploy keys → Add deploy key → tick "Allow write access"
* Bitbucket: repo → Repository settings → Access keys → Add key

GitHub does not allow the same public key to be used as a deploy key on more than one repository. If this server already uses its shared `/root/.ssh/id_rsa` as a deploy key elsewhere (e.g. for a wiki repo via `ansible-role-docker-gollum`), generate a dedicated keypair for the blog repo instead and add an alias in `/root/.ssh/config`, e.g.:

```
ssh-keygen -t ed25519 -f /root/.ssh/id_ed25519_blog_github -N '' -C "ghost-blog-deploy-key"
```

```
Host github.com-blog
    HostName github.com
    User git
    IdentityFile /root/.ssh/id_ed25519_blog_github
    IdentitiesOnly yes
```

Then set `ghost.remote` to use that alias host, e.g. `git@github.com-blog:org/blog.git`, instead of `git@github.com:org/blog.git` directly. Note this SSH config/keypair is manual, server-side setup — it isn't (currently) created by this role, so it won't exist after a from-scratch redeploy until repeated.

Role Variables
--------------

For let's encrypt certificate, and automatic reverse proxy

- `ghost.owner`  defaults to *admin*
- `ghost.owner_password`  defaults to *WHATEVER*
- `ghost.domain` defaults to *domain.tld*
- `ghost.source`: defaults to *domain.tld*
- `ghost.install_dir` defaults to */etc/ghost*
- `ghost.remote` defaults to *git@github.com:whoever/blog.git*
The `ghost.mail.*` variables are not currently wired into the container, so transactional mail (staff invites, password resets) is not configured.

- `ghost.mail.transport` defaults to *SMTP*
- `ghost.mail.smtp_service` defaults to *Mailgun*
- `ghost.mail.user` defaults to *postmaster@blog.domain.tld*
- `ghost.mail.pass` defaults to *password*

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
    traefik:
      traefik_testing: true
      owner: administrator
      domain: mebooks.co.nz
      email: mebooks.support@gmail.com
      users:
      # owner_password / owner_password_encrypted are defined in the unversioned group_vars/remote
      - username: "{{ ghost.owner }}"
        password: "{{ ghost.owner_password }}"
        acl:
        - traefik

    ghost:
      domain: blog.mebooks.co.nz
      install_dir: /etc/ghost
      remote: git@github.com-blog:nzmebooks/blog.git
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
