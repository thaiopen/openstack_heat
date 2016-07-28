================
Heat - OpenStack
================

Lab
***
Log in as admin, go to ``Project > Access & Security > Download Openstack RC file V3``

.. image:: _images/rc01.png

.. image:: _images/rc02.png

copy file and save to ``keystonerc_admin`` in /root

Create Project
==============
create 2 project Admin and Demo for testing.

Admin project
-------------
::

  source keystonerc_admin
  openstack project create --description "Admin Project" admin
  openstack user create --password-prompt admin
  openstack role create admin
  openstack role add --project admin --user admin admin

Demo project
------------
::

  openstack project create --description "Demo Project" demo
  +-------------+----------------------------------+
  | Field       | Value                            |
  +-------------+----------------------------------+
  | description | Demo Project                     |
  | enabled     | True                             |
  | id          | f47553804d5d48529eea395c29da941d |
  | name        | demo                             |
  +-------------+----------------------------------+

  openstack user create --password-prompt demo
  User Password:
  Repeat User Password:
  +----------+----------------------------------+
  | Field    | Value                            |
  +----------+----------------------------------+
  | email    | None                             |
  | enabled  | True                             |
  | id       | 1f119268b8e2437a8ce94f951c3ae116 |
  | name     | demo                             |
  | username | demo                             |
  +----------+----------------------------------+

  openstack role create user
  +-----------+----------------------------------+
  | Field     | Value                            |
  +-----------+----------------------------------+
  | domain_id | None                             |
  | id        | e718c008eb8343ada2f342c2ab986e00 |
  | name      | user                             |
  +-----------+----------------------------------+

  openstack role add --project demo --user demo user
  +-----------+----------------------------------+
  | Field     | Value                            |
  +-----------+----------------------------------+
  | domain_id | None                             |
  | id        | e718c008eb8343ada2f342c2ab986e00 |
  | name      | user                             |
  +-----------+----------------------------------+


Prepare for Test
================
Image & flover & key
--------------------
Download and create image::

  curl -O http://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2
  glance image-create --name='centos7_image' --visibility=public --container-format=bare --disk-format=qcow2 --file=CentOS-7-x86_64-GenericCloud.qcow2

  +------------------+--------------------------------------+
  | Property         | Value                                |
  +------------------+--------------------------------------+
  | checksum         | 9a9aa58ee594ed16032885df51beb7f3     |
  | container_format | bare                                 |
  | created_at       | 2016-07-27T10:31:15Z                 |
  | disk_format      | qcow2                                |
  | id               | a508d524-59c8-46da-9056-a3ec60d18194 |
  | min_disk         | 0                                    |
  | min_ram          | 0                                    |
  | name             | centos7_image                        |
  | owner            | adbe784d265b48eead84e0681e2f0beb     |
  | protected        | False                                |
  | size             | 217433246                            |
  | status           | active                               |
  | tags             | []                                   |
  | updated_at       | 2016-07-27T10:45:27Z                 |
  | virtual_size     | None                                 |
  | visibility       | public                               |
  +------------------+--------------------------------------+

  openstack image list
  +--------------------------------------+---------------+--------+
  | ID                                   | Name          | Status |
  +--------------------------------------+---------------+--------+
  | a508d524-59c8-46da-9056-a3ec60d18194 | centos7_image | active |
  | 60614167-dfec-4e5b-be5d-1b5217c53926 | cirros image  | active |
  +--------------------------------------+---------------+--------+


  openstack flavor list
  +----+-----------+-------+------+-----------+-------+-----------+
  | ID | Name      |   RAM | Disk | Ephemeral | VCPUs | Is Public |
  +----+-----------+-------+------+-----------+-------+-----------+
  | 1  | m1.tiny   |   512 |    1 |         0 |     1 | True      |
  | 2  | m1.small  |  2048 |   20 |         0 |     1 | True      |
  | 3  | m1.medium |  4096 |   40 |         0 |     2 | True      |
  | 4  | m1.large  |  8192 |   80 |         0 |     4 | True      |
  | 5  | m1.xlarge | 16384 |  160 |         0 |     8 | True      |
  +----+-----------+-------+------+-----------+-------+-----------+

