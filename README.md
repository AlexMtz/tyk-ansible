### Overview

This playbook builds out a single server installation of tyk.  It is intended to be used to deploy tyk for development and testing purposes or as a starting point for a production multi server playbook.

The playbook should be idempotent, so it can be run multiple times to update settings and configuration as needed.

The super-user api key will be written to the users.yml to prevent rebuilding the super-user account.

The super-user api key should be removed from the users.yml file for a new installation.

### OS/Ansible versions

Currently only Ubuntu is supported. This is being tested on Ubuntu 16.04.3 LTS and Ansible 2.4.

### Server credentials

Environment variables need to be set correctly prior to the run for the username and password of the Ubuntu server:

```
export ANSIBLE_SSH_USERNAME=ubuntu
export ANSIBLE_SSH_PASSWORD=ubuntu
```

### License key
You will need to set the license key prior to the playbook run.
```
export TYK_LICENSE_KEY='eyJhbGciOiJSUzI1Ni'
```

### Bundle base url

Set the URL where tyk gateway can download middleware bundles:

```
export TYK_BUNDLE_BASE_URL='http://172.16.124.184:9999/'
```

### Server IP address

The IP address of the Ubuntu server will also need to be updated if it cannot be found in DNS.  It can be found:

```shell
➜  tyk-ansible more inventory/inventory.txt
ubuntu ansible_host=172.16.124.173


[tyk_dashboard]
ubuntu

[mongo]
ubuntu

[redis]
ubuntu
```

### Users

Two users are built, one super-admin and one admin in the default organization.  The configuration for these users is in:

```
inventory/group_vars/tyk_dashboard/users.yml
inventory/group_vars/tyk_dashboard/organisation.yml
```

### Building the example python middleware

On the server itself:
```
git clone https://github.com/TykTechnologies/tyk-plugin-demo-python
cd tyk-plugin-demo-python
/opt/tyk-gateway/utils/tyk-cli bundle build -y
python3 -m http.server
```

add it to you API https://tyk.io/docs/customise-tyk/plugins/rich-plugins/python/custom-auth-python-tutorial/#dashboard

### Site specific configuration

A role called site_config exists which may contain changes and/or software specific to the deployment

### Usage

