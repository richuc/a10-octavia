# A10 Networks Openstack Octavia Driver
===========================================

This is currently in development. Official release slated for Nov 6 2019.
As this is currently in development, support is limited until official release. 

A10 Networks Octavia Driver for Thunder, vThunder and AX Series Appliances 
Supported releases:

* OpenStack: Rocky Releases
* Octavie versions: v2
* ACOS versions: ACOS 2/AxAPI 2.1 (ACOS 2.7.2+), ACOS 4/AxAPI 3.0 (ACOS 4.0.1-GA +)

## STEP1: Installation

Clone the repository and run the following command to install

### Register the A10 driver and plugin
`sudo python ./setup.py install`

Clone the acos client and install it from https://github.com/a10networks/acos-client

### Register acos clint by running following command in acos-client folder

`sudo python ./setup.py install`

## STEP2: Upload vThunder image and create vThunder flavor

Upload provided vThunder image in QCOW2 and create OpenStack flavor with minimum 8 vcpus, 8GB RAM and 30GB disk as admin user of OpenStack environment.

Use below commands for reference:

```shell
openstack image create --disk-format qcow2 --container-format bare   --public --file vThunder410.qcow2 vThunder.qcow2

openstack flavor create --vcpu 8 --ram 8196 --disk 30 vThunder_flavor
```

Note down the `image ID` and `flavor ID` of created resources.

## STEP3: Update the Octavia config file

Update the /etc/octavia/octavia.conf file with the following parameters:

```shell
octavia_plugins = a10_hot_plug_plugin

enabled_provider_drivers = a10:     'The A10 Octavia driver.',
                           noop_driver: 'The no-op driver.',
                           amphora: 'The Octavia Amphora driver.',
                           octavia: 'Deprecated alias of the Octavia Amphora driver.'

default_provider_driver = a10
```

In `[controller_worker]` section add following entries:

```
amp_flavor_id = [`ID of vThunder flavor`]
amp_image_id = [`ID of vThunder Image`]
```
comment any `amp_image_tag` entry if exists.

## STEP4: Add A10 config file 

create `/etc/a10/config.py` with proper access for octavia/stack user.
add following entries as default.

```shell
DEFAULT_VTHUNDER_USERNAME = "admin"
DEFAULT_VTHUNDER_PASSWORD = "a10"
DEFAULT_AXAPI_VERSION = "30"
```

## STEP5: Run database migrations

from `a10-octavia/a10_octavia/db/migrations` folder run 

```shell
alembic upgrade head
```

if older migrations not found, trucate `alembic_migrations` table from ocatvia database and re-run the above command.

## STEP6: Allow security group to access vThunder AXAPIs port

As `admin` OpenStack user, update security group `lb-mgmt-sec-grp` and allow `PORT 80` and `PORT 443` ingress traffic to allow AXAPI communication with vThunder instances.

## STEP7: Restart Related Octavia Services
### devstack development environment
`sudo systemctl restart devstack@o-api.service devstack@o-cw.service devstack@o-hk.service devstack@o-hm.service devstack@q-svc.service`

## other environments
Use `systemctl` or similar function to restart Octavia controller and health services. 
