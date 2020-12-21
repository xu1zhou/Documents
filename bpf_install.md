# 

# bpftool install
除了自己写代码操作BPF程序，一些工具也可以帮助我们做到这一点。例如linux源码自带的bpftool可以操作部分BPF程序和map，iproute可以将BPF程序加载到网卡，tc可以将tc相关BPF程序加载到tc。
```
 git clone --depth=1 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
 cd linux/tools/bpf/bpftool
 make
```

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