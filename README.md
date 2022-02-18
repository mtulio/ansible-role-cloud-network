cloud-vpc
=========

[![Project Status: Concept - initial development is in progress, but there has not yet been a stable, usable release suitable for the public.](http://www.repostatus.org/badges/latest/wip.svg)](http://www.repostatus.org/#wip)

Ansible role to manage all resources in Virtual Private Cloud Infrastructure.
from service provider - now we are supporting AWS (please help us to improve =] ).

Roadmap for Cloud Providers:
* AWS - Amazon Web Services
* GCE - Google Cloud

Main roles to manage:
* Facts
* VPC
* Subnets
* Internet Gateway
* Route tables

Requirements
------------

* ansible >= 2.3

* AWS

1. boto3

* GCE

> TODO

Role Variables
--------------

> TODO

Dependencies
------------

None

Example Playbook
----------------

* Setup Cloud Network configuration:

      - hosts: servers
        vars:
          ---
          #########################
          # Networks
          # 172.16-31.0.0/12
          #
          ## AWS
          ## 172.16.0-255.0/24
          ### us-east-1: 172.16.0-31.0/19
          ### us-east-2: 172.16.32-63.0/19
          ### ---------: 172.16.64-95.0/19
          ### ---------: 172.16.95-127.0/19
          ### ---------: 172.16.128-159.0/19
          ### ---------: 172.16.160-191.0/19
          ### ---------: 172.16.192-223.0/19
          ### ---------: 172.16.224-255.0/19
          ## GCLOUD
          ### 172.17.0-255.0/24
          #########################
          networks:
          - name: vpc_use1_aws
            block: 172.16.0.0/19
            provider: aws
            region: us-east-1
            igw: yes
            nat_gw: no
            nat_gw_subnet: net_natgw_public
            security_groups: "{{ security_groups | d([]) }}"
            routes:
              - name: rt_private
                table:
                  - dest: 0.0.0.0/0
                    gateway_id: natgw
              - name: rt_public
                table:
                  - dest: 0.0.0.0/0
                    gateway_id: igw
              # - name: rt_natgw
              #   table:
              #     - dest: 0.0.0.0/0
              #       gateway_id: igw
            subnets:
              - name: net_nodes_public_1
                az: us-east-1b
                cidr: 172.16.0.0/24
                route: rt_public
                public_ip: true
              - name: net_nodes_private_1
                az: us-east-1b
                cidr: 172.16.1.0/24
                route: rt_private
              - name: net_nodes_public_2
                az: us-east-1c
                cidr: 172.16.2.0/24
                route: rt_public
                public_ip: true
              - name: net_nodes_private_2
                az: us-east-1c
                cidr: 172.16.3.0/24
                route: rt_private
              - name: net_nodes_public_3
                az: us-east-1d
                cidr: 172.16.4.0/24
                route: rt_public
                public_ip: true
              - name: net_nodes_private_3
                az: us-east-1d
                cidr: 172.16.5.0/24
                route: rt_private
              - name: net_nodes_public_4
                az: us-east-1e
                cidr: 172.16.6.0/24
                route: rt_public
                public_ip: true
              - name: net_nodes_private_4
                az: us-east-1e
                cidr: 172.16.7.0/24
                route: rt_private
              - name: net_elb_public_1
                az: us-east-1b
                cidr: 172.16.10.0/24
                route: rt_public
                public_ip: true
              - name: net_elb_public_2
                az: us-east-1c
                cidr: 172.16.11.0/24
                route: rt_public
                public_ip: true
              - name: net_elb_public_3
                az: us-east-1d
                cidr: 172.16.12.0/24
                route: rt_public
                public_ip: true
              - name: net_elb_public_4
                az: us-east-1e
                cidr: 172.16.13.0/24
                route: rt_public
                public_ip: true
              - name: net_vpn_public_1
                az: us-east-1b
                cidr: 172.16.29.0/24
                route: rt_public
                public_ip: true
              # - name: net_vpn_public_2
              #   az: us-east-1e
              #   cidr: 172.16.30.0/24
              #   route: rt_public
              #   public_ip: true
              # - name: net_natgw_public
              #   az: us-east-1b
              #   cidr: 172.16.31.0/26
              #   route: rt_natgw
              #   public_ip: true
          - name: vpc_use1_gce
            block: 172.17.0.0/20
            provider: gce
            region: us-east-1
            subnets:
              - name: net-nodes-public-1
                cidr: 172.17.0.0/24
              - name: net-nodes-private-1
                cidr: 172.17.1.0/24
              - name: net-nodes-public-2
                cidr: 172.17.2.0/24
              - name: net-nodes-private-2
                cidr: 172.17.3.0/24
              - name: net-nzdes-public-3
                cidr: 172.17.4.0/24
              - name: net-nodes-private-3
                cidr: 172.17.5.0/24
              - name: net-nodes-public-4
                cidr: 172.17.6.0/24
              - name: net-nodes-private-4
                cidr: 172.17.7.0/24
              - name: net-elb-public-1
                cidr: 172.17.10.0/24
              - name: net-elb-public-2
                cidr: 172.17.11.0/24
              - name: net-elb-public-3
                cidr: 172.17.12.0/24
              - name: net-elb-public-4
                cidr: 172.17.13.0/24
              - name: net-vpn-public-1
                cidr: 172.17.31.0/24

        roles:
           - { role: cloud-iam.mtulio }



Changelog
---------

* Support NATGW discovery ID and attach it to the private RTb
* Support VPC Peering route in route table


License
-------

GPLv3

Author Information
------------------

[Marco Tulio R Braga](https://github.com/mtulio)