Create Key::

  openstack keypair create heat_key > heat_key.priv
  chmod 600 heat_key.priv

   cat heat_key.priv
  -----BEGIN RSA PRIVATE KEY-----
  MIIEqQIBAAKCAQEA2gBpygu1UuokIGQhorBlmx6aHor0XvHHIos4MOAm05LtBUE9
  l4yMYZuuM2y/2w1I0zc48JyJrbE94Itk8zLk6jGk3qOlt3D2lzh6KtmS1HmIaP9N
  urPq4TYF3fkJ72S9x7Q4M0hmEKusYOpGvzVqlXzkGVX9gqW14PKKqb5bwsdqRyYB
  PVBLFjKfU5T7zls/lkG0POhhlurJDtjflAfwuZj55sTsTkmGnE6Q5dI41sg+R/K2
  EskvX1nk/OVbXaAQEamxrgKiOqBa5BESkHGRCMo9HEWryI6J+5oLJxP1P6Svn30o
  quQVID7ijQwenUCmy4Cg/F0qXdCcWKPKMVT+MwIDAQABAoIBAC0rRsAZWvGaF/Aq
  LxnxaKwSVCad7fI6MGOE6xidaQ4PdFLRaLyc331d72MyVjJB92PkHkUkehZmEgcT
  etA1LF8l+B41VCHFIyLEDfQQ7q5dnBj9RRK5MqKFOgP7FMboXbYiLj7Z8mPj3iya
  fhwiKjpdDTn+qi9GvH+mYXq4OXUK0InSBdQYGcVX7QzalY/6SVx0iydsWoDtKJ4Z
  G9GxniVsyhfK/2qxX7j+eEhIDLkYU3S3dxIRFbkaQSXrTy2WAN+ifEyH3xFE3spg
  NzdUMx0X0dyXPACE64A7125v/rz6UGjXUV35Fr2mGyJz0z6TPwkmqyZ/FLux99tv
  urHlIpkCggCBAOIYzdwIX4XxFTcmGPpY5mk8n7y8ZMFpzXD0wZ+MgcoDwk7ZfKz/
  bN++nJCo1yD1LpebTebll1Ntt09TYSajq4WI/cI3RBpjAVc2eB4CPQO9HLWrXJJL
  V7zh1i0Q3o7AytB+OrYAt/q++kd4loXs5LAoUDf1ds6wyy2tXxkTTPMvAoIAgQD2
  1YTaAFyg67IIovsfByvaM3BkSEk4ufzoSDWkYr5kQ2xPh7p4s4n5pBjMli9tmfpk
  do6a2bZcBnLmzPg5KKK7Guefxf9fWBpiijrU7emMO/5jjoa3HuVR1tTve4O/pnDM
  bNhNuhgbFRswzAwQqNi8zxVoWFuUWeL/OTLkep60PQKCAIAArDg4lmHKYTxB7MAY
  Rplw6LhE9H6MQMdWtbO5yM4+DqLOX0FKtnvZcpS+rXHubCxWUST7MeWwXtwzxTi9
  x7Dj9dNjd4yhU3gWjVfwJuuNjkt+kuL0oHrDWhysB6fR3ur7kdSU5A2oFldiTQ6N
  mXXYvFX4xKc02DvSh/iHd5I8CwKCAIEAmirQWGzjberofrZ1DHMpM5bvOuNfQFwC
  F36Kaor2CZ+G9cF7eUoCAnc4ugePxe/nUXovJY6XdLDecUNSQjkcMqriQoujLgTU
  LfnSNw4F0f3irIHnqFZWup/ozU8jAb/OMhBFpX528etj2T2Ix5r+DCd3Yn3VbTob
  KneqTsQ7fa0CggCBAKK8Uq4wcV54gkcFL9LNt3JR2MJyoCZ53Z9U3gCM8K2737Q1
  7fwJ/OxD0M41rmUG16wUe3Df7BglRC89J4VTDlZr/4QPSoPwVj81lxMqbwDXy+v/
  GQlexZdvZeMcPYXOM3OZf5wbdndujmk92baxLDyXNVpcnjFFTxDsXaFZwTzO
  -----END RSA PRIVATE KEY-----

