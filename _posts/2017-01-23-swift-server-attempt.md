---
layout: post
title: "开始尝试 Swift server 开发"
categories:
- Swift
tags:
- Swift

---

![Vapor](http://mpclarkson.github.io/assets/vapor.png)

Swift 语言发展的非常迅速，官方最近还专门成立了一个 `Server APIs` 的 group。今天尝试着使用 Swift 来写了一个服务端的小程序，我本身并没有太多服务端的开发经验，但还是为 Swift 能做服务器的开发而感到兴奋不已。

之所以会有写 server 端的程序的需求，是因为最近购买了一个自己的 vps，于是开始着手将博客迁移到自己服务器上。由于使用 jekyll 的博客系统，最终生成的都是静态的 html 文件，于是在服务器上安装了 nginx 来做服务器和网关。配置好 nginx 后，并将博客的域名服务地址改为指向 vps 的 ip 后，博客就已经正常运行了，但是接下来就碰到了另外一个问题。

## 问题
---

jekyll 是集成在 github 里的。由于之前是使用了 github 的服务，每当有新的博客文章写好，并以 git commit 形式 push 到相应的 github 仓库后，github 会做自动处理，然后生成最新的文章。但是换到自己的服务器后，这一切就需要自己来做了。如何能够自动化做到，每当有文章更新后，服务器上对应的博客内容也同样更新，便成了需要解决的问题。

## 解决思路及框架选择
---

明确了问题，那么就可以想办法来解决问题了。     

解决思路如下：

1. 当 git 库发生 push 后，通过 github 的 webhook 功能发出一个 http 调用到服务器。
2. 服务器收到这个 http 调用后，更新本地对应的博客 git 仓库内容，然后调用 jekyll 命令，生成最新的博客文件。

这就需要搭建一个 http server ，该 server 在收到 hook 请求之后，执行相应的更新脚本，完成 git 仓库更新以及 jekyll 命令的调用。我决定用 Swift 来完成这些工作。

## server 框架选择
---

Swift 发展到现在，已经出现了不少的 server 端框架，比较知名的有 [Perfect](https://github.com/PerfectlySoft/Perfect) 和 [Vapor](https://github.com/vapor/vapor)，也已经有文章比较详细的对比了这些框架的区别。对我来说， Vapor 的代码风格更加符合我的口味，于是便选择了 Vapor 来完成这次工作。

Vapor 提供的文档很详细，根据文档，新建一个对应的工程，接收一个指定链接 post 的 http 请求，并调用脚本完成任务。

完成主要功能的代码非常少：

```swift
import Vapor
import Foundation

#if os(Linux)
    typealias CommandProcess = Task
#else
    typealias CommandProcess = Process
#endif

let drop = Droplet()
drop.post("/hooker/updateBlog") { _ in
    let task = CommandProcess()
    task.launchPath = "/carl/bin/blogRefresh"
    task.launch()
    return JSON([:])
}
drop.run()
```

`/carl/bin/blogRefresh` 是我在服务上写好的一个 shell 脚本，功能就是更新 git 仓库，然后执行 `jekyll build` 命令生成最新的博客内容。

这里有一个坑，由于我是在 Mac 上开发，当我把写好的工程在 linux 服务器上编译时，就报错了，提示找不到 `Process`。最后才知道原来 linux 上的 swift 最新 release 版本的完成度并没有 Mac 平台上高。所以代码中使用了编译指令加上 typealias 来屏蔽了这个差异。

相应工程已经在我的 github 上：[https://github.com/webfrogs/blogHooker](https://github.com/webfrogs/blogHooker).在工程路径下执行以下命令，可以编译出 release 环境的可执行文件

    swift build -c release

文件位于工程跟路径下的 `.build/release` 文件夹下。

## 环境配置
---

Swift 工程编译完之后，直接执行编译好的可执行文件便可以启动服务。由于 Vapor 本身项目的限制，如果 http server 的服务不是启动在 80 端口或者是 443 端口（https 默认端口），则外界无法直接访问。需要用 nginx 做一个代理，在 nginx 的配置中设置相应的 http 代理。

nginx 代理配置
```
server {
	listen 80;
	server_name domain;

	location /hooker/updateBlog {
		      proxy_pass http://127.0.0.1:8080;
        	proxy_pass_header Server;
        	proxy_set_header Host $host;
        	proxy_set_header X-Real-IP $remote_addr;
        	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        	proxy_pass_header Server;
        	proxy_connect_timeout 3s;
        	proxy_read_timeout 10s;
	}
}
```

将 http://domain/hooker/updateBlog 这个请求代理到本地的 8080 端口服务，这个端口正是 hooker 这个 http 服务所启动的端口。

由于做为服务需要一直启动着，这里使用了 Vapor 官方推荐的 Supervisor  来管理 http 服务，具体可以查看相应[文档](https://vapor.github.io/documentation/deploy/supervisor.html)。

在这里我碰到了一些障碍，服务器上使用 rvm 来管理 ruby 相应的版本，在调用 jekyll 这个 ruby 的库时，碰到了一些环境上的问题，最终在 Supervisor 的服务配置上加上了下面配置得以解决

    environment=PATH="/usr/local/rvm/gems/ruby-2.3.3/bin:/usr/local/rvm/gems/ruby-2.3.3@global/bin:/usr/local/rvm/rubies/ruby-2.3.3/bin:%(ENV_PATH)s", GEM_HOME="/usr/local/rvm/gems/ruby-2.3.3", GEM_PATH="/usr/local/rvm/gems/ruby-2.3.3:/usr/local/rvm/gems/ruby-2.3.3@global"

所有设置没问题后，到 github 仓库的 setting 中将这个 webhook 地址填好，最后测试通过。

## 总结
---
这次只是写了一个非常简单的 Swift server 程序，虽然感觉 Swift 在 server 端只是在起步阶段，很多方面还不够成熟。但是可以使用自己熟悉的语言来写一些完成自己需求的服务器程序，就已经很激动人心了嘛～～
