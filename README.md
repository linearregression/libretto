[![License][License-Image]][License-URL] [![ReportCard][ReportCard-Image]][ReportCard-URL] [![Build][Build-Status-Image]][Build-Status-URL]
Libretto
===============

Libretto is a Golang library to create Virtual Machines (VM) on any cloud and
Virtual Machine hosting platforms such as AWS, Azure, OpenStack, vSphere,
or VirtualBox. Different providers have different utilities and API interfaces
to achieve that, but the abstractions of their interfaces are quite similar.

Supported Providers
====================
* vSphere > 5.0
* AWS
* Openstack (Mirantis)
* VMware Fusion >= 8.0
* VMware Workstation >= 8.0
* Virtualbox >= 4.3.30
* Azure
* DigitalOcean

Getting Started
================

`go get github.com/apcera/libretto/...`

`go build ./...`

Examples
==========

AWS
----

``` go

rawKey, err := ioutil.ReadFile(*key)
if err != nil {
        return err
}

vm := &aws.VM{
        Name:         "libretto-aws",
        AMI:          "ami-984734",
        InstanceType: "m4.large",
        SSHCreds: ssh.Credentials{
                SSHUser:       "ubuntu",
                SSHPrivateKey: string(rawKey),
        },
        DeviceName:    "/dev/sda1",
        Region:        "ap-northeast-1",
        KeyPair:       strings.TrimSuffix(filepath.Base(*key), filepath.Ext(*key)),
        SecurityGroup: "sg-9fdsfds",
}

if err := aws.ValidCredentials(vm.Region); err != nil {
        return err
}

if err := vm.Provision(); err != nil {
        return err
}

```

vSphere
--------

``` go

vm := &vsphere.VM{
        Host:       "10.2.1.11",
        Username:   "username",
        Password:   "password",
        Datacenter: "test-dc",
        Datastores: "datastore1, datastore2",
        Networks:   "network1",
        Credentials: ssh.Credentials{
            SSHUser:     "ubuntu",
            SSHPassword: "ubuntu",
        },
        SkipExisting: true,
        DestinationName: "Host1",
        DestinationType: "host",
        Name: "test-vm",
        Template: "test-template",
        OvfPath: "/Users/Test/Downloads/file.ovf",
}
if err := vm.Provision(); err != nil {
        return err
}
```

Digital Ocean
-------------

``` go
token := os.Getenv("DIGITALOCEAN_API_KEY")
if token == "" {
    return fmt.Errorf("Please export your DigitalOcean API key to 'DIGITALOCEAN_API_KEY' and run again.")
}
config := digitalocean.Config{
    Name:   defaultDropletName,
    Region: defaultDropletRegion,
    Image:  defaultDropletImage,
    Size:   defaultDropletSize,
}

vm := digitalocean.VM{
    ApiToken: token,
    Config:   config,
}

if err := vm.Provision(); err != nil {
    return err
}
```

Virtualbox
-------------

``` go

var config virtualbox.Config
config.NICs = []virtualbox.NIC{
    virtualbox.NIC{Idx: 1, Backing: virtualbox.Bridged, BackingDevice: "en0: Wi-Fi (AirPort)"},
}
vm := virtualbox.VM{Src: "/Users/Admin/vm-bfb21a62-60c5-11e5-9fc5-a45e60e45ad5.ova",
    Credentials: ssh.Credentials{
        SSHUser:     "ubuntu",
        SSHPassword: "ubuntu",
    },
    Config: config,
}
if err := vm.Provision(); err != nil {
    return err
}
```

VMware Fusion/Workstation (vmrun)
---------------------------------

``` go
var config vmrun.Config
config.NICs = []vmrun.NIC{
    vmrun.NIC{Idx: 0, Backing: vmrun.Nat, BackingDevice: "en0"},
}
vm := vmrun.VM{Src: "/Users/Admin/vmware_desktop/trusty-orchestrator-dev.vmx",
    Dst: "/Users/Admin/Documents/VMs",
    Credentials: ssh.Credentials{
        SSHUser:     "ubuntu",
        SSHPassword: "ubuntu",
    },
    Config: config,
}
if err := vm.Provision(); err != nil {
    return err
}
```

Openstack
---------

``` go

    metadata := openstack.NewDefaultImageMetadata()
	volume := openstack.NewDefaultVolume()

	vm := &openstack.VM{
		IdentityEndpoint: os.Getenv("OS_AUTH_URL"),
		Username:         os.Getenv("OS_USERNAME"),
		Password:         os.Getenv("OS_PASSWORD"),
		Region:           os.Getenv("OS_REGION_NAME"),
		TenantName:       os.Getenv("OS_TENANT_NAME"),
		FlavorName:       "m1.medium",
		ImageID:          "",
		ImageMetadata:    metadata,
		ImagePath:        os.Getenv("OS_IMAGE_PATH"),
		Volume:           volume,
		InstanceID:       "",
		Name:             "test",
		Networks:         []string{"eab29109-3363-4b03-8a56-8fe27b71f3a0"},
		FloatingIPPool:   "net04_ext",
		FloatingIP:       nil,
		SecurityGroup:    "test",
		Credentials: ssh.Credentials{
			SSHUser:     "ubuntu",
			SSHPassword: "ubuntu",
		},
	}

	err := vm.Provision()
	if err != nil {
        return err
	}
 ```

FAQ
===

* Why write Libretto?

We couldn't find a suitable Golang binding for this functionality, so we
created this library. There are a couple of similar libraries but not in golang
(fog.io in ruby, jcloud in java, libcloud in python).

Docker Machine is an effort toward that direction, but it is very Docker
specific, and its providers dictate the VM images in many cases to reduce the
number of parameters, but reduces the flexibility of the tool.

* What is the scope for Libretto?

Virtual machine creation and life cycle
management as well as common configuration steps during a deploy such as
configuring SSH keys.

* Why use this library over other tools?

Actively used and developed. Can be called natively from Go in a Go application
instead of shelling out to other tools.

Known Issues
=============

*  Virtualbox networking is limited to using Bridged mode.

  Host to guest OS connectivity is not possible when using NAT networking in
  Virtualbox. As a result, presently networking configuration for VMs
  provisioned using the Virtualbox provider is limited to using Bridged
  networking. There should be a DHCP server running on the network that the VMs
  are bridged to.

Supported Platforms
====================
* Linux x64
* Windows 7 >= x64
* OS X >= 10.11

Other Operating Systems might work but have not been tested.

Adding Provisioners
=========================

Create a new package inside the `virtualmachine` folder and implement the
Libretto `VirtualMachine` interface. The provider should work at the minimum on
the Linux, Windows and OS X platforms unless it is a platform specific provider
in which case it should at least compile and return a descriptive error.

Dependencies should be versioned and stored using `Godeps`
(https://github.com/tools/godep)

Errors should be lower case so that they can be wrapped by the calling code. If
possible, types defined in the top level `virtualmachine` package should be
reused.

Contributors
=============

* David Liu
* Jaime Piña
* Josh Ellithorpe
* Kerim Oktay
* Lirui Li
* Preet Bhinder
* Ryan Boehning

[License-URL]: https://opensource.org/licenses/Apache-2.0
[License-Image]: https://img.shields.io/:license-apache-blue.svg
[ReportCard-URL]: https://goreportcard.com/report/github.com/apcera/libretto
[ReportCard-Image]: http://goreportcard.com/badge/apcera/libretto
[Build-Status-URL]: http://travis-ci.org/apcera/libretto
[Build-Status-Image]: https://travis-ci.org/apcera/libretto.svg?branch=master
