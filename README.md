[![Docker Stars](https://img.shields.io/docker/stars/fiercely/centos7.svg)](https://hub.docker.com/r/fiercely/centos7/)[![Docker Stars](https://img.shields.io/docker/pulls/fiercely/centos7.svg)](https://hub.docker.com/r/fiercely/centos7/)

## Centos Systemd enabled image to be used with Molecule for instance

This repo contains an image in Centos7 with systemd enabled in order to be used with molecule

## Usage with molecule

First initiate a molecule scenario in a given ansible role root folder

```sh
    molecule init scenario --scenario-name fiercely-systemd -r <my_awesome_ansible_role>
```
This uses the default Docker driver for molecule

Now under

        <my_awesome_ansible_role>/molecule/fiercely-systemd/molecule.yml

Change its `platform` contents to:

```yml
platforms:
  - name: instance
    image: fiercely/centos7:systemd
    privileged: True
    volume_mounts:
      - "/sys/fs/cgroup:/sys/fs/cgroup:rw"
    command: "/usr/sbin/init"
```

And there you have it, you are now able to use a systemd role.

Test example

Under

        <my_awesome_ansible_role>/molecule/fiercely-systemd

Create a requirements.yml file

With the following contents


```yml
- src: resmo.ntp
  version: 0.4.0
```

And under the same directory, change `playbook.yml` to:

```yml
---
- name: Converge
  hosts: all
  roles:
    - role: resmo.ntp
```

Now from the root folder of your role
```sh
    molecule test
```

Resulting in

```yml


--> Scenario: 'default'
--> Action: 'create'

    PLAY [Create] ******************************************************************

    TASK [Log into a Docker registry] **********************************************
    skipping: [localhost] => (item=None)

    TASK [Create Dockerfiles from image names] *************************************
    changed: [localhost] => (item=None)

    TASK [Discover local Docker images] ********************************************
    ok: [localhost] => (item=None)

    TASK [Build an Ansible compatible image] ***************************************
    changed: [localhost] => (item=None)

    TASK [Create docker network(s)] ************************************************

    TASK [Create molecule instance(s)] *********************************************
    changed: [localhost] => (item=None)

    TASK [Wait for instance(s) creation to complete] *******************************
    changed: [localhost] => (item=None)

    PLAY RECAP *********************************************************************
    localhost                  : ok=5    changed=4    unreachable=0    failed=0


--> Scenario: 'default'
--> Action: 'prepare'
Skipping, prepare playbook not configured.
--> Scenario: 'default'
--> Action: 'converge'

    PLAY [Converge] ****************************************************************

    TASK [Gathering Facts] *********************************************************
    ok: [instance]

    TASK [resmo.ntp : Add the OS specific variables] *******************************
    ok: [instance]

    TASK [resmo.ntp : Install the required packages in Redhat derivatives] *********
    changed: [instance]

    TASK [resmo.ntp : Install the required packages in Debian derivatives] *********
    skipping: [instance]

    TASK [resmo.ntp : Copy the ntp.conf template file] *****************************
    changed: [instance]

    TASK [resmo.ntp : Start/stop ntp service] **************************************
 [WARNING]: Ignoring "pattern" as it is not used in "systemd"

    changed: [instance]

    RUNNING HANDLER [resmo.ntp : restart ntp] **************************************
    changed: [instance]

    PLAY RECAP *********************************************************************
    instance                   : ok=6    changed=4    unreachable=0    failed=0


--> Scenario: 'default'
--> Action: 'idempotence'
Idempotence completed successfully.
--> Scenario: 'default'
--> Action: 'side_effect'
Skipping, side effect playbook not configured.
--> Scenario: 'default'
--> Action: 'verify'
--> Executing Testinfra tests found in /CENSORED/tests/...
    ============================= test session starts ==============================
    platform linux2 -- Python 2.7.15rc1, pytest-3.6.2, py-1.5.3, pluggy-0.6.0
    rootdir: /CENSORED/molecule/default, inifile:
    plugins: testinfra-1.12.0
collected 1 item

    tests/test_default.py .                                                  [100%]

    =========================== 1 passed in 5.62 seconds ===========================
Verifier completed successfully.
--> Scenario: 'default'
--> Action: 'destroy'

    PLAY [Destroy] *****************************************************************

    TASK [Destroy molecule instance(s)] ********************************************
    changed: [localhost] => (item=None)

    TASK [Wait for instance(s) deletion to complete] *******************************
    changed: [localhost] => (item=None)

    TASK [Delete docker network(s)] ************************************************

    PLAY RECAP *********************************************************************
    localhost                  : ok=2    changed=2    unreachable=0    failed=0
```