```shell
➜  tyk-ansible git:(master) ✗ ansible-playbook -i inventory/inventory.txt site.yml

PLAY [all] *********************************************************************

TASK [install python 2] ********************************************************
ok: [ubuntu]

PLAY [mongo] *******************************************************************

TASK [setup] *******************************************************************
ok: [ubuntu]

TASK [mongodb : Import the public key as required by Ubuntu APT] ***************
changed: [ubuntu]

TASK [mongodb : Create a MongoDb source list file] *****************************
changed: [ubuntu]

TASK [mongodb : Reload the package database] ***********************************
ok: [ubuntu]

TASK [mongodb : Run the install script] ****************************************
changed: [ubuntu]

TASK [mongodb : Copy the mongo.service file] ***********************************
changed: [ubuntu]

TASK [mongodb : Start mongodb] *************************************************
changed: [ubuntu]

PLAY [redis] *******************************************************************

TASK [setup] *******************************************************************
ok: [ubuntu]

TASK [redis : Install Redis] ***************************************************
changed: [ubuntu]

TASK [redis : Start redis] *****************************************************
ok: [ubuntu]

PLAY [tyk_dashboard] ***********************************************************

TASK [setup] *******************************************************************
ok: [ubuntu]

TASK [tyk_dashboard : Add tyk GPG key which signs tyk binaries] ****************
changed: [ubuntu]

TASK [tyk_dashboard : Reload the package database] *****************************
ok: [ubuntu]

TASK [tyk_dashboard : Make sure APT supports HTTPS] ****************************
ok: [ubuntu]

TASK [tyk_dashboard : Add the required repose for tyk dashboard] ***************
changed: [ubuntu]

TASK [tyk_dashboard : Add the required repose tyk dashboard] *******************
changed: [ubuntu]

TASK [tyk_dashboard : Reload the package database] *****************************
ok: [ubuntu]

TASK [tyk_dashboard : Install the Tyk Dashboard] *******************************
changed: [ubuntu]

TASK [tyk_dashboard : Write out the final conf file] ***************************
changed: [ubuntu]

TASK [tyk_dashboard : Restart Tyk Dashboard] ***********************************
changed: [ubuntu]

TASK [tyk_dashboard_users : Creating super-user] *******************************
ok: [ubuntu]

TASK [tyk_dashboard_users : Setting admin password] ****************************
ok: [ubuntu]

TASK [tyk_dashboard_users : Write the admin key to the vars file so we don\'t recreate the admin user] ***
changed: [ubuntu -> localhost]

TASK [tyk_dashboard_users : Set a fact for the super-user key] *****************
ok: [ubuntu]

TASK [tyk_dashboard_users : Load the exisiting organisations] ******************
ok: [ubuntu]

TASK [tyk_dashboard_users : Set a fact for the matching organisations] *********
ok: [ubuntu]

TASK [tyk_dashboard_users : Get the ID for the exisitng organisation] **********
skipping: [ubuntu]

TASK [tyk_dashboard_users : Creating Organisation] *****************************
ok: [ubuntu]

TASK [tyk_dashboard_users : Retrieve the exisiting users] **********************
ok: [ubuntu]

TASK [tyk_dashboard_users : Set a fact for the matching users] *****************
ok: [ubuntu]

TASK [tyk_dashboard_users : Creating User in organization] *********************
ok: [ubuntu]

TASK [tyk_dashboard_users : Setting user password] *****************************
ok: [ubuntu]

PLAY [tyk_pump] ****************************************************************

TASK [setup] *******************************************************************
ok: [ubuntu]

TASK [tyk_pump : Add tyk GPG key which signs tyk binaries] *********************
ok: [ubuntu]

TASK [tyk_pump : Reload the package database] **********************************
ok: [ubuntu]

TASK [tyk_pump : Make sure APT supports HTTPS] *********************************
ok: [ubuntu]

TASK [tyk_pump : Add the required repose for tyk pump] *************************
changed: [ubuntu]

TASK [tyk_pump : Add the required repose for tyk pump] *************************
changed: [ubuntu]

TASK [tyk_pump : Reload the package database] **********************************
ok: [ubuntu]

TASK [tyk_pump : Install Tyk pump] *********************************************
changed: [ubuntu]

TASK [tyk_pump : Write out the final conf file] ********************************
changed: [ubuntu]

TASK [tyk_pump : Restart Tyk pump] *********************************************
changed: [ubuntu]

PLAY [tyk_gateway_with_dashboard] **********************************************

TASK [setup] *******************************************************************
ok: [ubuntu]

TASK [tyk_gateway_with_dashboard : Add tyk GPG key which signs tyk binaries] ***
ok: [ubuntu]

TASK [tyk_gateway_with_dashboard : Reload the package database] ****************
ok: [ubuntu]

TASK [tyk_gateway_with_dashboard : Make sure APT supports HTTPS] ***************
ok: [ubuntu]

TASK [tyk_gateway_with_dashboard : Add the required repose for tyk gateway] ****
changed: [ubuntu]

TASK [tyk_gateway_with_dashboard : Add the required repose for tyk gateway] ****
changed: [ubuntu]

TASK [tyk_gateway_with_dashboard : Reload the package database] ****************
ok: [ubuntu]

TASK [tyk_gateway_with_dashboard : Install Tyk gateway] ************************
changed: [ubuntu]

TASK [tyk_gateway_with_dashboard : Write out the final conf file] **************
changed: [ubuntu]

TASK [tyk_gateway_with_dashboard : Restart Tyk gateway] ************************
changed: [ubuntu]

PLAY RECAP *********************************************************************
ubuntu                     : ok=52   changed=23   unreachable=0    failed=0

➜  tyk-ansible git:(master) ✗
```