Create provider Network
-----------------------
before create provider network (Admin Privileges) for external network, We must
check config below ``bridge_mappings`` that we will user in ``--provider:physical_network ext-net``
::

  vim /etc/neutron/plugins/ml2/openvswitch_agent.ini
  217 bridge_mappings =ext-net:br-ex

create network (L2 segment)::

  neutron net-create external_network --provider:network_type flat --provider:physical_network ext-net  --router:external
  Created a new network:
  +---------------------------+--------------------------------------+
  | Field                     | Value                                |
  +---------------------------+--------------------------------------+
  | admin_state_up            | True                                 |
  | availability_zone_hints   |                                      |
  | availability_zones        |                                      |
  | created_at                | 2016-07-27T11:17:07                  |
  | description               |                                      |
  | id                        | 59e6dd3e-68b4-49ed-863d-84651330ba65 |
  | ipv4_address_scope        |                                      |
  | ipv6_address_scope        |                                      |
  | is_default                | False                                |
  | mtu                       | 1500                                 |
  | name                      | external_network                     |
  | provider:network_type     | flat                                 |
  | provider:physical_network | ext-net                              |
  | provider:segmentation_id  |                                      |
  | router:external           | True                                 |
  | shared                    | False                                |
  | status                    | ACTIVE                               |
  | subnets                   |                                      |
  | tags                      |                                      |
  | tenant_id                 | adbe784d265b48eead84e0681e2f0beb     |
  | updated_at                | 2016-07-27T11:17:07                  |
  +---------------------------+--------------------------------------+

Verify from UI
.. image:: _images/heatlab01.png

Edit Network to share

.. image:: _images/heatlab02.png

Verify Network info::

  neutron net-show 59e6dd3e-68b4-49ed-863d-84651330ba65
  +---------------------------+--------------------------------------+
  | Field                     | Value                                |
  +---------------------------+--------------------------------------+
  | admin_state_up            | True                                 |
  | availability_zone_hints   |                                      |
  | availability_zones        |                                      |
  | created_at                | 2016-07-27T11:17:07                  |
  | description               |                                      |
  | id                        | 59e6dd3e-68b4-49ed-863d-84651330ba65 |
  | ipv4_address_scope        |                                      |
  | ipv6_address_scope        |                                      |
  | is_default                | False                                |
  | mtu                       | 1500                                 |
  | name                      | external_network                     |
  | provider:network_type     | flat                                 |
  | provider:physical_network | ext-net                              |
  | provider:segmentation_id  |                                      |
  | router:external           | True                                 |
  | shared                    | True                                 |
  | status                    | ACTIVE                               |
  | subnets                   |                                      |
  | tags                      |                                      |
  | tenant_id                 | adbe784d265b48eead84e0681e2f0beb     |
  | updated_at                | 2016-07-27T11:36:04                  |
  +---------------------------+--------------------------------------+

or verify from ui

.. image:: _images/heatlab03.png

.. image:: _images/heatlab04.png

create sub network (L3 segment)
-------------------------------
check subnet of provider network::

  ip -4 a s br-ex
  10: br-ex: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN
      inet 192.168.56.200/24 brd 192.168.56.255 scope global br-ex
         valid_lft forever preferred_lft forever

create subnet no dhcp and pool in ``192.168.56.0/24``
::

  neutron subnet-create --name public_subnet --enable_dhcp=False --allocation-pool=start=192.168.56.210,end=192.168.56.220 \
  --dns-nameserver 8.8.8.8  --gateway=192.168.56.1 external_network 192.168.56.0/24

  Created a new subnet:
  +-------------------+------------------------------------------------------+
  | Field             | Value                                                |
  +-------------------+------------------------------------------------------+
  | allocation_pools  | {"start": "192.168.56.210", "end": "192.168.56.220"} |
  | cidr              | 192.168.56.0/24                                      |
  | created_at        | 2016-07-27T11:54:34                                  |
  | description       |                                                      |
  | dns_nameservers   | 8.8.8.8                                              |
  | enable_dhcp       | False                                                |
  | gateway_ip        | 192.168.56.1                                         |
  | host_routes       |                                                      |
  | id                | 93cd697a-c480-42bf-9d46-ed45dea185bb                 |
  | ip_version        | 4                                                    |
  | ipv6_address_mode |                                                      |
  | ipv6_ra_mode      |                                                      |
  | name              | public_subnet                                        |
  | network_id        | 59e6dd3e-68b4-49ed-863d-84651330ba65                 |
  | subnetpool_id     |                                                      |
  | tenant_id         | adbe784d265b48eead84e0681e2f0beb                     |
  | updated_at        | 2016-07-27T11:54:34                                  |
  +-------------------+------------------------------------------------------+

