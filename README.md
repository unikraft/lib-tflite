TensorFlowLite for Unikraft
=============================

This is the port of tensorflowlite as external library
Please refer to the `README.md` as well as the documentation in the `doc/`
subdirectory of the main unikraft repository.

## Build
TensorFlowLite interpreter depends on the following libraries, that need to
be added to `Makefile` in this order:

* `pthreads`, e.g. `pthread-embedded`
* `libcxx`
* `libcxxabi`
* `libc`, e.g. `newlib`
* `libunwind`
* `libcompilerrt`
* `libgemmlowp`
* `libflatbuffers`
* `libfarmhash`
* `libeigen`
* `libfft2`

## Root filesystem
### Creating the filesystem
TensorFlowLite needs a filesystem which should contain one or more *tflite*
models. Therefore, the filesystem needs to be created before running the VM. 

### Using the filesystem
Mounting the filesystem is a transparent operation. All you have to do
is to provide the right Qemu parameters in order for Unikraft to mount
the filesystem.  We will use the 9pfs support for filesystems and for
this you will need to use the following parameters:

```bash
-fsdev local,id=myid,path=<some directory>,security_model=none \
-device virtio-9p-pci,fsdev=myid,mount_tag=rootfs,disable-modern=on,disable-legacy=off
```
You should also use `vfs.rootdev=rootfs` (set by default) to specify the 9pfs mounting
tag to Unikraft. To enable 9pfs, you'll need to select the following
menu options, all under `Library Configuration` (this should be already done by the
`tflite` config file):

* `uk9p: 9p client`
* `vfscore: VFS Core Interface`
	  &rarr; `vfscore: Configuration`
	  &rarr; `Automatically mount a root filesysytem`
	  &rarr; `Default root filesystem`
	  &rarr; `9PFS`

Alternatively, and perhaps easier, is to use the qemu-guest script [here](https://github.com/unikraft/kraft/blob/master/scripts/qemu-guest):
```bash
kvm-guest -k build/helloworld_kvm-x86_64 -e rootfs -a "vfs.rootdev=fs0 --" -m 1024
```

## How to run
Currently, `main.cpp` contains a minimal example for loading a *tflite* model and
printing the interpreter state. The sample program will try to load the model from
`mobilenet_v1_1.0_224.tflite` (this model and other models from the same family can
be downloaded from [here](https://github.com/tensorflow/models/blob/master/research/slim/nets/mobilenet_v1.md))
