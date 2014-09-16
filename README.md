# Packer Templates for Mesos
__Bake your own mesos pre-installed images!!__  This contains packer templates for building mesos pre-installed images.  This also outputs vagrant boxes.

* `virtualbox.json`: building virtualbox image(`.ova`) and vagrant box file(`.box`) for virtualbox provider.
* `amis.json`: building AMIs and vagrant box(`.box`) for aws provider.

## What's inside.
Both templates will bake a virtual machine image and vagrant box containing:

* OS: Ubuntu 14.04 LTS 64-bit
* Mesos 0.20.0 (by Mesosphere's package)
  * _`mesos-{master|slave}` will not be started at bootup. But, `mesos-{master|slave}` service was registered.  So, you can start them manually by `# start mesos-{master| slave}`_
  * Please see the links for how to configure `mesos-{master|slave}`
    * [Mesos Master Configuration](http://mesosphere.io/docs/mesos/deep-dive/mesos-master/)
    * [Mesos Slave Configuration](http://mesosphere.io/docs/mesos/deep-dive/mesos-slave/)
  * Or, you can use [`cookbook-mesos`](https://github.com/everpeace/cookbook-mesos) to configure them.
* Marathon 0.7.0-RC2 (in /opt/marathon-`marathon_version`)
    * [Setting Up An Running Marathon](https://mesosphere.github.io/marathon/docs/)
* Chronos 2.1.0 (in /opt/chronos)
  * [Configuring Chronos](https://github.com/mesosphere/chronos#configuring-chronos)

## Hosted Images
Do you want to save time to bake these boxes by yourself!?  Yes! Images baked by these templates are uploaded on the web.

* VirtualBox:
  * to be uploaded
* AWS:
  * to be uploaded.
* [Vagrant Cloud](https://vagrantcloud.com/):
  * to be uploaded

## How to bake
### Pre-requisites
You need [berkshelf](http://berkshelf.com) and some cookbooks which these templates depends on and have to be placed in `vendor-cookbooks` directory.  You can install berkshelf with bundler and install dependencies with it:

```
$ bundle install
$ berks vendor vendor-cookbooks
```

### Virtualbox
```
$ packer build virtualbox.json
```

This build is done automatically (but it will take about 10 -15 minitutes), which performs bellows in summary:
* download ubuntu 14.04 LTS installation iso
* install ubuntu to vm (basic installation with Open SSH server)
* `apt-get update && apt-get upgrade`
* ssh configurations to make vagrant be able to ssh.
* install docker, mesos, marathon, chronos

If it went well, you can see

* `build/mesos-0.20.0_ubuntu-14.04_amd64_virtualbox_<timestamp>.ova`
* `build/mesos-0.20.0_ubuntu-14.04_amd64_virtualbox_<timestamp>.box`


Then,

* you can add this box to your vagrant:

  ```
  $ vagrant box add -name mesos-0.20.0-vbox mesos-0.20.0_ubuntu-14.04_virtualbox_<timestamp>.box
  ```
* you can import `.ova` file to your virtualbox.

### AWS
```
$ packer build -var 'aws_access_key=<your aws_access_key>' -var 'aws_secret_key=<your aws_secret_access_key>' amis.json
```

This build is done automatically (but it will take about 10 -15 minitutes), which performs bellows in summary:

* base AMI is  [`ami-73717d36`](http://thecloudmarket.com/image/ami-73717d36--ubuntu-images-ebs-ubuntu-trusty-14-04-amd64-server-20140816) in North California region(`us-west-1`)
* `apt-get update && apt-get upgrade`
* install docker, mesos, marathon, chronos
* AMI is distributed to all AWS regions:
  * `ap-northeast-1`, `ap-southeast-1`, `ap-southeast-2`, `eu-west-1`, `sa-east-1`, `us-east-1`, `us-west-1`, `us-west-2`
  * _you can control this `"ami_regions"` clause in the template_
* make them public.

If it went well, you can see `build/mesos-0.20.0_ubuntu-14.04_aws_<timestamp>.box`.

Then, you can add this box to your vagrant

```
$ vagrant box add -name mesos-0.20.0-aws mesos-0.20.0_ubuntu-14.04_aws_<timestamp>.box
```