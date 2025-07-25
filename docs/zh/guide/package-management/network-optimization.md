# 网络优化 {#network-optimization}

如果由于网络不稳定，导致下载包速度很慢或者下载失败，我们可以通过下面的一些方式来解决。

## 手动下载 {#manual-download}

默认 xmake 会调用 curl、wget 等工具来下载，用户也可以手动用自己的下载器下载（也可以使用代理），把下载后的包放到自己的目录下，比如：`/download/packages/zlib-v1.0.tar.gz`

然后使用下面的命令，设置包下载的搜索目录：

```sh
$ xmake g --pkg_searchdirs="/download/packages"
```

然后重新执行 xmake 编译时，xmake 会优先从 `/download/packages` 找寻源码包，然后直接使用，不再自己下载了。

至于查找的包名是怎样的呢，可以通过下面的命令查看：

```sh
$ xmake require --info zlib
-> searchdirs: /download/packages
-> searchnames: zlib-1.2.11.tar.gz
```

我们可以看到对应的搜索目录以及搜索的包名。

## 设置代理 {#proxy-setting}

如果觉得手动下载还是麻烦，我们也可以让 xmake 直接走代理。

```sh
$ xmake g --proxy="socks5://127.0.0.1:1086"
$ xmake g --help
    -x PROXY, --proxy=PROXY  Use proxy on given port. [PROTOCOL://]HOST[:PORT]
                                 e.g.
                                 - xmake g --proxy='http://host:port'
                                 - xmake g --proxy='https://host:port'
                                 - xmake g --proxy='socks5://host:port'
```

--proxy 参数指定代理协议和地址，具体语法可以参考 curl 的，通常可以支持 http、https、socks5 等协议，但实际支持力度依赖 curl、wget 和 git，比如 wget 就不支持 socks5 协议。

我们可以通过下面的参数指定哪些 host 走代理，如果没设置，默认全局走代理。

```sh
--proxy_hosts=PROXY_HOSTS    Only enable proxy for the given hosts list, it will enable all if be unset,
                             and we can pass match pattern to list:
                                 e.g.
                                 - xmake g --proxy_hosts='github.com,gitlab.*,*.xmake.io'
```

如果设置了 hosts 列表，那么之后这个列表里面匹配的 host 才走代理。

--proxy_host 支持多个 hosts 设置，逗号分隔，并且支持基础的模式匹配 *.github.com，以及其他 lua 模式匹配规则也支持。

如果觉得上面的 hosts 模式配置还不够灵活，我们也可以走 pac 的自动代理配置规则：

```sh
--proxy_pac=PROXY_PAC    Set the auto proxy configuration file. (default: pac.lua)
                                     e.g.
                                     - xmake g --proxy_pac=pac.lua (in /Users/ruki/.xmake or absolute path)
                                     - function main(url, host)
                                           if host == 'github.com' then
                                                return true
                                           end
                                       end
```

::: tip 注意
如果有proxy_hosts优先走hosts配置，没有的话才走pac配置。
:::

pac 的默认路径：~/.xmake/pac.lua，如果 --proxy 被设置，并且这个文件存在，就会自动走 pac，如果不存在，也没 hosts，那就全局生效代理。

也可以手动指定 pac 全路径。

```sh
$ xmake g --proxy_pac=/xxxx/xxxxx_pac.lua
```

配置规则描述：

```lua
function main(url, host)
    if host:find("bintray.com") then
        return true
    end
end
```

如果返回 true，那么这个 url 和 host 就是走的代理，不返回或者返回 false，就是不走代理。

这块的具体详情见：https://github.com/xmake-io/xmake/issues/854

::: tip 注意
另外，除了依赖包下载，其他涉及网络下载的命令也都支持代理，比如：`xmake update`
:::

## 镜像代理 {#mirror-proxy}

v2.5.4 之后，pac.lua 配置里面还可以配置镜像代理规则，比如对所有 github.com 域名的访问切到 hub.fastgit.org 域名，实现加速下载包。

```lua
function mirror(url)
     return url:gsub("github.com", "hub.fastgit.org")
end
```

```sh
$ xrepo install libpng
> curl https://hub.fastgit.org/glennrp/libpng/archive/v1.6.37.zip -o v1.6.37.zip
```