# How to write bpf

1. [BBC方式]编写BPF程序，iovisor/bcc[here](https://github.com/iovisor/bcc)项目是一个很好的开始。使用这个项目，可以在python中写BPF代码直接运行，源码里有许多例子可以学习。BCC is a toolkit for creating efficient kernel tracing and manipulation programs, and includes several useful tools and examples
2. [直接编译bpf字节码方式+bpf调用]：前面提到，我们可以自己编写BPF程序，然后使用llvm+clang编译成BPF格式的字节码(.o字节码)编译命令也十分简单，clang -O2 -target bpf -o bpf prog.o -c bpf prog.c.，然后可以使用系统调用bpf(...)来加载到内核。
3. [bpftool]除了自己写代码操作BPF程序，一些工具也可以帮助我们做到这一点。例如linux源码自带的bpftool可以操作部分BPF程序和map，iproute可以将BPF程序加载到网卡，tc可以将tc相关BPF程序加载到tc。tool for inspection and simple manipulation of eBPF programs and maps

##  BPF 程序开发方式：
1. 编写一段 BPF 程序
2. 编译这段 BPF 程序
3. 用一个特殊的系统调用将编译后的代码加载到内核

## BCC
将bpf代码嵌入在python中,解决了加载的问题？
```python
b = BPF(text="""
#include <uapi/linux/ptrace.h>
#include <linux/blkdev.h>

BPF_HISTOGRAM(dist);
BPF_HISTOGRAM(dist_linear);

int kprobe__blk_account_io_done(struct pt_regs *ctx, struct request *req)
{
	dist.increment(bpf_log2l(req->__data_len / 1024));
	dist_linear.increment(req->__data_len / 1024);
	return 0;
}
""")
```
## clang+bpf load
## bpftool 
https://github.com/torvalds/linux/blob/v5.4/tools/bpf/bpftool/Documentation/bpftool.rst
install commands
```
 git clone --depth=1 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
 cd linux/tools/bpf/bpftool
 make
```
output
 ```bash
Auto-detecting system features:
...                        libbfd: [ on  ]
...        disassembler-four-args: [ on  ]
...                          zlib: [ on  ]
...                        libcap: [ OFF ]
...               clang-bpf-co-re: [ on  ]

  CC       map_perf_ring.o
  CC       xlated_dumper.o
  CC       iter.o
  CC       btf.o
  CC       tracelog.o
  CC       perf.o
  CC       common.o
  CC       btf_dumper.o
  CC       net.o
  CC       struct_ops.o
  CC       netlink_dumper.o
  CC       link.o
  CC       cgroup.o
  CC       cfg.o
  CC       gen.o
  CC       main.o
  CC       json_writer.o
  CC       prog.o
  CC       map.o
  CC       pids.o
  CC       feature.o
  CC       jit_disasm.o
  CC       disasm.o
make[1]: Entering directory '/vagrant/linux/tools/lib/bpf'

Auto-detecting system features:
...                        libelf: [ on  ]
...                          zlib: [ on  ]
...                           bpf: [ on  ]

  GEN      bpf_helper_defs.h
  MKDIR    staticobjs/
  CC       staticobjs/libbpf.o
  CC       staticobjs/bpf.o
  CC       staticobjs/nlattr.o
  CC       staticobjs/btf.o
  CC       staticobjs/libbpf_errno.o
  CC       staticobjs/str_error.o
  CC       staticobjs/netlink.o
  CC       staticobjs/bpf_prog_linfo.o
  CC       staticobjs/libbpf_probes.o
  CC       staticobjs/xsk.o
  CC       staticobjs/hashmap.o
  CC       staticobjs/btf_dump.o
  CC       staticobjs/ringbuf.o
  LD       staticobjs/libbpf-in.o
  LINK     libbpf.a
make[1]: Leaving directory '/vagrant/linux/tools/lib/bpf'
  LINK     bpftool
```
## bpftool usage
```
root@node-00:/vagrant/linux/tools/bpf/bpftool# ./bpftool 
Usage: ./bpftool [OPTIONS] OBJECT { COMMAND | help }
       ./bpftool batch file FILE
       ./bpftool version

       OBJECT := { prog | map | link | cgroup | perf | net | feature | btf | gen | struct_ops | iter }
       OPTIONS := { {-j|--json} [{-p|--pretty}] | {-f|--bpffs} |
                    {-m|--mapcompat} | {-n|--nomount} }
```


## compile generate bpf bytecode for sockops
> root@node-00:/vagrant/bpf/sockops# make all
```
clang -I/vagrant/bpf/include -I/vagrant/bpf -D__NR_CPUS__=2 -O2 -g -target bpf -emit-llvm -Wall -Werror -Wno-address-of-packed-member -Wno-unknown-warning-option -DSKIP_DEBUG -DHAVE_LPM_MAP_TYPE -DHAVE_LRU_MAP_TYPE -DENABLE_IPV4 -c bpf_sockops.c -o bpf_sockops.ll

llc -march=bpf -mcpu=probe -mattr=dwarfris -filetype=obj -o bpf_sockops.o bpf_sockops.ll
clang -I/vagrant/bpf/include -I/vagrant/bpf -D__NR_CPUS__=2 -O2 -g -target bpf -emit-llvm -Wall -Werror -Wno-address-of-packed-member -Wno-unknown-warning-option -DSKIP_DEBUG -DHAVE_LPM_MAP_TYPE -DHAVE_LRU_MAP_TYPE -DENABLE_IPV4 -c bpf_redir.c -o bpf_redir.ll

llc -march=bpf -mcpu=probe -mattr=dwarfris -filetype=obj -o bpf_redir.o bpf_redir.ll
```

## load bpf bytecode with bpftool
```
root@node-00:/vagrant# bash load.sh 
make: Entering directory '/vagrant/bpf/sockops'
rm -fr *.o *.ll *.i
make: Leaving directory '/vagrant/bpf/sockops'
make: Entering directory '/vagrant/bpf/sockops'
clang -I/vagrant/bpf/include -I/vagrant/bpf -D__NR_CPUS__=2 -O2 -g -target bpf -emit-llvm -Wall -Werror -Wno-address-of-packed-member -Wno-unknown-warning-option -DSKIP_DEBUG -DHAVE_LPM_MAP_TYPE -DHAVE_LRU_MAP_TYPE -DENABLE_IPV4 -c bpf_sockops.c -o bpf_sockops.ll
llc -march=bpf -mcpu=probe -mattr=dwarfris -filetype=obj -o bpf_sockops.o bpf_sockops.ll
clang -I/vagrant/bpf/include -I/vagrant/bpf -D__NR_CPUS__=2 -O2 -g -target bpf -emit-llvm -Wall -Werror -Wno-address-of-packed-member -Wno-unknown-warning-option -DSKIP_DEBUG -DHAVE_LPM_MAP_TYPE -DHAVE_LRU_MAP_TYPE -DENABLE_IPV4 -c bpf_redir.c -o bpf_redir.ll
llc -march=bpf -mcpu=probe -mattr=dwarfris -filetype=obj -o bpf_redir.o bpf_redir.ll
make: Leaving directory '/vagrant/bpf/sockops'
libbpf: maps section in bpf/sockops/bpf_sockops.o: "sock_ops_map" has unrecognized, non-zero options
Error: failed to pin program sockops
libbpf: maps section in bpf/sockops/bpf_redir.o: "sock_ops_map" has unrecognized, non-zero options
```

## load bpf analysis

bpftool -m prog load bpf/sockops/bpf_sockops.o "/sys/fs/bpf/bpf_sockop"

bpftool cgroup attach "/sys/fs/cgroup/unified/" sock_ops pinned "/sys/fs/bpf/bpf_sockop"

bpftool prog show pinned "/sys/fs/bpf/bpf_sockop" 
```
29: sock_ops  name bpf_sockmap  tag 9bd001600514d41a  gpl
        loaded_at 2020-12-21T16:30:21+0800  uid 0
        xlated 608B  jited 362B  memlock 4096B  map_ids 1
        btf_id 4

```
$MAP_ID=map_ids=1

bpftool map pin id $MAP_ID "/sys/fs/bpf/sock_ops_map"

bpftool -m prog load bpf/sockops/bpf_redir.o "/sys/fs/bpf/bpf_redir" map name sock_ops_map pinned "/sys/fs/bpf/sock_ops_map"
> /sys/fs/bpf/bpf_sockop ===> /sys/fs/bpf/sock_ops_map ==> /sys/fs/bpf/bpf_redir
bpftool prog attach pinned "/sys/fs/bpf/bpf_redir" msg_verdict pinned "/sys/fs/bpf/sock_ops_map"


## 
/sys/fs/bpf/bpf_sockop ===> /sys/fs/bpf/sock_ops_map ==> 


## bpftool 使用
### bpftool prog show 
列出了bpftrace的program ID (232), BCC的program ID (263) 以及其他BPF programs.

请注意，BCC kprobe程序具有BPF类型格式(BTF)信息，此输出中存在btf_id来显示该信息


```bash
29: sock_ops  name bpf_sockmap  tag 9bd001600514d41a  gpl
        loaded_at 2020-12-21T16:30:21+0800  uid 0
        xlated 608B  jited 362B  memlock 4096B  map_ids 1
        btf_id 4
39: sk_msg  name bpf_redir_proxy  tag 99daa20a0b4a3074  gpl
        loaded_at 2020-12-21T16:30:39+0800  uid 0
        xlated 304B  jited 195B  memlock 4096B  map_ids 1
        btf_id 12
46: cgroup_skb  tag 6deef7357e7b4530  gpl
        loaded_at 2020-12-21T23:24:10+0800  uid 0
        xlated 64B  jited 61B  memlock 4096B
```
> map_ids 1 关联了什么??  

### bpftool-map

# http://manpages.ubuntu.com/manpages/focal/man8/bpftool-map.8.html