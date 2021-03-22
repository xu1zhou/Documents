

## 增加printk 函数到helper

## 增加macro定义

## 在函数中调用宏



# debug 循环

## 加载
bash load.sh

## curl productpage

## 查看printk信息
cat /sys/kernel/debug/tracing/trace_pipe


## 查看map中的socket
watch ./bpftool map dump id 117

## 性能测试
CID=$(docker run -d fortio/fortio:latest_release server)


POD_IP=$(kubectl get pod -o wide | grep productpage | awk '{print $6}')

./bpftool prog show | grep sock
CID=0a443a17162d
docker exec -it $CID fortio load -c 1 -qps 1000 -t 10s -a -r 0.00005 -httpbufferkb=128 "$POD_IP:9080/productpage"

# Verification redirection
默认情况下,host上 /var/log/kern.log 目录无法查看container中iptables log记录
可以使用以下命令开启

echo 1 > /proc/sys/net/netfilter/nf_log_all_netns

在iptables log中插入不同prefix以示区分
iptables -I INPUT -s 127.0.0.6 -j LOG --log-prefix '** host namespace **'
iptables -I INPUT -s 127.0.0.6 -j LOG --log-prefix '** container namespace **'

tail -f /var/log/kern.log

## test 
```sh
### enable
bash unload.sh;bash load.sh
CID=$(docker run -d fortio/fortio:latest_release server)
POD_IP=$(kubectl get pod -o wide | grep productpage | awk '{print $6}')
>test_result/enable_ebpf_performence_test1000_times.log
for i in {1..100};do docker exec -it $CID fortio load -c 1 -qps 1000 -t 10s -a -r 0.00005 -httpbufferkb=128 "$POD_IP:9080/productpage" >>test_result/enable_ebpf_performence_test1000_times.log;done

# disable
bash unload.sh
CID=$(docker run -d fortio/fortio:latest_release server)
POD_IP=$(kubectl get pod -o wide | grep productpage | awk '{print $6}')
cat >test_result/disable_ebpf_performence_test1000_times.log;done
for i in {1..10};do docker exec -it $CID fortio load -c 1 -qps 1000 -t 10s -a -r 0.00005 -httpbufferkb=128 "$POD_IP:9080/productpage" >>test_result/disable_ebpf_performence_test1000_times.log;done
```
