Jupyterhub
===============

Adapted from [ansible-jupyterhub](https://github.com/jenkstom/ansible-jupyterhub) by Tom White <http://k5ehx.net>. 
This particular role is meant for provisioning RHEL/CentOS 7 hosts and provides support for:
    - jupyterhub
    - authentication via PAM
    - Sudo and Docker Spawners
    - jupyterlab

Dependencies
------------

`Python 3.5+` is required. Also for the DockerSpawner, docker (version 17+) is required as well.
Check the project datalab - https://github.com/natbusa/datalab for a vagrant/ansible setup, using the above configuration.

Role Variables
--------------

The only role vars that the user needs to worry about are:

- `nossl: False`: if set to True, will enable https for the jupyterhub proxy
- `jupyterhub_spawner: sudospawner`: Select either `imagespawner` or `dockerspawner` or `sudospawner` or `sudospawner-lab`
- `jupyterhub_spawner_docker_image: jupyterhub/systemuser`: select which image you would like to start up in `dockerspawner` mode
- `generate_testusers: False`: will generate test users. Test users will be assigned to the `jupyter` group.

Spawners
------------

*dockerspawner* will use the `dockerspawner.SystemUserSpawner` and match the host username and uid, with the one in the container.
It will also map the container home dir with the home dir in the host. Depending on the setting of `install_jupyterlab` it will start `jupyter-singleuser` running either the notebook or the lab webapp. 

*imagespawner* same as *dockerspawner* but will allow you to set a list of images to be selected at run time by each user.

*sudospawner* will spawn a jupyter-singleuser process in the host process space.

*sudospawner-lab* same as the *sudospawner* but starts lab by default.

Pro and cons:

sudospawner does not require docker and is lighter in the integration/configuration. 
However, it will need the given Python/R packages to be installed in the host. 
This means heavier provisioning. This approach can potentially pollute the host will conflicting Python/R packages.

dockerspawner is a more indirect/complex in the setup and require an extra level of indirection.
However, Python and R packages are contained in the image and the host is only responsible for the proxy/auth/spawn logistics
It also separate the versioning of R/Python from those setup in the host environment

Examples Playbook
----------------

docker spawner with four test users:

```
    - name: Install jupyterhub
      hosts: localhost
      sudo: yes
      
    - role: centos-jupyterhub
        jupyterhub_spawner: 'dockerspawner'
        jupyterhub_spawner_docker_image: 'natbusa/jupyterhub-scipy-lab'
        generate_testusers: True
        gen_test_username:
          - amy
          - joe
          - sue
          - bob
```

docker spawner with four test users:

```
    - name: Install jupyterhub
      hosts: localhost
      sudo: yes
      
    - role: centos-jupyterhub
      jupyterhub_spawner: 'imagespawner'
      jupyterhub_spawner_docker_image:
        - 'natbusa/jupyterhub-scipy-lab'
        - 'natbusa/jupyterhub-datascience-lab'
        - 'natbusa/jupyterhub-tensorflow-lab'
        - 'natbusa/jupyterhub-all-spark-lab'
      generate_testusers: True
      gen_test_username:
        - amy
        - joe
        - sue
        - bob
```

About the stacks for dockerspawer and imagespawner:

This setup uses the systemuser dockerspawner configuration. 
At the moment this particular script is not part of the jupyter/docker-stacks.
  
avaiable docker images for this setup:

- 'jupyterhub/systemuser'
- 'natbusa/jupyterhub-scipy-lab'
- 'natbusa/jupyterhub-datascience-lab'
- 'natbusa/jupyterhub-tensorflow-lab'
- 'natbusa/jupyterhub-all-spark-lab'    

License
-------

MIT

Author Information
------------------

Tom White <http://k5ehx.net>
Natalino Busa <natalino.busa@gmail.com>
