cloud-network
=============

[![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)
[![](https://github.com/mtulio/ansible-role-cloud-network/actions/workflows/release.yml/badge.svg)](https://github.com/mtulio/ansible-role-cloud-network/actions/workflows/release.yml)
[![](https://github.com/mtulio/ansible-role-cloud-network/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/mtulio/ansible-role-cloud-network/actions/workflows/ci.yml)
[![](https://img.shields.io/ansible/role/59532)](https://galaxy.ansible.com/mtulio/cloud_network)


Ansible role to manage all resources in Cloud Network Infrastructure.

Supported Cloud Providers:

* AWS - Amazon Web Services

Main roles to manage:

* Facts
* VPC
* Subnets
* Internet Gateway
* Route tables

Requirements
------------

All Providers:

* ansible >= 2.3

Provider=AWS:

* boto3


Role Variables
--------------

`cloud_networks`: List of networks to be created on the Cloud Provider.

Dependencies
------------

> TBD

Example Playbook
----------------

Create the var file `vars/networks/k8s-aws.yaml`:

```yaml
---
cloud_networks:
  ## AWS US East 1 (HA NatGW topology)
  - name: k8s-vpc-use1
    block: 10.0.0.0/16
    provider: aws
    region: us-east-1
    security_groups: "{{ security_groups | d([]) }}"
    internet_gateway: true
    carrier_gateway: true
    nat_gateways:
      - name: "k8s-natgw-1a"
        subnet: "k8s-net-public-use1-1a"
        tags: "{{ tags | d({}) }}"
        wait: false
      - name: "k8s-natgw-1b"
        subnet: "k8s-net-public-use1-1b"
        tags: "{{ tags | d({}) }}"
        wait: false
      - name: "k8s-natgw-1c"
        subnet: "k8s-net-public-use1-1c"
        tags: "{{ tags | d({}) }}"
        wait: true

    route_tables:
      - name: "k8s-rt-private-1a"
        routes:
          - dest: 0.0.0.0/0
            gw_type: natgw
            target: "k8s-natgw-1a"

      - name: "k8s-rt-private-1b"
        routes:
          - dest: 0.0.0.0/0
            gw_type: natgw
            target: "k8s-natgw-1b"

      - name: "k8s-rt-private-1c"
        routes:
          - dest: 0.0.0.0/0
            gw_type: natgw
            target: "k8s-natgw-1c"

      - name: "k8s-rt-public"
        routes:
          - dest: 0.0.0.0/0
            gw_type: igw

      - name: "k8s-rt-public-edge"
        routes:
          - dest: 0.0.0.0/0
            gw_type: cagw

   subnets:
      - name: "k8s-net-public-use1-1a"
        az: us-east-1a
        cidr: 10.0.16.0/22
        route_table: "k8s-rt-public"
        map_public: true
      - name: "k8s-net-public-use1-1b"
        az: us-east-1b
        cidr: 10.0.20.0/22
        route_table: "k8s-rt-public"
        map_public: true
      - name: "k8s-net-public-use1-1c"
        az: us-east-1c
        cidr: 10.0.24.0/22
        route_table: "k8s-rt-public"
        map_public: true

      - name: "k8s-net-private-use1-1a"
        az: us-east-1a
        cidr: 10.0.48.0/22
        route_table: "k8s-rt-private-1a"
        map_public: false
      - name: "k8s-net-private-use1-1b"
        az: us-east-1b
        cidr: 10.0.52.0/22
        route_table: "k8s-rt-private-1b"
        map_public: false
      - name: "k8s-net-private-use1-1c"
        az: us-east-1c
        cidr: 10.0.56.0/22
        route_table: "k8s-rt-private-1c"
        map_public: false

      # Edge (Local Zone) subnets
      - name: "k8s-net-public-use1-nyc-lz-1a"
        az: us-east-1-nyc-1a
        cidr: 10.0.60.0/22
        route_table: "k8s-rt-public"
        map_public: true

      - name: "k8s-net-private-use1-nyc-lz-1a"
        az: us-east-1-nyc-1a
        cidr: 10.0.64.0/22
        route_table: "k8s-rt-private-1a"
        map_public: false

      # Edge (Wavelength) subnets
      - name: "k8s-net-public-use1-nyc-wlz-1"
        az: us-east-1-wl1-nyc-wlz-1
        cidr: 10.0.68.0/22
        route_table: "k8s-rt-public-edge"
        map_public: false

    endpoint_services:
      - name: s3
        service: com.amazonaws.us-east-1.s3
        route_tables:
          - "k8s-rt-public"
          - "k8s-rt-private-1a"
          - "k8s-rt-private-1b"
          - "k8s-rt-private-1c"
          - "k8s-rt-public-edge"

```

Create the Plabook `net-create.yaml`:

```yaml
---
- hosts: localhost

  # To skip prompt, define the extra-arg 'name'
  vars_prompt:
    - name: provider
      prompt: What is the cloud provider name?
      private: no
    - name: name
      prompt: What is the network name?
      private: no

  pre_tasks:
    - include_vars: "vars/networks/{{ name }}-{{ provider }}.yaml"

  roles:
    - role: mtulio.cloud-vpc
```

Run the Playbook:

```bash
ansible-playbook net-create.yaml -e provider=aws -e name=k8s
```

License
-------

GPLv3

Author Information
------------------

[Marco Tulio R Braga](https://github.com/mtulio)
