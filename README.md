# Ansible Role: Bitbucket Sources

[![Build Status](https://travis-ci.org/cognifloyd/ansible-role-bitbucket-sources.svg?branch=master)](https://travis-ci.org/cognifloyd/ansible-role-bitbucket-sources)

This role clones/pulls a bitbucket cloud repository using an access key. The clone repository can be located anywhere owned by an indicated user. If the user or location does not exist, they will be created.

This role can be used more than once when using `include_role`. Other methods are untested.

For now, only git has been tested, but, theoretically hg should work as well. 
## Requirements

This role assumes that the source control executable (git or hg) is already installed. If you need to specify a non-standard executable location, set the optional variable `bitbucket_sources_executable`.

## Role Variables

### Constructing the Clone URL

This role uses these three variables (which I will conveniently reference as `<type>`, `<account>`, and `<name>`) to generate the bitbucket cloud repo url:

 - `bitbucket_sources_repo_type` _(valid options: "git", "hg")_
 - `bitbucket_sources_repo_account`
 - `bitbucket_sources_repo_name`

Based on whether `<type>` is git or hg, the url will be (see the [bitbucket docs][1]):

 - `ssh://git@bitbucket.org/<account>/<name>.git`
 - `ssh://hg@bitbucket.org/<account>/<name>/`

You can set the `bitbucket_sources_altssh` boolean to "`yes`" to use the [altssh urls][2] instead:

 - `ssh://git@altssh.bitbucket.org:443/<account>/<name>/`
 - `ssh://hg@altssh.bitbucket.org:443/<account>/<name>/`

### The Clone Destination

The repository will be cloned as `bitbucket_sourcces_dest` owned by `bitbucket_sources_owner:bitbucket_sources_group` (conveniently referenced as `<dest>`, `<owner>`, and `<group>`). The parent directory of `<dest>` must be a directory owned by `<owner>:<group>` and will be created if it doesn't exist. The directory will have the permissions mode defined in `bitbucket_sources_mode`.

The clone will be created by `bitbucket_sources_owner` with group `gadget_pppd_clone_group`, and will have the permissions of that user/group.

Bitbucket requires some kind of credentials to access a repository, so you'll need to provide a [bitbucket access key][3] in `bitbucket_sources_key`.

** defaults/main.yml **:
```yaml
bitbucket_sources_repo_type: git
bitbucket_sources_owner: "{{ ansible_user }}"
bitbucket_sources_group: "{{ ansible_user }}"
bitbucket_sources_mode: 0755
bitbucket_sources_altssh: no
bitbucket_sources_key_dest: "~{{ bitbucket_sources_owner }}/.ssh/{{ bitbucket_sources_key | basename }}"
```

** vars/main.yml **:
```yaml
none
```

** role parameters **:

You must set these as role parameters (There is no default, and an assertion will fail if they are not defined):
```yaml
bitbucket_sources_repo_account: "<bitbucket user>"
bitbucket_sources_repo_name: "<bitbucket repo (without .git)>"
bitbucket_sources_dest: "~<user>/scm/<account>/<name>.git"
bitbucket_sources_key: "~/.ssh/access_key"
```

You may also override any of the defaults (see above). Other optional variables include:
```yaml
bitbucket_sources_version: a83b8a42
bitbucket_sources_executable: "/home/acme/gentoo-prefix/usr/bin/git"
```

** TODO: ** I don't know how to make hg use the indicated key.

** global scope vars **:
Any variables that are read from global scope (ie. hostvars, group vars, etc.)

By default `<owner>` and `<group>` are set to `ansible_user`.
```yaml
ansible_user
```

** vars from other roles **:
Any variables that are read from other roles
```yaml
none
```

## Dependencies

No external dependencies.

## Example Playbook

```yaml
- hosts: vagrant
  tasks:
    - name: Clone or update example-magnificent from bitbucket.
      include_role:
        name: cognifloyd.bitbucket-sources
        allow_duplicates: yes
        private: yes
      vars:
        bitbucket_sources_repo_type: git
        bitbucket_sources_repo_account: example
        bitbucket_sources_repo_name: magnificent
        bitbucket_sources_dest: /var/scm/bitbucket/example/magnificent.git
        bitbucket_sources_owner: vagrant
        bitbucket_sources_group: vagrant
        bitbucket_sources_key: "~/.ssh/example_access_key"
```

## License

MIT

## Author Information

Created by [Jacob Floyd](https://github.com/cognifloyd), employed by [Theatro](theatro.com), in 2017. I extracted these tasks from another playbook I was writing. After writing much of this, I found [webbylab.sources](https://galaxy.ansible.com/webbylab/sources/) and [Stouts.source](https://galaxy.ansible.com/Stouts/source/). The name "sources" was so much better than my working "bitbucket-repo-clone", so I used "bitbucket-sources" instead. Sadly, I'm using EL 7, so these Ubuntu-focused roles would have required adaptation, even if I had found them before writing much of this role. Even though I didn't reuse more than ideas, they deserve credit for thinking of this before I did.


<!-- footnote reference links -->
[1]: https://confluence.atlassian.com/bitbucket/use-the-ssh-protocol-with-bitbucket-cloud-221449711.html#UsetheSSHprotocolwithBitbucketCloud-RepositoryURLformatsbyconnectionprotocol
[2]: https://confluence.atlassian.com/bitbucket/use-the-ssh-protocol-with-bitbucket-cloud-221449711.html#UsetheSSHprotocolwithBitbucketCloud-SSHonPort443
[3]: https://confluence.atlassian.com/bitbucket/use-access-keys-294486051.html
