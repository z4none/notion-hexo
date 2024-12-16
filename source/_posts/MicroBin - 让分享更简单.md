---
updated: '2024-11-22 18:03:00'
categories: share
excerpt: MicroBin是一个轻量级的在线文件分享工具，支持文本和文件的跨设备分享，无需登录即可使用。它具备多种安全功能，包括端到端加密、密码保护和自动过期删除，并支持Docker等多种部署方式。
date: '2024-11-22 00:00:00'
tags:
  - Software
urlname: microbin-make-share-easy
title: MicroBin - 让分享更简单
cover: 'https://s.z4none.me/blog/40a975bd3c71a772a9311e8d0cff2997.png'
---

高效的文件分享工具至关重要。今天，我想向大家推荐一个出色的在线文件分享工具 —— **Microbin**。



有时你需要向朋友分享文件或文本，但不想使用复杂的云存储服务。或者你需要临时分享一些敏感信息，希望在一定时间后自动删除。这时，一个轻量级、安全且易于使用的文件分享工具就显得尤为重要。


MicroBin无需登录即可实现文本和文件的跨设备分享。它具备URL重定向、自动文件过期、原始文件服务支持等功能，支持3种加密级别，并可通过多种方式部署，比如 Docker。 


以下是它的特点：

- 非常小巧，部署方便
- 安全性：支持端到端加密，可设置密码保护和阅后即焚
- 灵活部署：支持 Docker 容器化部署，也可直接在服务器上运行
- 自动过期：可设置文件的自动过期时间，到期后自动删除
- 多功能：支持文本、文件分享，URL短链重定向等功能
- 支持私有链接和公开链接
- 支持生成二维码

界面


![image.png](https://s.z4none.me/blog/4cda76a5c0af2b7278b93220bdb224cd.png)


[官方网站](https://microbin.eu/)


[https://github.com/szabodanika/microbin](https://github.com/szabodanika/microbin)


# 部署方式

1. 通过 Docker 部署

	```javascript
	bash <(curl -s https://microbin.eu/docker.sh)
	```

2. 通过 cargo 安装

	```javascript
	cargo install microbin
	```


不管哪种方式，都需要通过配置环境变量实现功能的自定义


# 环境变量配置


参数太多，参见


[https://microbin.eu/docs/installation-and-configuration/configuration](https://microbin.eu/docs/installation-and-configuration/configuration)


# 在 Serv00 上部署


首先使用 cargo 安装


因为 Serv00 无法添加系统服务，所以我们可以使用脚本运行，并通过 crontab 设置在系统重启后自动启动


```bash
# run-microbin.sh
pkill -f microbin 
 
# 启动微服务，并确保它在后台运行 
/home/z4none/.cargo/bin/microbin \ 
    --auth-admin-username admin \ 
    --auth-admin-password 123123 \ 
    --port 17788 \ 
    --bind 0.0.0.0 \ 
    --gc-days 28 \ 
    --default-expiry 24hour \ 
    --enable-burn-after \ 
    --enable-readonly \ 
    --max-file-size-encrypted-mb 128 \ 
    --max-file-size-unencrypted-mb 128 \ 
    --threads 2 \ 
    --wide \ 
    --qr \ 
    --hash-ids \ 
    --public-path https://you-address/ \ 
    & 
echo "Microbin has been started." 
 
exit 0                                                                                                         23        23,7          All
```


参数不进一步解释了，参见官方环境变量说明


```bash
# crontab -e
@reboot /home/<username>/run-microbin.sh
```


注意事项：

1. 请根据实际需求调整配置参数；
2. 确保在 Serv00 后台完成端口绑定；
3. 如需绑定域名，可在 Serv00 后台的 WWW Websites 中设置反向代理
