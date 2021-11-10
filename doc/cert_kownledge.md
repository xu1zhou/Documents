# 自签名证书

## 非对称加密
密钥分为公私key，public key是公开的，用于加密传送的数据,server端收到消息后用私钥解码
>实际因为使用public key加密开销很大，加密使用的是协商出来的短密钥？
## 证书签名原理
server端将自己的密钥（指public key）送到第三方CA进行注册，这样client收到public key时可以像第三方查询确认此public key的可靠性，从而保证其不会被伪造

签名流程： 
1. server端用自己的私钥（公私钥）？？生成一个CSR（证书签发请求）------》第三方
2. 第三方将自己的私钥加入到csr里？参考以下命令，ca.crt, ca.key 都需要，最终生成server的证书
```
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in core.harbor.domain.csr \
    -out core.harbor.domain.crt
```
## 自签名证书
尽管理论上只需要服务端的公私钥就可以进行加密，但开启https是强制需要证书认证的，因此如果内部环境想要使用https就需要自己扮演第三方CA为自己的密钥做证书
## CA
ca是

## 要生成的几个文件
1. ca.key
2. ca.crt
3. server 证书
   - yourdomain.com.key
      - 由key生成yourdomain.com.csr
      - x509 v3.ext： Regardless of whether you’re using either an FQDN or an IP address to connect to your Harbor host, you must create this file so that you can generate a certificate for your Harbor host that complies with the Subject Alternative Name (SAN) and x509 v3 extension requirements. Replace the DNS entries to reflect your domain.
  -  v3.ext + ca.crt + ca.key + yourdomain.com.csr 生成 yourdomain.com.crt
    

## 服务器证书签发流程
- 因为重新签发服务器证书也要用到ca.key,
```
openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=core.harbor.com" \
 -key ca.key \
 -out ca.crt
# 一步生成自签名证书生成命令，合并生成ca.key，此命令不含ext
openssl req -x509 -newkey rsa:2048 -keyout a/front-proxy-key.pem -out  a/front-proxy-crt.pem -days 3650 -nodes -subj '/CN=front-envoy'
```
- yourdomain.com.key (似乎可以不用变，一直使用)
- csr （似乎也不需要变，会加入组织信息，对测试无影响）
```
openssl req -sha512 -new \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=core.harbor.domain" \
    -key core.harbor.domain.key \
    -out core.harbor.domain.csr
```
- v3.ext（要生成，并加入ip的SAN alt_names）
```
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=core.harbor.domain
DNS.2=192.168.1.100
EOF
```
- 生成新的crt
```
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in core.harbor.domain.csr \
    -out core.harbor.domain.crt
```
- 替换secret 中的crt

## 给harbor配置证书
上面生成的一大堆东西，真正要用到的是：
- yourdomain.com.crt (v3.ext + ca.crt + ca.key + yourdomain.com.csr 生成)
- yourdomain.com.key

harbor 需要的
```
cp yourdomain.com.crt /data/cert/
cp yourdomain.com.key /data/cert/
```


## 给docker配置证书
一下都存在secret中
- ca.crt(ca.key 生成,在secret中可以找到)
- yourdomain.com.cert(由yourdomain.com.crt转化) 
  - openssl x509 -inform PEM -in tls.crt -out tls.cert
- tls.key

## 查看证书内容
openssl x509 -in client.cert -noout -text 

