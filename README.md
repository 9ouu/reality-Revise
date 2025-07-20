## 原作者
`https://github.com/howmp/reality`

## 说明

基于源代码修改，为socks5添加帐号密码。

## 使用步骤

### 生成配置、客户端、用户端

`grss gen  --user admin --pass 123123 www.qq.com:443 127.0.0.1:443`

1. `www.qq.com:443` 是被模拟的目标
1. `127.0.0.1:443` 是服务器监听地址，这里要填写公网IP，端口最好和模拟目标一致

若SNIAddr或ServerAddr不指定，则尝试加载已有配置文件

默认生成3个不同id文件名的客户端，可通过`-c`参数指定

```txt
Usage:
  grss [OPTIONS] gen [gen-OPTIONS] [SNIAddr] [ServerAddr]

generate server config and client

Help Options:
  -h, --help                                                 Show this help message

[gen command options]
      -d                                                     debug
      -f=[chrome|firefox|safari|ios|android|edge|360|qq]     client finger print (default: chrome)
      -e=                                                    expire second (default: 30)
      -o=                                                    server config output path (default: config.json)
      -c=                                                    client count (default: 3)
      -s                                                     skip client cert verify
          --dir=                                             client output directory (default: .)
          --user=                                            SOCKS5 username
          --pass=                                            SOCKS5 password

[gen command arguments]
  SNIAddr:                                                   tls server address, e.g. example.com:443
  ServerAddr:                                                server address, e.g. 8.8.8.8:443
```

### 启动服务端

`grss serv`

```txt
Usage:
  grss [OPTIONS] serv [serv-OPTIONS]

run server

Help Options:
  -h, --help      Show this help message

[serv command options]
      -o=         server config path (default: config.json)
```

### 启动客户端

`grscX`

**X表示id**

### 启动用户端

`grsu -id 0`

**这里id参数对应了grsc的id，不同id会连接不同的grsc**

```txt
Usage of grsu:
  -i uint
        id
  -l string
        socks5 listen address (default "127.0.0.1:61080")
```

## 常见问题

### 服务端被探测时使用的“真证书”吗?

是，准确的说被探测时，服务端相当于一个端口转发，证书与被模拟的目标完全一致

这样一点可以通过修改本地Hosts文件后，通过浏览器访问来验证

或通过curl验证: `curl -v -I --resolve "www.qq.com:443:127.0.0.1" https://www.qq.com`

### 为什么客户端/用户端提示`verify failed`?

1. 服务端时间和客户端时间相差超过`expire second`
   1. 为了防重放，默认不能相差30秒，可在生成时修改最大超时时间`grss gen -e 60 www.qq.com:443 127.0.0.1:443`
   1. 也可以NTP同步客户端、用户端、服务端时间
1. 服务端配置重新生成后，也需要使用最新的`grsc`和`grsu`，否则预共享密钥不匹配
1. 客户端的网络可能被劫持

### 为什么客户端/用户端提示`certificate signed by unknown authority`?

运行环境缺少根证书，可以生成时指定`-s`选项，跳过验证

`grss gen -s www.qq.com:443 127.0.0.1:443`
