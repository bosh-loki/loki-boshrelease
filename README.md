# Loki BOSH Release

This is a [BOSH](http://bosh.io/) release for [Loki](https://grafana.com/loki), and [Grafana](https://grafana.com/).


## Get Started

Assuming you've installed `BOSH CLI` and had a running BOSH Director.

For example:

```
$ bosh --version
version 5.5.1-7850ac98-2019-05-21T22:28:39Z

Succeeded

$ bosh env
Using environment '10.197.81.30' as client 'admin'

Name               bosh-1
UUID               a1dc5a23-b10d-40ba-8391-085d612f90e0
Version            270.2.0 (00000000)
Director Stemcell  ubuntu-xenial/315.64
CPI                vsphere_cpi
Features           compiled_package_cache: disabled
                   config_server: enabled
                   local_dns: enabled
                   power_dns: disabled
                   snapshots: disabled
User               admin

Succeeded
```

> Note: please follow the docs [here](https://github.com/cloudfoundry/bosh-deployment) for how to spin up a BOSH Director.

### Install Loki BOSH Release

```
$ git clone https://github.com/bosh-loki/loki-boshrelease.git

$ pushd loki-boshrelease \
    && bosh create-release --force && bosh upload-release \
    && bosh -d loki deploy manifests/loki.yml \
    && popd

$ bosh -d loki vms
Using environment '10.197.81.30' as client 'admin'

Task 318. Done

Deployment 'loki'

Instance                                       Process State  AZ  IPs           VM CID                                   VM Type  Active
database/82519de7-f196-4e15-a8fa-bef7d69a1134  running        z1  10.197.81.51  vm-3d21f0d7-e944-4f59-871b-0c6e213cb1ea  default  true
grafana/b897d579-b610-43ab-a2f3-ba82d8d1e286   running        z1  10.197.81.52  vm-d6a5c669-b0ad-4255-b6c1-765640f1ee87  default  true
loki/b96a577d-9e4c-48e3-a019-27c8a2b89e81      running        z1  10.197.81.50  vm-ee071874-d04a-422b-847a-a44c172d53b8  default  true

3 vms

Succeeded
```

### Access Loki Grafana

Access it through: `http://<GRAFANA_IP>:3000`.
So in my case it's: http://10.197.81.52:3000/

But what's the login credential?
Well, above deployment indicates that the BOSH Director has integrated with the Credential Management tool -- in my case, it's [CredHub](https://github.com/cloudfoundry-incubator/credhub).

So let's get back the generated credentials:

```
$ credhub find | grep loki
- name: /bosh-1/loki/grafana_secret_key
- name: /bosh-1/loki/grafana_password
- name: /bosh-1/loki/postgres_grafana_password
- name: /bosh-1/loki/prometheus_password

$ credhub get -n /bosh-1/loki/grafana_password
id: 618369c9-f446-48b7-a218-30f6cfad8f19
name: /bosh-1/loki/grafana_password
type: password
value: YEWlKixs72Hfx886xNvJv9sU2I5Z8K
version_created_at: "2019-07-31T05:59:08Z"
```

So you can login Grafana by: admin/YEWlKixs72Hfx886xNvJv9sU2I5Z8K

> Note: Of course, you can set your own credentials by either of the two ways:
> 1. (Recommended) Set them in CredHub first, by using commands like `credhub set -n /bosh-1/loki/grafana_password -t password`, and the deployment will just use them without the need to generate;
> 2. Set them directly through the `bosh -d loki deploy xxx -v grafana_password=MySuperPassword`. But it's not recommended as these credentails can be retrieved back in plain text if you run `bosh -d loki manifest`.

## Tear Down

To remove the deployment:

```
$ bosh -d loki delete-deployment
```
