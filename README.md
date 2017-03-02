# configuration management for MySQL Galera cluster

This repository is based on the good work of the following repository [https://raw.githubusercontent.com/adfinis-sygroup/mariadb-ansible-galera-cluster](https://raw.githubusercontent.com/adfinis-sygroup/mariadb-ansible-galera-cluste)

I have taken the structure of that repository and modified it for MySQL instead of MariaDB.

## Installing pre-requisites

Run the following to get the Ansible Pre-Requisite Roles

    ansible-galaxy -r ansible/requirements.yml install

## Enable Virtual Environment 

Run the following to start the Virtual Environment

    vagrant up

## Installing requirements

To install the required packages and configure SELinux and the firewall you can
run only the tasks tagged with ``setup``

    ansible-playbook -i ansible/inventory ansible/galera.yml --tags setup

## Configure MySQL Galera cluster

To run all further tasks to configure MySQL Galera cluster and add the
required user for the **S**tate **S**napshot **T**ransfer (SST) either skip the
tasks tagged ``setup`` or run the tags ``config`` or ``auth`` directly.

    ansible-playbook -i ansible/inventory ansible/galera.yml --skip-tags setup

## Bootstrapping MySQL Galera cluster

Bootstrapping the cluster can be done using a playbook dedicated for
bootstrapping the MariaDB Galera cluster called galera_bootstrap.yml

    ansible-playbook -i ansible/inventory ansible/galera_bootstrap.yml

It uses the first node in the group ``galera_cluster`` to bootstrap the cluster.

## Rolling MySQL Galera cluster updates

The role ``galera_conf`` is set up to allow rolling updates of the configuration
of the Galera cluster. When the variable ``bootstrapped`` is set to ``yes`` the
``mariadb`` service is restarted after a change of the configuration. It is
important to note that the playbook contains the keyword ``serial: 1``, meaning
that the configuration is applied one node at a time so that the will never lose
quorum in the process of applying the new configuration.

    ansible-playbook -i ansible/inventory ansible/galera_rolling_update.yml

## Testing

* on `galera1` open a mysql prompt and issue the following

  ``` sql
  create database test;
  use test;
  CREATE TABLE loltest (i INT);
  INSERT INTO loltest () VALUES (1);
  select * from loltest;
  ```

  Should return

  ``` sql
  +------+
  | i    |
  +------+
  |    1 |
  +------+
  ```

* on `galera2` open a mysql prompt and issue the following

  ``` sql
  use test;
  INSERT INTO loltest () VALUES (1);
  select * from loltest;
  ```

  Should return

  ``` sql
  +------+
  | i    |
  +------+
  |    1 |
  |    2 |
  +------+
  ```

* on `galera3` open a mysql prompt and issue the following

  ``` sql
  use test;
  INSERT INTO loltest () VALUES (3);
  select * from loltest;
  ```

  Should return

  ``` sql
  +------+
  | i    |
  +------+
  |    1 |
  |    2 |
  |    3 |
  +------+
  ```

* on `galera1` and `galera2` open a mysql prompt and issue the following

  ``` sql
  use test;
  select * from loltest;
  ```

  Should return

  ``` sql
  +------+
  | i    |
  +------+
  |    1 |
  |    2 |
  |    3 |
  +------+
  ```