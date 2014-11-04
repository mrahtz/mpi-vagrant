# MPI Cluster Vagrantfile

This is a Vagrantfile (to be used with Hashicorp's
[Vagrant](https://vagrantcloud.com/) tool) for automatically bringing up
a cluster suitable for testing MPI loads.

Practically, all this involves is bringing up several VMs on a private network,
setting up SSH key-based authentication between them, and installing OpenMPI.

Currently only works with the VirtualBox provider.

## Usage

In your checkout directory, simply run:
```
vagrant up
```
By default, the cluster will be made of 3 VMs. If you want more,
change `N_VMS` in the `Vagrantfile`.

The VMs will be named `node1` through `node<n>`. To SSH to, say, `node1`:
```
vagrant ssh node1
```

As a simple sanity check, try running `hostname` on each machine in the
cluster:
```
mpirun -np 3 --host node1,node2,node3 hostname
```
OpenMPI will try to use all networks it thinks are common
to all hosts, including Vagrant's host-only networks. This will
break any MPI jobs which attempt to actually do any communication.
To  work around this, you should tell `mpirun` explicitly
which networks to use:
```
mpirun -np 3 --host node1,node2,node3 --mca btl_tcp_if_include 192.168.0.0/24 hostname
```
For more detail, see http://www.open-mpi.org/faq/?category=tcp#tcp-selection.
