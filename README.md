# Cluster of Docker Servers deployed with BOSH

Run a cluster of VMs each running Docker server, deployed and managed with BOSH.

## Usage

To use this bosh release, first upload it to your bosh:

```
bosh target BOSH_HOST
git clone https://github.com/cloudfoundry-community/docker-server-boshrelease.git
cd docker-server-boshrelease
bosh upload release releases/docker-server-1.yml
```

### AWS

For AWS EC2, upload the custom stemcell and then create a single VM:

```
bosh upload stemcell https://s3.amazonaws.com/cloudcredo-internal-distribution/bosh-stemcell-71-aws-xen-ubuntu.tgz
templates/make_manifest aws-ec2 templates/booting-new-server.yml
bosh -n deploy
```

Subsequent deploys where new servers are not being added can have the `templates/booting-new-server.yml` removed from the `make_manifest` line. It exists to extend the timeout long enough to allow `docker-lxc` to be installed.

```
templates/make_manifest aws-ec2
bosh -n deploy
```

### Additional infrastructures & operating systems

BOSH supports AWS, OpenStack, vSphere and many other infrastuctures. There may not be stemcells available that support Docker.

In this current spike it is hard coded to support Ubuntu; and thus you must use an Ubuntu stemcell that is ready to install LXC and Docker (which is automated at runtime).

More work is coming for optimizing the installation of Docker; for support for CentOS; and for support for additional infrastructures.

## Options

There are also additional templates that you can mix in to enabled/disable/configure your Docker servers.

### Disable restarting of containers on docker restart

If you are delegating the start/stop of containers to BOSH itself then you will want to disable Docker automatically restarting containers:

```
templates/make_manifest aws-ec2 templates/disable-restart-containers.yml
bosh -n deploy
```

### Public access to Docker server

If you want to allow clients to the Docker server to access it from outside the VM, you can include the following template:

```
templates/make_manifest aws-ec2 templates/public-access.yml
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

### Contributions

* Dr Nic Williams wrote a lot of this
* Colin Humpheries created the initial `bosh-stemcell-71-aws-xen-ubuntu` stemcell that allowed work to be kicked off

