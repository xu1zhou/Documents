docker run -d fortio/fortio:latest_release server

curl 192.168.219.91:9080

CID="e9389c429e86"
docker exec -it $CID fortio load -c 1 -qps 100 -t 10s -a -r 0.00005 -httpbufferkb=128 "192.168.219.91:9080"


