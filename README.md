# BOSH Release for docker-server

WIP. Currently trying to debug the following error:

```
==> /var/vcap/sys/log/docker_server/docker_server.stderr.log <==
2014/02/12 08:36:27 WARNING: You are running linux kernel version 3.0.0-32-virtual, which might be unstable running docker. Please upgrade your kernel to 3.8.0.
[/var/vcap/data/docker|d8ad31bb] +job initserver()
[/var/vcap/data/docker|d8ad31bb.initserver()] Creating server
mkdir /var/vcap/data/docker/containers: permission denied[/var/vcap/data/docker|d8ad31bb] -job initserver() = ERR (1)
2014/02/12 08:36:27 initserver: mkdir /var/vcap/data/docker/containers: permission denied
```

## Usage

To use this bosh release, first upload it to your bosh:

```
bosh target BOSH_HOST
git clone https://github.com/cloudfoundry-community/docker-server-boshrelease.git
cd docker-server-boshrelease
bosh upload release releases/docker-server-1.yml
```

For [bosh-lite](https://github.com/cloudfoundry/bosh-lite), you can quickly create a deployment manifest & deploy a 3 VM cluster:

```
templates/make_manifest warden
bosh -n deploy
```

For Openstack (Nova Networks), create a single VM:

```
templates/make_manifest openstack-nova
bosh -n deploy
```

For AWS EC2, create a single VM:

```
templates/make_manifest aws-ec2
bosh -n deploy
```

### Override security groups

For AWS & Openstack, the default deployment assumes there is a `default` security group. If you wish to use a different security group(s) then you can pass in additional configuration when running `make_manifest` above.

Create a file `my-networking.yml`:

``` yaml
---
networks:
  - name: docker-server1
    type: dynamic
    cloud_properties:
      security_groups:
        - docker-server
```

Where `- docker-server` means you wish to use an existing security group called `docker-server`.

You now suffix this file path to the `make_manifest` command:

```
templates/make_manifest openstack-nova my-networking.yml
bosh -n deploy
```

## Development

### Bump docker CLI version

```
wget https://get.docker.io/builds/Linux/x86_64/docker-latest -O docker
mv docker blobs/docker/docker-X.Y.Z
```

Then remove the old version from `config/blobs.yml`.

Next, update `packages/docker/spec` and `packages/docker/packaging` for the new version.
