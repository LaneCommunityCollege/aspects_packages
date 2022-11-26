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

### aspects_packages_add_repo_apt_repos
A dictionary/hash of apt repositories to add.

See the example playbook for how to mix and match the different ways to set the key.

```yaml
aspects_packages_add_repo_apt_repos:
  < package key >:
    enabled: < True or False >
    repo:
      < OS >:
        < OS Release >: "< deb or ppa line >"
    key_url:
      < OS >:
        < OS Release >: "< url to key >"
    keyserver:
      < OS >:
        < OS Release >: "< keyserver url >"
    key_id:
      < OS >:
        < OS Release >: "< key id >"
    data:
      < OS >:
        < OS Release >: |
          < data >
```
### aspects_packages_add_repo_apt_keys
A dictionary/hash of apt keys to add. Currently only supports Ubuntu 22.04.

I believe this method is needed for Debian 11 and up. But I have not tested to find out.

This is due to apt-key being deprecated. See [this issue](https://github.com/ansible/ansible/issues/78063) for details.

```yaml
aspects_packages_add_repo_apt_keys:
  < repo name >:
    enabled: < True|False >
    Ubuntu:
      2204:
        key_url: "< url of the key >"
        key_dest: "/etc/apt/keyrings/< repo name >.asc"
```
Then, in `aspects_packages_add_repo_apt_repos`, set the `repo` line to look like:
```yaml
aspects_packages_add_repo_apt_repos:
  < repo name >:
    Ubuntu:
      2204: "deb [arch=amd64 signed-by=/etc/apt/keyrings/< repo name >.asc] < rest of the line here >"
```
> Note: If you use the `.gpg` extension, you need to run `gpg --dearmor` on the file. See [Docker's docs](https://docs.docker.com/engine/install/ubuntu/) for an example.

### aspects_packages_add_repo_yum_repos

```yaml
aspects_packages_add_repo_yum_repos:
  < repo key >:
    < OS >:
      < OS Release >:
        state: "< present | absent >"
        name: "< what the file in /etc/yum.repos.d will be called >"
        description: "< short description of the repo >"
        gpgkey: "< url of the gpg key >"
        baseurl: "< the yum repo url >"
        sslverify: "< yes or no >"
        repo_gpgcheck: "< yes or no >"
        gpgcheck: "< yes or no >"
        enabled: "< yes or no >"
```

### aspects_packages_package_urls
Default: undefined.

Dictionary/hash of deb files to download and install.

```yaml
aspects_packages_package_urls:
  < key >:
    enabled: < True | False >
    state: < present | absent >
    url: < url to .deb file >
```

### aspects_packages_snap_items
Default: undefined.

Dictionary/hash of snaps to install or remove.

```yaml
aspects_packages_snap_items:
  < key >:
    enabled: < True | False >
    state: < present | absent >
    name: < name of snap >
```

### aspects_packages_flatpak_remotes
Default: undefined.

Dictionary/hash of Flatpak remotes to install or remove.

```yaml
aspects_packages_flatpak_remotes:
  < key >:
    enabled: < True | False >
    state: < present | absent >
    name: < name of remote >
    flatpakrepo_url: < remote url >
    executable: < flatpak executable. Defaults to 'flatpak' >
    method: < system|user. Defaults to 'system' >
```

### aspects_packages_flatpak_packages
Default: undefined.

Dictionary/hash of Flatpak packages to install or remove.

```yaml
aspects_packages_flatpak_packages:
  < key >:
    enabled: < True | False >
    state: < present | absent >
    name: < name or url of package >
    remote: < name of remote. Defaults to 'flathub' >
    executable: < flatpak executable. Defaults to 'flatpak' >
    method: < system|user Defaults to 'system' >
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
    aspects_packages_prerequisite_packages:
      epel-release:
        state: "present"
        CentOS:
          7: "epel-release"
        OracleLinux:
          7: "oracle-epel-release-el7"
      apt-transport-https:
        state: "latest"
        Debian:
          9: "apt-transport-https"
          10: "apt-transport-https"
        Ubuntu:
          1604: "apt-transport-https"
          1804: "apt-transport-https"
          2004: "apt-transport-https"
      dirmngr:
        state: "present"
        Debian:
          9: "dirmngr"
          10: "dirmngr"
        Ubuntu:
          1604: "dirmngr"
          1804: "dirmngr"
          2004: "dirmngr"
      gnupg:
        state: "present"
        Debian:
          9: "gnupg"
          10: "gnupg"
        Ubuntu:
          1604: "gnupg"
          1804: "gnupg"
          2004: "gnupg"
      ca-certificates:
        state: "present"
        Debian:
          9: "ca-certificates"
          10: "ca-certificates"
        Ubuntu:
          1604: "ca-certificates"
          1804: "ca-certificates"
          2004: "ca-certificates"
      ftp:
        state: "purged"
        Ubuntu:
          2004: "ftp"
          1604: "ftp"
          1804: "ftp"
        Debian:
          9: "ftp"
        CentOS:
          7: "ftp"
        OracleLinux:
          7: "ftp"
      wpasupplicant:
        state: "default"
        Ubuntu:
          2004: "wpasupplicant"
          1604: "wpasupplicant"
          1804: "wpasupplicant"
        Debian:
          9: "wpasupplicant"
        CentOS:
          7: "wpa_supplicant"
        OracleLinux:
          7: "wpa_supplicant"
      ppp:
        state: "absent"
        Ubuntu:
          2004: "ppp"
          1604: "ppp"
          1804: "ppp"
        Debian:
          9: "ppp"
        CentOS:
          7: "ppp"
        OracleLinux:
          7: "ppp"
    aspects_packages_packages:
      curl:
        state: "latest"
        Ubuntu:
          2004: "curl"
          1604: "curl"
          1804: "curl"
        Debian:
          9: "curl"
          10: "curl"
        CentOS:
          7: "curl"
        OracleLinux:
          7: "curl"
      openssh:
        state: "present"
        Ubuntu:
          2004: "openssh-server"
          1604: "openssh-server"
          1804: "openssh-server"
        Debian:
          9: "openssh-server"
        CentOS:
          7: "openssh-server"
        OracleLinux:
          7: "openssh-server"
      locate:
        state: "present"
        Ubuntu:
          2004: "mlocate"
          1604: "mlocate"
          1804: "mlocate"
        Debian:
          9: "mlocate"
          10: "mlocate"
        CentOS:
          7: "mlocate"
        OracleLinux:
          7: "mlocate"
      git:
        state: "present"
        Ubuntu:
          2004: "git"
          1604: "git"
          1804: "git"
        Debian:
          9: "git"
          10: "git"
        CentOS:
          7: "git"
        OracleLinux:
          7: "git"
      htop:
        state: "present"
        Ubuntu:
          2004: "htop"
          1604: "htop"
          1804: "htop"
        CentOS:
          7: "htop"
        OracleLinux:
          7: "htop"
        Debian:
          9: "htop"
          10: "htop"
      vim:
        state: "present"
        Ubuntu:
          2004: "vim"
          1604: "vim"
          1804: "vim"
        Debian:
          9: "vim"
          10: "vim"
        CentOS:
          7: "vim"
      python_apt:
        state: "present"
        Ubuntu:
          2004: "python-apt"
          1604: "python-apt"
          1804: "python-apt"
        Debian:
          9: "python-apt"
      nfs-utils:
        state: "present"
        Ubuntu:
          2004: "nfs-common"
          1604: "nfs-common"
          1804: "nfs-common"
        Debian:
          9: "nfs-common"
        CentOS:
          7: "nfs-utils"
        OracleLinux:
          7: "nfs-utils"
      landscape-common:
        state: "purged"
        Debian: "landscape-common"
        Ubuntu:
          2004: "landscape-common"
          1604: "landscape-common"
          1804: "landscape-common"
      laptopdetect:
        state: "purged"
        Ubuntu:
          2004: "laptop-detect"
          1604: "laptop-detect"
          1804: "laptop-detect"
        Debian:
          9: "laptop-detect"
      mtrtiny:
        state: "purged"
        Ubuntu:
          2004: "mtr-tiny"
          1604: "mtr-tiny"
          1804: "mtr-tiny"
        Debian:
          9: "mtr-tiny"
      popularity-contest:
        state: "purged"
        Ubuntu:
          2004: "popularity-contest"
          1604: "popularity-contest"
          1804: "popularity-contest"
        Debian:
          9: "popularity-contest"
      pppconfig:
        state: "purged"
        Ubuntu:
          2004: "pppconfig"
          1604: "pppconfig"
          1804: "pppconfig"
        Debian:
          9: "pppconfig"
      pppoeconf:
        state: "purged"
        Ubuntu:
          2004: "pppoeconf"
          1604: "pppoeconf"
          1804: "pppoeconf"
        Debian:
          9: "pppoeconf"
      tasksel:
        state: "purged"
        Ubuntu:
          2004: "tasksel"
          1604: "tasksel"
          1804: "tasksel"
        Debian:
          9: "tasksel"
      tasksel-data:
        state: "purged"
        Ubuntu:
          2004: "tasksel-data"
          1604: "tasksel-data"
          1804: "tasksel-data"
        Debian:
          9: "tasksel-data"
      wireless-tools:
        state: "purged"
        Ubuntu:
          2004: "wireless-tools"
          1604: "wireless-tools"
          1804: "wireless-tools"
        Debian:
          9: "wireless-tools"
        CentOS:
          7: "wireless-tools"
      python3apport:
        state: "purged"
        Ubuntu:
          2004: "python3-apport"
          1604: "python3-apport"
          1804: "python3-apport"
      libselinuxpython:
        state: "present"
        CentOS:
          7: "libselinux-python"
        OracleLinux:
          7: "libselinux-python"
      unzip:
        state: "present"
        Ubuntu:
          2004: "unzip"
          1604: "unzip"
          1804: "unzip"
        Debian:
          9: "unzip"
        CentOS:
          7: "unzip"
        OracleLinux:
          7: "unzip"
      tcsh:
        state: "absent"
        Ubuntu:
          2004: "tcsh"
          1604: "tcsh"
          1804: "tcsh"
        Debian:
          10: "tcsh"
        CentOS:
          7: "tcsh"
        OracleLinux:
          7: "tcsh"
      dos2unix:
        state: "purged"
        CentOS:
          7: "dos2unix"
        OracleLinux:
          7: "dos2unix"
    aspects_packages_add_repo_apt_repos:
      git-core:
        enabled: True
        repo:
          Ubuntu:
            1604: "ppa:git-core/ppa"
            1804: "ppa:git-core/ppa"
            2004: "ppa:git-core/ppa"
      syncthing:
        enabled: True
        repo:
          Ubuntu:
            1804: "deb https://apt.syncthing.net/ syncthing stable"
        key_url:
          Ubuntu:
            1804: "https://syncthing.net/release-key.txt"
      ansible:
        enabled: False
        repo:
          Ubuntu:
            1604: "ppa:ansible/ansible"
            1804: "ppa:ansible/ansible"
      docker:
        enabled: True
        repo:
          Ubuntu:
            1804: "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
          Debian:
            9: "deb [arch=amd64] https://download.docker.com/linux/debian stretch stable"
        key_url:
          Ubuntu:
            1804: "https://download.docker.com/linux/ubuntu/gpg"
          Debian:
            9: "https://download.docker.com/linux/debian/gpg"
      elastic:
        enabled: True
        repo:
          Ubuntu:
            1604: "deb https://artifacts.elastic.co/packages/7.x/apt stable main"
            1804: "deb https://artifacts.elastic.co/packages/7.x/apt stable main"
            2004: "deb https://artifacts.elastic.co/packages/7.x/apt stable main"
          Debian:
            9: "deb https://artifacts.elastic.co/packages/7.x/apt stable main"
            10: "deb https://artifacts.elastic.co/packages/7.x/apt stable main"
        data:
          Ubuntu:
            1604: |
              -----BEGIN PGP PUBLIC KEY BLOCK-----
              Version: GnuPG v2.0.14 (GNU/Linux)

              mQENBFI3HsoBCADXDtbNJnxbPqB1vDNtCsqhe49vFYsZN9IOZsZXgp7aHjh6CJBD
              A+bGFOwyhbd7at35jQjWAw1O3cfYsKAmFy+Ar3LHCMkV3oZspJACTIgCrwnkic/9
              CUliQe324qvObU2QRtP4Fl0zWcfb/S8UYzWXWIFuJqMvE9MaRY1bwUBvzoqavLGZ
              j3SF1SPO+TB5QrHkrQHBsmX+Jda6d4Ylt8/t6CvMwgQNlrlzIO9WT+YN6zS+sqHd
              1YK/aY5qhoLNhp9G/HxhcSVCkLq8SStj1ZZ1S9juBPoXV1ZWNbxFNGwOh/NYGldD
              2kmBf3YgCqeLzHahsAEpvAm8TBa7Q9W21C8vABEBAAG0RUVsYXN0aWNzZWFyY2gg
              KEVsYXN0aWNzZWFyY2ggU2lnbmluZyBLZXkpIDxkZXZfb3BzQGVsYXN0aWNzZWFy
              Y2gub3JnPokBOAQTAQIAIgUCUjceygIbAwYLCQgHAwIGFQgCCQoLBBYCAwECHgEC
              F4AACgkQ0n1mbNiOQrRzjAgAlTUQ1mgo3nK6BGXbj4XAJvuZDG0HILiUt+pPnz75
              nsf0NWhqR4yGFlmpuctgCmTD+HzYtV9fp9qW/bwVuJCNtKXk3sdzYABY+Yl0Cez/
              7C2GuGCOlbn0luCNT9BxJnh4mC9h/cKI3y5jvZ7wavwe41teqG14V+EoFSn3NPKm
              TxcDTFrV7SmVPxCBcQze00cJhprKxkuZMPPVqpBS+JfDQtzUQD/LSFfhHj9eD+Xe
              8d7sw+XvxB2aN4gnTlRzjL1nTRp0h2/IOGkqYfIG9rWmSLNlxhB2t+c0RsjdGM4/
              eRlPWylFbVMc5pmDpItrkWSnzBfkmXL3vO2X3WvwmSFiQbkBDQRSNx7KAQgA5JUl
              zcMW5/cuyZR8alSacKqhSbvoSqqbzHKcUQZmlzNMKGTABFG1yRx9r+wa/fvqP6OT
              RzRDvVS/cycws8YX7Ddum7x8uI95b9ye1/Xy5noPEm8cD+hplnpU+PBQZJ5XJ2I+
              1l9Nixx47wPGXeClLqcdn0ayd+v+Rwf3/XUJrvccG2YZUiQ4jWZkoxsA07xx7Bj+
              Lt8/FKG7sHRFvePFU0ZS6JFx9GJqjSBbHRRkam+4emW3uWgVfZxuwcUCn1ayNgRt
              KiFv9jQrg2TIWEvzYx9tywTCxc+FFMWAlbCzi+m4WD+QUWWfDQ009U/WM0ks0Kww
              EwSk/UDuToxGnKU2dQARAQABiQEfBBgBAgAJBQJSNx7KAhsMAAoJENJ9ZmzYjkK0
              c3MIAIE9hAR20mqJWLcsxLtrRs6uNF1VrpB+4n/55QU7oxA1iVBO6IFu4qgsF12J
              TavnJ5MLaETlggXY+zDef9syTPXoQctpzcaNVDmedwo1SiL03uMoblOvWpMR/Y0j
              6rm7IgrMWUDXDPvoPGjMl2q1iTeyHkMZEyUJ8SKsaHh4jV9wp9KmC8C+9CwMukL7
              vM5w8cgvJoAwsp3Fn59AxWthN3XJYcnMfStkIuWgR7U2r+a210W6vnUxU4oN0PmM
              cursYPyeV0NX/KQeUeNMwGTFB6QHS/anRaGQewijkrYYoTNtfllxIu9XYmiBERQ/
              qPDlGRlOgVTd9xUfHFkzB52c70E=
              =92oX
              -----END PGP PUBLIC KEY BLOCK-----
            1804: |
              -----BEGIN PGP PUBLIC KEY BLOCK-----
              Version: GnuPG v2.0.14 (GNU/Linux)

              mQENBFI3HsoBCADXDtbNJnxbPqB1vDNtCsqhe49vFYsZN9IOZsZXgp7aHjh6CJBD
              A+bGFOwyhbd7at35jQjWAw1O3cfYsKAmFy+Ar3LHCMkV3oZspJACTIgCrwnkic/9
              CUliQe324qvObU2QRtP4Fl0zWcfb/S8UYzWXWIFuJqMvE9MaRY1bwUBvzoqavLGZ
              j3SF1SPO+TB5QrHkrQHBsmX+Jda6d4Ylt8/t6CvMwgQNlrlzIO9WT+YN6zS+sqHd
              1YK/aY5qhoLNhp9G/HxhcSVCkLq8SStj1ZZ1S9juBPoXV1ZWNbxFNGwOh/NYGldD
              2kmBf3YgCqeLzHahsAEpvAm8TBa7Q9W21C8vABEBAAG0RUVsYXN0aWNzZWFyY2gg
              KEVsYXN0aWNzZWFyY2ggU2lnbmluZyBLZXkpIDxkZXZfb3BzQGVsYXN0aWNzZWFy
              Y2gub3JnPokBOAQTAQIAIgUCUjceygIbAwYLCQgHAwIGFQgCCQoLBBYCAwECHgEC
              F4AACgkQ0n1mbNiOQrRzjAgAlTUQ1mgo3nK6BGXbj4XAJvuZDG0HILiUt+pPnz75
              nsf0NWhqR4yGFlmpuctgCmTD+HzYtV9fp9qW/bwVuJCNtKXk3sdzYABY+Yl0Cez/
              7C2GuGCOlbn0luCNT9BxJnh4mC9h/cKI3y5jvZ7wavwe41teqG14V+EoFSn3NPKm
              TxcDTFrV7SmVPxCBcQze00cJhprKxkuZMPPVqpBS+JfDQtzUQD/LSFfhHj9eD+Xe
              8d7sw+XvxB2aN4gnTlRzjL1nTRp0h2/IOGkqYfIG9rWmSLNlxhB2t+c0RsjdGM4/
              eRlPWylFbVMc5pmDpItrkWSnzBfkmXL3vO2X3WvwmSFiQbkBDQRSNx7KAQgA5JUl
              zcMW5/cuyZR8alSacKqhSbvoSqqbzHKcUQZmlzNMKGTABFG1yRx9r+wa/fvqP6OT
              RzRDvVS/cycws8YX7Ddum7x8uI95b9ye1/Xy5noPEm8cD+hplnpU+PBQZJ5XJ2I+
              1l9Nixx47wPGXeClLqcdn0ayd+v+Rwf3/XUJrvccG2YZUiQ4jWZkoxsA07xx7Bj+
              Lt8/FKG7sHRFvePFU0ZS6JFx9GJqjSBbHRRkam+4emW3uWgVfZxuwcUCn1ayNgRt
              KiFv9jQrg2TIWEvzYx9tywTCxc+FFMWAlbCzi+m4WD+QUWWfDQ009U/WM0ks0Kww
              EwSk/UDuToxGnKU2dQARAQABiQEfBBgBAgAJBQJSNx7KAhsMAAoJENJ9ZmzYjkK0
              c3MIAIE9hAR20mqJWLcsxLtrRs6uNF1VrpB+4n/55QU7oxA1iVBO6IFu4qgsF12J
              TavnJ5MLaETlggXY+zDef9syTPXoQctpzcaNVDmedwo1SiL03uMoblOvWpMR/Y0j
              6rm7IgrMWUDXDPvoPGjMl2q1iTeyHkMZEyUJ8SKsaHh4jV9wp9KmC8C+9CwMukL7
              vM5w8cgvJoAwsp3Fn59AxWthN3XJYcnMfStkIuWgR7U2r+a210W6vnUxU4oN0PmM
              cursYPyeV0NX/KQeUeNMwGTFB6QHS/anRaGQewijkrYYoTNtfllxIu9XYmiBERQ/
              qPDlGRlOgVTd9xUfHFkzB52c70E=
              =92oX
              -----END PGP PUBLIC KEY BLOCK-----
            2004: |
              -----BEGIN PGP PUBLIC KEY BLOCK-----
              Version: GnuPG v2.0.14 (GNU/Linux)

              mQENBFI3HsoBCADXDtbNJnxbPqB1vDNtCsqhe49vFYsZN9IOZsZXgp7aHjh6CJBD
              A+bGFOwyhbd7at35jQjWAw1O3cfYsKAmFy+Ar3LHCMkV3oZspJACTIgCrwnkic/9
              CUliQe324qvObU2QRtP4Fl0zWcfb/S8UYzWXWIFuJqMvE9MaRY1bwUBvzoqavLGZ
              j3SF1SPO+TB5QrHkrQHBsmX+Jda6d4Ylt8/t6CvMwgQNlrlzIO9WT+YN6zS+sqHd
              1YK/aY5qhoLNhp9G/HxhcSVCkLq8SStj1ZZ1S9juBPoXV1ZWNbxFNGwOh/NYGldD
              2kmBf3YgCqeLzHahsAEpvAm8TBa7Q9W21C8vABEBAAG0RUVsYXN0aWNzZWFyY2gg
              KEVsYXN0aWNzZWFyY2ggU2lnbmluZyBLZXkpIDxkZXZfb3BzQGVsYXN0aWNzZWFy
              Y2gub3JnPokBOAQTAQIAIgUCUjceygIbAwYLCQgHAwIGFQgCCQoLBBYCAwECHgEC
              F4AACgkQ0n1mbNiOQrRzjAgAlTUQ1mgo3nK6BGXbj4XAJvuZDG0HILiUt+pPnz75
              nsf0NWhqR4yGFlmpuctgCmTD+HzYtV9fp9qW/bwVuJCNtKXk3sdzYABY+Yl0Cez/
              7C2GuGCOlbn0luCNT9BxJnh4mC9h/cKI3y5jvZ7wavwe41teqG14V+EoFSn3NPKm
              TxcDTFrV7SmVPxCBcQze00cJhprKxkuZMPPVqpBS+JfDQtzUQD/LSFfhHj9eD+Xe
              8d7sw+XvxB2aN4gnTlRzjL1nTRp0h2/IOGkqYfIG9rWmSLNlxhB2t+c0RsjdGM4/
              eRlPWylFbVMc5pmDpItrkWSnzBfkmXL3vO2X3WvwmSFiQbkBDQRSNx7KAQgA5JUl
              zcMW5/cuyZR8alSacKqhSbvoSqqbzHKcUQZmlzNMKGTABFG1yRx9r+wa/fvqP6OT
              RzRDvVS/cycws8YX7Ddum7x8uI95b9ye1/Xy5noPEm8cD+hplnpU+PBQZJ5XJ2I+
              1l9Nixx47wPGXeClLqcdn0ayd+v+Rwf3/XUJrvccG2YZUiQ4jWZkoxsA07xx7Bj+
              Lt8/FKG7sHRFvePFU0ZS6JFx9GJqjSBbHRRkam+4emW3uWgVfZxuwcUCn1ayNgRt
              KiFv9jQrg2TIWEvzYx9tywTCxc+FFMWAlbCzi+m4WD+QUWWfDQ009U/WM0ks0Kww
              EwSk/UDuToxGnKU2dQARAQABiQEfBBgBAgAJBQJSNx7KAhsMAAoJENJ9ZmzYjkK0
              c3MIAIE9hAR20mqJWLcsxLtrRs6uNF1VrpB+4n/55QU7oxA1iVBO6IFu4qgsF12J
              TavnJ5MLaETlggXY+zDef9syTPXoQctpzcaNVDmedwo1SiL03uMoblOvWpMR/Y0j
              6rm7IgrMWUDXDPvoPGjMl2q1iTeyHkMZEyUJ8SKsaHh4jV9wp9KmC8C+9CwMukL7
              vM5w8cgvJoAwsp3Fn59AxWthN3XJYcnMfStkIuWgR7U2r+a210W6vnUxU4oN0PmM
              cursYPyeV0NX/KQeUeNMwGTFB6QHS/anRaGQewijkrYYoTNtfllxIu9XYmiBERQ/
              qPDlGRlOgVTd9xUfHFkzB52c70E=
              =92oX
              -----END PGP PUBLIC KEY BLOCK-----
          Debian:
            9: |
              -----BEGIN PGP PUBLIC KEY BLOCK-----
              Version: GnuPG v2.0.14 (GNU/Linux)

              mQENBFI3HsoBCADXDtbNJnxbPqB1vDNtCsqhe49vFYsZN9IOZsZXgp7aHjh6CJBD
              A+bGFOwyhbd7at35jQjWAw1O3cfYsKAmFy+Ar3LHCMkV3oZspJACTIgCrwnkic/9
              CUliQe324qvObU2QRtP4Fl0zWcfb/S8UYzWXWIFuJqMvE9MaRY1bwUBvzoqavLGZ
              j3SF1SPO+TB5QrHkrQHBsmX+Jda6d4Ylt8/t6CvMwgQNlrlzIO9WT+YN6zS+sqHd
              1YK/aY5qhoLNhp9G/HxhcSVCkLq8SStj1ZZ1S9juBPoXV1ZWNbxFNGwOh/NYGldD
              2kmBf3YgCqeLzHahsAEpvAm8TBa7Q9W21C8vABEBAAG0RUVsYXN0aWNzZWFyY2gg
              KEVsYXN0aWNzZWFyY2ggU2lnbmluZyBLZXkpIDxkZXZfb3BzQGVsYXN0aWNzZWFy
              Y2gub3JnPokBOAQTAQIAIgUCUjceygIbAwYLCQgHAwIGFQgCCQoLBBYCAwECHgEC
              F4AACgkQ0n1mbNiOQrRzjAgAlTUQ1mgo3nK6BGXbj4XAJvuZDG0HILiUt+pPnz75
              nsf0NWhqR4yGFlmpuctgCmTD+HzYtV9fp9qW/bwVuJCNtKXk3sdzYABY+Yl0Cez/
              7C2GuGCOlbn0luCNT9BxJnh4mC9h/cKI3y5jvZ7wavwe41teqG14V+EoFSn3NPKm
              TxcDTFrV7SmVPxCBcQze00cJhprKxkuZMPPVqpBS+JfDQtzUQD/LSFfhHj9eD+Xe
              8d7sw+XvxB2aN4gnTlRzjL1nTRp0h2/IOGkqYfIG9rWmSLNlxhB2t+c0RsjdGM4/
              eRlPWylFbVMc5pmDpItrkWSnzBfkmXL3vO2X3WvwmSFiQbkBDQRSNx7KAQgA5JUl
              zcMW5/cuyZR8alSacKqhSbvoSqqbzHKcUQZmlzNMKGTABFG1yRx9r+wa/fvqP6OT
              RzRDvVS/cycws8YX7Ddum7x8uI95b9ye1/Xy5noPEm8cD+hplnpU+PBQZJ5XJ2I+
              1l9Nixx47wPGXeClLqcdn0ayd+v+Rwf3/XUJrvccG2YZUiQ4jWZkoxsA07xx7Bj+
              Lt8/FKG7sHRFvePFU0ZS6JFx9GJqjSBbHRRkam+4emW3uWgVfZxuwcUCn1ayNgRt
              KiFv9jQrg2TIWEvzYx9tywTCxc+FFMWAlbCzi+m4WD+QUWWfDQ009U/WM0ks0Kww
              EwSk/UDuToxGnKU2dQARAQABiQEfBBgBAgAJBQJSNx7KAhsMAAoJENJ9ZmzYjkK0
              c3MIAIE9hAR20mqJWLcsxLtrRs6uNF1VrpB+4n/55QU7oxA1iVBO6IFu4qgsF12J
              TavnJ5MLaETlggXY+zDef9syTPXoQctpzcaNVDmedwo1SiL03uMoblOvWpMR/Y0j
              6rm7IgrMWUDXDPvoPGjMl2q1iTeyHkMZEyUJ8SKsaHh4jV9wp9KmC8C+9CwMukL7
              vM5w8cgvJoAwsp3Fn59AxWthN3XJYcnMfStkIuWgR7U2r+a210W6vnUxU4oN0PmM
              cursYPyeV0NX/KQeUeNMwGTFB6QHS/anRaGQewijkrYYoTNtfllxIu9XYmiBERQ/
              qPDlGRlOgVTd9xUfHFkzB52c70E=
              =92oX
              -----END PGP PUBLIC KEY BLOCK-----
            10: |
              -----BEGIN PGP PUBLIC KEY BLOCK-----
              Version: GnuPG v2.0.14 (GNU/Linux)

              mQENBFI3HsoBCADXDtbNJnxbPqB1vDNtCsqhe49vFYsZN9IOZsZXgp7aHjh6CJBD
              A+bGFOwyhbd7at35jQjWAw1O3cfYsKAmFy+Ar3LHCMkV3oZspJACTIgCrwnkic/9
              CUliQe324qvObU2QRtP4Fl0zWcfb/S8UYzWXWIFuJqMvE9MaRY1bwUBvzoqavLGZ
              j3SF1SPO+TB5QrHkrQHBsmX+Jda6d4Ylt8/t6CvMwgQNlrlzIO9WT+YN6zS+sqHd
              1YK/aY5qhoLNhp9G/HxhcSVCkLq8SStj1ZZ1S9juBPoXV1ZWNbxFNGwOh/NYGldD
              2kmBf3YgCqeLzHahsAEpvAm8TBa7Q9W21C8vABEBAAG0RUVsYXN0aWNzZWFyY2gg
              KEVsYXN0aWNzZWFyY2ggU2lnbmluZyBLZXkpIDxkZXZfb3BzQGVsYXN0aWNzZWFy
              Y2gub3JnPokBOAQTAQIAIgUCUjceygIbAwYLCQgHAwIGFQgCCQoLBBYCAwECHgEC
              F4AACgkQ0n1mbNiOQrRzjAgAlTUQ1mgo3nK6BGXbj4XAJvuZDG0HILiUt+pPnz75
              nsf0NWhqR4yGFlmpuctgCmTD+HzYtV9fp9qW/bwVuJCNtKXk3sdzYABY+Yl0Cez/
              7C2GuGCOlbn0luCNT9BxJnh4mC9h/cKI3y5jvZ7wavwe41teqG14V+EoFSn3NPKm
              TxcDTFrV7SmVPxCBcQze00cJhprKxkuZMPPVqpBS+JfDQtzUQD/LSFfhHj9eD+Xe
              8d7sw+XvxB2aN4gnTlRzjL1nTRp0h2/IOGkqYfIG9rWmSLNlxhB2t+c0RsjdGM4/
              eRlPWylFbVMc5pmDpItrkWSnzBfkmXL3vO2X3WvwmSFiQbkBDQRSNx7KAQgA5JUl
              zcMW5/cuyZR8alSacKqhSbvoSqqbzHKcUQZmlzNMKGTABFG1yRx9r+wa/fvqP6OT
              RzRDvVS/cycws8YX7Ddum7x8uI95b9ye1/Xy5noPEm8cD+hplnpU+PBQZJ5XJ2I+
              1l9Nixx47wPGXeClLqcdn0ayd+v+Rwf3/XUJrvccG2YZUiQ4jWZkoxsA07xx7Bj+
              Lt8/FKG7sHRFvePFU0ZS6JFx9GJqjSBbHRRkam+4emW3uWgVfZxuwcUCn1ayNgRt
              KiFv9jQrg2TIWEvzYx9tywTCxc+FFMWAlbCzi+m4WD+QUWWfDQ009U/WM0ks0Kww
              EwSk/UDuToxGnKU2dQARAQABiQEfBBgBAgAJBQJSNx7KAhsMAAoJENJ9ZmzYjkK0
              c3MIAIE9hAR20mqJWLcsxLtrRs6uNF1VrpB+4n/55QU7oxA1iVBO6IFu4qgsF12J
              TavnJ5MLaETlggXY+zDef9syTPXoQctpzcaNVDmedwo1SiL03uMoblOvWpMR/Y0j
              6rm7IgrMWUDXDPvoPGjMl2q1iTeyHkMZEyUJ8SKsaHh4jV9wp9KmC8C+9CwMukL7
              vM5w8cgvJoAwsp3Fn59AxWthN3XJYcnMfStkIuWgR7U2r+a210W6vnUxU4oN0PmM
              cursYPyeV0NX/KQeUeNMwGTFB6QHS/anRaGQewijkrYYoTNtfllxIu9XYmiBERQ/
              qPDlGRlOgVTd9xUfHFkzB52c70E=
              =92oX
              -----END PGP PUBLIC KEY BLOCK-----
      monodevelop:
        enabled: True
        repo:
          Ubuntu:
            1804: "deb https://download.mono-project.com/repo/ubuntu vs-bionic main"
            1604: "deb https://download.mono-project.com/repo/ubuntu vs-xenial main"
          Debian:
            9: "deb https://download.mono-project.com/repo/debian stable-stretch main"
            10: "deb https://download.mono-project.com/repo/debian stable-buster main"
        keyserver:
          Ubuntu:
            1604: "hkp://keyserver.ubuntu.com:80"
            1804: "hkp://keyserver.ubuntu.com:80"
            2004: "hkp://keyserver.ubuntu.com:80"
          Debian:
            9: "hkp://keyserver.ubuntu.com:80"
            10: "hkp://keyserver.ubuntu.com:80"
        key_id:
          Ubuntu:
            1604: "3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF"
            1804: "3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF"
            2004: "3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF"
          Debian:
            9:  "3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF"
            10: "3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF"
    aspects_packages_add_repo_yum_repos:
      elastic:
        CentOS:
          7:
            state: "absent"
            name: "elastic-7"
            description: "elastic-7.x"
            gpgkey: "https://artifacts.elastic.co/GPG-KEY-elasticsearch"
            baseurl: "https://artifacts.elastic.co/packages/7.x/yum"
            sslverify: "yes"
            repo_gpgcheck: "yes"
            gpgcheck: "yes"
            enabled: "yes"
        OracleLinux:
          7:
            state: "present"
            name: "elastic-7"
            description: "elastic-7.x"
            gpgkey: "https://artifacts.elastic.co/GPG-KEY-elasticsearch"
            baseurl: "https://artifacts.elastic.co/packages/7.x/yum"
            sslverify: "yes"
            repo_gpgcheck: "yes"
            gpgcheck: "yes"
            enabled: "yes"
    aspects_packages_snap_items:
      adguard:
        enabled: True
        state: "present"
        name: "adguard"
    aspects_packages_flatpak_remotes:
      flathub:
        enabled: True
        state: "present"
        name: "flathub"
        flatpakrepo_url: "https://flathub.org/repo/flathub.flatpakrepo"
    aspects_packages_flatpak_packages:
      firefox:
        enabled: True
        state: "present"
        name: "org.mozilla.firefox"
  roles:
  - aspects_packages
```

# License
MIT