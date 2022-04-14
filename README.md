cloud-network
=============

[![Project Status: Concept - initial development is in progress, but there has not yet been a stable, usable release suitable for the public.](http://www.repostatus.org/badges/latest/wip.svg)](http://www.repostatus.org/#wip)

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

> TODO

Dependencies
------------

`cloud_networks`: List of networks to be created on the Cloud Provider.

Example Playbook
----------------

Create the var file `vars/networks/k8s-aws.yaml`:

```yaml
---
cloud_networks:
  ## AWS South East 1
  - name: k8s-vpc-use1
    block: 10.82.0.0/16
    provider: aws
    region: us-east-1
    igw: yes
    nat_gw: yes
    nat_gw_subnet: k8s-net-public-natgw-use1-1a
    nat_gw_name: search-natgw-1a
    security_groups: "{{ security_groups | d([]) }}"
    routes:
      - name: k8s-rt-private
        table:
          - dest: 0.0.0.0/0
            gateway_id: natgw
      - name: k8s-rt-public
        table:
          - dest: 0.0.0.0/0
            gateway_id: igw
      - name: k8s-rt-natgw
        table:
          - dest: 0.0.0.0/0
            gateway_id: igw
    subnets:
      - name: k8s-net-public-natgw-use1-1a
        az: us-east-1a
        cidr: 10.82.0.0/28
        route: k8s-rt-natgw
        public_ip: true

      - name: k8s-net-public-lb-use1-1a
        az: us-east-1a
        cidr: 10.82.1.0/24
        route: k8s-rt-public
        public_ip: true
      - name: k8s-net-public-lb-use1-1b
        az: us-east-1b
        cidr: 10.82.2.0/24
        route: k8s-rt-public
        public_ip: true

      - name: k8s-net-private-lb-use1-1a
        az: us-east-1a
        cidr: 10.82.8.0/24
        route: k8s-rt-private
        public_ip: false
      - name: k8s-net-private-lb-use1-1b
        az: us-east-1b
        cidr: 10.82.9.0/24
        route: k8s-rt-private
        public_ip: false

      - name: k8s-net-public-nodes-use1-1a
        az: us-east-1a
        cidr: 10.82.16.0/22
        route: k8s-rt-public
        public_ip: true
      - name: k8s-net-public-nodes-use1-1b
        az: us-east-1b
        cidr: 10.82.20.0/22
        route: k8s-rt-public
        public_ip: true

      - name: k8s-net-private-nodes-use1-1a
        az: us-east-1a
        cidr: 10.82.48.0/22
        route: k8s-rt-private
        public_ip: false
      - name: k8s-net-private-nodes-use1-1b
        az: us-east-1b
        cidr: 10.82.52.0/22
        route: k8s-rt-private
        public_ip: false

    endpoint_services:
      - name: s3
        service: com.amazonaws.us-east-1.s3
        route_table_names:
          - k8s-rt-public
          - k8s-rt-private

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
