# aspects_packages
An Ansible utility role that helps manage OS package state based on what OS is on the target. Can also manage simple pip packages.
  
# Requirements
Set ```hash_behaviour=merge``` in your ansible.cfg file.

`python-apt` should be installed on Debian and Ubuntu targets.
  
# Role Variables
### `aspects_packages_enabled`
Boolean. Whether the tasks in this role should be run or not.

Default: `False`

Set to `True` to run this role.

### `aspects_packages_prerequisite_packages`
A dictionary/hash of packages that should be installed before any others.

Uses the same format as `aspects_packages_packages`.

Currently only used for CentOS 7, since CentOS uses packages to configure extra repositories like the EPEL repo.

### `aspects_packages_packages.epel`
This controls the Extra Packages for Enterprise Linux (EPEL) `yum` repository. Currently it is only configured for CentOS 7.

The default value of `aspects_packages_packages.epel.state` is `default`. So the repository will not be affected by this role unless it is changed.

Set `aspects_packages_packages.epel.state` to `present` to enable the repository.

Set `aspects_packages_packages.epel.state` to `absent` to disable the repository.

See the [AdditionalResources/Repositories](https://wiki.centos.org/AdditionalResources/Repositories) page on the CentOS wiki for more details.

### `aspects_packages_packages`
A dictionary/hash of packages to install.

Use this pattern:

```yaml
aspects_packages_packages:
  <package key>:
    state: "<present or latest>"
    <ansible_distribution>:
      <ansible_distribution_version or ansible_distribution_major_version>: "<package name>"
```
Set `state` to "default" if you wish to list a package but not install it.

Set `state` to "absent" if you wish to remove a package.

Check the [tasks/aptInstallpackages.yml](aptInstallpackages.yml) or [tasks/yumInstallPackages.yml](yumInstallPackages.yml) files to find out what values are accepted for the `ansible_distribution_*` variables.

### `aspects_packages_pip_enabled`
Boolean. Whether the role should run pip tasks.

Default: `False`

Set to `True` to run pip tasks.

> Remember to install pip on your target system first.

### `aspects_packages_pip_packages`
A dictionary/hash of packages to install.

Use this pattern:

```yaml
aspects_packages_pip_packages:
  < key >:
    state: "< present|absent >"
    name: "< package name >"
```

# Dependencies
None.

# Example Playbook
```yaml
- hosts:
  - aspectscentos7
  - aspectsoraclelinux7
  - aspectsxenial
  - aspectsbionic
  - aspectsstretch
  vars:
    aspects_packages_enabled: True
    aspects_packages_add_repo_apt_repos:
      git:
        enabled: True
        repo: "ppa:git-core/ppa"
    aspects_packages_add_repo_yum_repos:
      docker-ce-stable:
        state: "present"
        name: "docker-ce-stable"
        description: "docker-ce-stable"
        gpgkey: "https://download.docker.com/linux/centos/gpg"
        baseurl: "https://download.docker.com/linux/centos/7/$basearch/stable"
        sslverify: "yes"
        repo_gpgcheck: "no"
        gpgcheck: "no"
        enabled: "yes"
        sslcacert: "/etc/ssl/certs/ca-bundle.crt"
        metadata_expire: "300"
      docker-ce-stable-source:
        state: "present"
        name: "docker-ce-stable-source"
        description: "docker-ce-stable-source"
        gpgkey: "https://download.docker.com/linux/centos/gpg"
        baseurl: "https://download.docker.com/linux/centos/7/source/stable"
        sslverify: "yes"
        repo_gpgcheck: "no"
        gpgcheck: "no"
        enabled: "yes"
        sslcacert: "/etc/ssl/certs/ca-bundle.crt"
        metadata_expire: "300"
    aspects_packages_prerequisite_packages:
      epel-release:
        state: "present"
        CentOS:
          7: "epel-release"
      oracle-epel-release-el7:
        state: "present"
        OracleLinux:
          7 : "oracle-epel-release-el7"
      oracle-release-el7:
        state: "present"
        OracleLinux:
          7: "oracle-release-el7"
      oracle-softwarecollection-release-el7:
        state: "present"
        OracleLinux:
          7: "oracle-softwarecollection-release-el7"
    aspects_packages_packages:
      htop:
        state: "present"
        Ubuntu:
          1604: "htop"
          1804: "htop"
        CentOS:
          7: "htop"
        Debian:
          9: "htop"
      vim:
        state: "present"
        Ubuntu:
          1404: "vim"
          1604: "vim"
        CentOS:
          7: "vim"
        Debian:
          9: "vim"
      python_apt:
        state: "present"
        Ubuntu:
          1604: "python-apt"
          1804: "python-apt"
        Debian:
          9: "python-apt"
      python_pip:
        state: "present"
        Ubuntu:
          1604: "python-pip"
          1804: "python-pip"
        Debian:
          9: "python-pip"
        CentOS:
          7: "python2-pip"
        OracleLinux:
          7: "python2-pip"
    aspects_packages_pip_enabled: True
    aspects_packages_pip_packages:
      passlib:
        state: "present"
        name: "passlib"
  roles:
  - aspects_packages
```

# License
MIT