verify from ui

.. image:: _images/heatlab05.png

.. image:: _images/heatlab06.png

From now change user to demo user. we need credential file like admin have. we will
get it from  horizon

.. image:: _images/demo01.png

create demorc_v3 by goto ``Project > Access & Security > Download Openstack RC File V3``

.. image:: _images/demo02.png

::

  vim  demorc_v3
  source demorc_v3
  Please enter your OpenStack Password:

create Tenent Network or Private network (user privileges)::

  source source demorc_v3

  neutron net-create private
  Created a new network:
  +-------------------------+--------------------------------------+
  | Field                   | Value                                |
  +-------------------------+--------------------------------------+
  | admin_state_up          | True                                 |
  | availability_zone_hints |                                      |
  | availability_zones      |                                      |
  | created_at              | 2016-07-27T12:26:08                  |
  | description             |                                      |
  | id                      | bd8b7de6-c013-43a6-b32a-08747d467e8a |
  | ipv4_address_scope      |                                      |
  | ipv6_address_scope      |                                      |
  | mtu                     | 1500                                 |
  | name                    | private                              |
  | router:external         | False                                |
  | shared                  | False                                |
  | status                  | ACTIVE                               |
  | subnets                 |                                      |
  | tags                    |                                      |
  | tenant_id               | f47553804d5d48529eea395c29da941d     |
  | updated_at              | 2016-07-27T12:26:08                  |
  +-------------------------+--------------------------------------+

Non-privileged users typically cannot supply additional parameters to this command.
The service automatically chooses paramters using information from the following files:

ml2_conf.ini::

  vim /etc/neutron/plugins/ml2/ml2_conf.ini

  [ml2]
  type_drivers = vlan,vxlan,gre,flat,local
  tenant_network_types = local,vlan,gre,vxlan

  [ml2_type_vlan]
  network_vlan_ranges =physnet2:1:1000

  [ml2_type_vxlan]
  vni_ranges =10:100

create subnet::

  neutron subnet-create private 172.16.1.0/24 --name private_subnet \
  --dns-nameserver 8.8.4.4 --gateway 172.16.1.1

  Created a new subnet:
  +-------------------+------------------------------------------------+
  | Field             | Value                                          |
  +-------------------+------------------------------------------------+
  | allocation_pools  | {"start": "172.16.1.2", "end": "172.16.1.254"} |
  | cidr              | 172.16.1.0/24                                  |
  | created_at        | 2016-07-27T12:33:58                            |
  | description       |                                                |
  | dns_nameservers   | 8.8.4.4                                        |
  | enable_dhcp       | True                                           |
  | gateway_ip        | 172.16.1.1                                     |
  | host_routes       |                                                |
  | id                | 81c13ec9-b2c2-4a7b-b98e-9fb5502824c0           |
  | ip_version        | 4                                              |
  | ipv6_address_mode |                                                |
  | ipv6_ra_mode      |                                                |
  | name              | private_subnet                                 |
  | network_id        | bd8b7de6-c013-43a6-b32a-08747d467e8a           |
  | subnetpool_id     |                                                |
  | tenant_id         | f47553804d5d48529eea395c29da941d               |
  | updated_at        | 2016-07-27T12:33:58                            |
  +-------------------+------------------------------------------------+

verify ui

.. image:: _images/demo03.png

.. image:: _images/demo04.png

Network type of Tenent network
------------------------------
from value ``tenant_network_types = local,vlan,gre,vxlan`` openstack choose first
option is list is local for tenent network.

.. important:: only admin can change Network type

.. image:: _images/demo05.png

Definition
----------

.. image:: _images/demo06.png
