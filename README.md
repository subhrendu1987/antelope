# OS Configuration
## CentOS 8.4.2105 VirtualBox Image (Using Virtual Box)
### Image name 
* [Download](https://www.linuxvmimages.com/images/virtualbox/)
* Configure yum repository
* * Modify `/etc/yum.repos.d/CentOS-Linux-AppStream.repo` `baseurl=http://vault.centos.org/$contentdir/$releasever/AppStream/$basearch/os/`
  * Instruction to download custom kernel `https://wiki.centos.org/MarcusFurlong(2f)Custom_Kernel_draft.html` to fix the dependencies
  * Inside `~/rpmbuild/SPEC` download kernel.spec file from `https://git.centos.org/rpms/kernel/blob/cb9fcbaee7eed97ebe55d71dd693715741dfefb2/f/SPECS/kernel.spec`
  * `sudo dnf builddep kernel.spec` Resolve building dependencies.
  * `rpmbuild -bp --target=$(uname -m) kernel.spec`
  * `cd ../BUILD/kernel-XXXX/linux-xxx`
  * `make menuconfig`
  * `make -j4`
  * `make -j4 modules install`
  * `sudo make install`


# Antelope: 
## a system which can adaptively choose the most suitable congestion control mechanism for a certain flow. 

The antelope system is divided into two parts, one is the kernel module and the other is the user_space module.

Before you run antelope, you need to modify the linux kernel so that antelope can get the required information. The modification is shown in linux-4.18.0-147_el8_diff.txt.

Then you need to install bcc, so that getSocketInfo.py use bcc mechanism to read the socket information.

Being able to switch CC mechainsm flexibly is very important for antelope, and ebpf will help us achieve this function.To load the ebpf program into the kernel, you need：
  1: make samples/bpf/
  
  2: mkdir -p /tmp/cgroupv2
 
  3: mount -t cgroup2 none /tmp/cgroupv2
  
  4: mkdir -p /tmp/cgroupv2/foo
  
  5: bash
  
  6: echo $$ >> /tmp/cgroupv2/foo/cgroup.procs
  
  7: ./samples/bpf/load_sock_ops -l /tmp/cgroupv2/foo ./samples/bpf/tcp_changecc_kern.o &

When the preparatory work is completed, you can run the recvAndSetCC.py, then Antelope which try to choose a suitable CC mechanism for different TCP flows according their in-flow statistical information is runing


=======Distributed scene==========

In order to reduce the overhead of machine learning on the end server, we split the recvAndSetCC.py into cc-server.py and recvAndSetCC_distributed.py. Learning cloud server run cc-server.py to choose the suitable CC mechanism. End server run recvAndSetCC_distributed.py to obtain the CC selected by learning cloud server, and perform CC switching.
