# Indico Cloud Images

This repository contains a set of scripts that easily allow you to create Indico images that can be run on virtualization software (VirtualBox, VMWare, etc...) or cloud services.

## Deploying on cloud providers (cloud-init)

All the documentation about cloud-init and configuration files can be found here: http://cloudinit.readthedocs.org/en/latest/.

Without going into much detail, the `user-data` file we will need for the remote deployment will be a MIME multipart file.
In particular, this MIME file will be composed by two different files:

 * `user-data-script.sh`: a bash script executed on the first boot to install and configure Indico on the VM.
 * `cloud-config`: a cloud-init configuration file, used to copy several files to the VM on the cloud.

We provide a useful script (`cloud-init/gen-user-data.py`) to automatically generate this `user-data` file, which can be used in RHEL/CentOS-compatible systems.

The script can take a configuration file to generate images with identical parameters.

```console
$ python gen-user-data.py
```

When the script finishes, a `user-data` file will be created in the same directory.

Then all you have to do is to boot a new instance of the base image you choose specifying the user-data file just created.
Be wary that the corresponding command will be different depending on the cloud service provider chosen.

### OpenStack

For instance, if we want to deploy a new Indico image into an OpenStack infrastructure we'll need to use the `nova` command. The actual command should be something like that:

```console
$ nova boot --image 'bfa5783c-e40e-4668-adc1-feb0ae3d7a46' --key-name your-nova-key-name --flavor general1-2 --config-drive true --user-data user-data indico-cloud-test
```

`indico-cloud-test` being the chosen hostname for the Indico server.

## Creating your own image

If you want to create your own image for testing purposes, you can do so by using the fabric script located under `dev`.
You should first have created a valid `user-data` file. It is important to include some extra information, such as a password for the default user (and an optional ssh key if you'd like). Networking should be enabled, so that the image can be accessed from outside QEMU/KVM:

```yaml
ssh_keys:
    - /home/jdoe/.ssh/id_rsa.pub
password: some_password
enable_networking: true
```

That said, you will also have to customize `dev/fabfile.conf`, namely:

```python
user = 'centos'
password = 'some_password'
img_name = "CentOS-7-x86_64-GenericCloud.qcow2"
```

In order to make sure that the default user and password are set and that the image file corresponds to what you have downloaded.
Then just do:

```console
$ fab create_vm_image
```

And wait till the program exits. You can check `qemu-output.log` in order to get some more debug data.

## Managing the server

Once you have a server deployed, you will probably want to start the database, the web server and the scheduler. Fortunately, we provide a fabric script that allows you to exactly that. You will need to set up a small config file based on `fabfile.conf.sample`. Normally, you will only need to change this part:

```python
machine = {
    "name": 'your-test-server-hostname',
    "ssh_port": 22
}
```

in order to suit your needs.

You will want to start all services:

```console
$ fab start:all
```

You can also start/stop individual services:

```console
$ fab stop:scheduler
```

