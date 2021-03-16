

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

#性能测试
docker run -d fortio/fortio:latest_release server

curl 192.168.219.91:9080

./bpftool prog show | grep sock
CID="e9389c429e86"
docker exec -it $CID fortio load -c 1 -qps 1000 -t 10s -a -r 0.00005 -httpbufferkb=128 "192.168.219.91:9080"


