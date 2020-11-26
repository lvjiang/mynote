## 一、简介

​		wrk 是一款针对 Http 协议的基准测试工具，它能够在单机多核 CPU 的条件下，使用系统自带的高性能 I/O 机制，如 epoll，kqueue 等，通过多线程和事件模式，对目标机器产生大量的负载。



**优势：**

* 轻量级性能测试工具
* 安装简单
* 学习曲线基本为0，几分钟就学会使用了
* 基于系统自带的高性能I/O机制，如epoll，kqueue，利用异步的事件驱动框架，通过很少的线程就可以压出很大的并发量，例如几万、几十万，这是很多性能测试工具无法做到的。

**劣势：**

* wrk 目前仅支持单机压测，后续也不太可能支持多机器对目标机压测，因为它本身的定位，并不是用来取代 JMeter, LoadRunner 等专业的测试工具。



## 二、下载安装

wrk是开源的, 代码在 github 上：https://github.com/wg/wrk

```
lvsj@lvsj-machine:~/study$ git clone https://github.com/wg/wrk.git
Cloning into 'wrk'...
remote: Enumerating objects: 1085, done.
remote: Total 1085 (delta 0), reused 0 (delta 0), pack-reused 1085
Receiving objects: 100% (1085/1085), 27.42 MiB | 20.00 KiB/s, done.
Resolving deltas: 100% (336/336), done.
Checking connectivity... done.

lvsj@lvsj-machine:~/study$ cd wrk/
lvsj@lvsj-machine:~/study/wrk$ ls
azure-piplines.yml  CHANGES  deps  INSTALL  LICENSE  Makefile  NOTICE  README.md  SCRIPTING  scripts  src
lvsj@lvsj-machine:~/study/wrk$ ls src/
ae.c        ae_evport.c  ae_kqueue.c  aprintf.c  atomicvar.h  http_parser.c  main.h  net.h     script.h  ssl.h    stats.h  units.h  wrk.h    zmalloc.c
ae_epoll.c  ae.h         ae_select.c  aprintf.h  config.h     http_parser.h  net.c   script.c  ssl.c     stats.c  units.c  wrk.c    wrk.lua  zmalloc.h
lvsj@lvsj-machine:~/study/wrk$ ls scripts/
addr.lua  auth.lua  counter.lua  delay.lua  pipeline.lua  post.lua  report.lua  setup.lua  stop.lua
lvsj@lvsj-machine:~/study/wrk$ ls deps/
LuaJIT-2.1.0-beta3.tar.gz  openssl-1.1.1b.tar.gz

lvsj@lvsj-machine:~/study/wrk$ make
```



## 三、格式及用法

```
lvsj@lvsj-machine:~/study/wrk$ ./wrk
Usage: wrk <options> <url>                            
  Options:                                            
    -c, --connections <N>  Connections to keep open   
    -d, --duration    <T>  Duration of test           
    -t, --threads     <N>  Number of threads to use   
                                                      
    -s, --script      <S>  Load Lua script file       
    -H, --header      <H>  Add header to request      
        --latency          Print latency statistics   
        --timeout     <T>  Socket/request timeout     
    -v, --version          Print version details      
                                                      
  Numeric arguments may include a SI unit (1k, 1M, 1G)
  Time arguments may include a time unit (2s, 2m, 2h)

```

翻译成中文：

```
使用方法: wrk <选项> <被测HTTP服务的URL>                           

  Options:                                           
    -c, --connections <N>  跟服务器建立并保持的TCP连接数量 
    -d, --duration    <T>  压测时间          
    -t, --threads     <N>  使用多少个线程进行压测，压测时，是有一个主线程来控制我们设置的n个子线程间调度  
                                                    
    -s, --script      <S>  指定Lua脚本路径      
    -H, --header      <H>  为每一个HTTP请求添加HTTP头     
        --latency          在压测结束后，打印延迟统计信息  
        --timeout     <T>  超时时间    
    -v, --version          打印正在使用的wrk的详细版本信                                              

  <N>代表数字参数，支持国际单位 (1k, 1M, 1G)
  <T>代表时间参数，支持时间单位 (2s, 2m, 2h)
```



## 四、简单压测及结果分析

```
lvsj@lvsj-machine:~/study/wrk$ ./wrk -t8 -c200 -d30s --latency  http://www.bing.com
Running 30s test @ http://www.bing.com
  8 threads and 200 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   181.00ms  168.17ms   1.64s    84.84%
    Req/Sec   166.18    127.87   460.00     54.10%
  Latency Distribution
     50%   68.21ms
     75%  321.19ms
     90%  357.20ms
     99%  669.63ms
  35839 requests in 30.10s, 10.08MB read
  Socket errors: connect 0, read 0, write 0, timeout 39
Requests/sec:   1190.72
Transfer/sec:    342.85KB
```

以上是使用8个线程200个连接，对bing首页进行了30秒的压测，并要求在压测结果中输出响应延迟信息。

以下是解释压测结果：

```
Running 30s test @ http://www.bing.com （压测时间30s）

  8 threads and 200 connections （共8个测试线程，200个连接）

  Thread Stats   Avg      Stdev     Max   +/- Stdev
              （平均值） （标准差）（最大值）（正负一个标准差所占比例）
    Latency    181.00ms  168.17ms   1.64s    84.84%
    （延迟）
    Req/Sec    166.18    127.87   460.00     54.10%
    （处理中的请求数）

  Latency Distribution （延迟分布）
     50%   68.21ms
     75%  321.19ms
     90%  357.20ms
     99%  669.63ms （99分位的延迟：%99的请求在1.35s以内）
  35839 requests in 30.10s, 10.08MB read （30.10秒内共处理完成了35839个请求，读取了10.08MB数据）
Requests/sec:  1190.72 （平均每秒处理完成1190.72个请求）
Transfer/sec:     342.85KB （平均每秒读取数据342.85KB）
```



## 五、使用lua脚本进行压测

​	lua脚本是一种轻量小巧的脚本语言，用标准c语言编写，并以源代码形式开放，其设计目的是为了嵌入应用程序中，从而为程序提供灵活的扩展和定制功能。wrk工具嵌入了lua脚本语言，因此，在自定义压测场景时，可在wrk目录下使用lua定制压测场景。

​	

### 5.1 lua环境

lua的官网地址：http://www.lua.org

wrk内置了lua解释器，不用单独的去安装lua环境。但是测试脚本中，会使用一些第三方库，所以需要安装好，lua的第三方库。



#### 5.1.1 LuaRocks

 LuaRocks 是 Lua 模块的安装和部署工具，官网：https://luarocks.org

**源码安装方式**

```
wget http://luarocks.org/releases/luarocks-2.0.13.tar.gz
cd luarocks-2.0.13

./configure
--prefix=/usr/local/luarocks/
--rocks-tree=/usr/local
--sysconfdir=/usr/local/etc/luarocks

make

install
```

"rocks-tree" 是指所要安装的 Lua 模块的默认安装目录，"sysconfdir" 是指 LuaRocks 的配置文件存放的地方



**Ubuntu安装**

```
lvsj@lvsj-machine:~/study/lua-5.4.1$ sudo apt install luarocks
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  apport liblua5.1-0 liblua5.1-0-dev libtool-bin lua5.1 python-samba
The following NEW packages will be installed:
  liblua5.1-0 liblua5.1-0-dev libtool-bin lua5.1 luarocks
The following packages will be upgraded:
  apport python-samba
2 upgraded, 5 newly installed, 0 to remove and 243 not upgraded.
37 not fully installed or removed.
Need to get 480 kB/1,662 kB of archives.
After this operation, 2,388 kB of additional disk space will be used.
Do you want to continue? [Y/n] y

```



#### 5.1.2 安装第三方库

```
lvsj@lvsj-machine:~/study/lua-5.4.1$ sudo luarocks install md5
Installing https://rocks.moonscript.org/md5-1.3-1.rockspec...
Using https://rocks.moonscript.org/md5-1.3-1.rockspec... switching to 'build' mode
gcc -O2 -fPIC -I/usr/include/lua5.1 -c src/compat-5.2.c -o src/compat-5.2.o -Isrc/
gcc -O2 -fPIC -I/usr/include/lua5.1 -c src/md5.c -o src/md5.o -Isrc/
gcc -O2 -fPIC -I/usr/include/lua5.1 -c src/md5lib.c -o src/md5lib.o -Isrc/
gcc -shared -o md5/core.so -L/usr/local/lib src/compat-5.2.o src/md5.o src/md5lib.o
gcc -O2 -fPIC -I/usr/include/lua5.1 -c src/compat-5.2.c -o src/compat-5.2.o -Isrc/
gcc -O2 -fPIC -I/usr/include/lua5.1 -c src/des56.c -o src/des56.o -Isrc/
gcc -O2 -fPIC -I/usr/include/lua5.1 -c src/ldes56.c -o src/ldes56.o -Isrc/
gcc -shared -o des56.so -L/usr/local/lib src/compat-5.2.o src/des56.o src/ldes56.o
Updating manifest for /usr/local/lib/luarocks/rocks
No existing manifest. Attempting to rebuild...
md5 1.3-1 is now built and installed in /usr/local (license: MIT/X11)
```



### 5.2 lua脚本的生命周期

共有三个阶段，启动阶段，运行阶段，结束阶段。wrk支持在这三个阶段对压测进行个性化

**启动阶段**

```
function setup(thread)
```

在脚本文件中实现setup方法，wrk就会在测试线程已经初始化但还没有启动的时候调用该方法。wrk会为每一个测试线程调用一次setup方法，并传入代表测试线程的对象thread作为参数。setup方法中可操作该thread对象，获取信息、存储信息、甚至关闭该线程。

```
thread.addr - get or set the thread's server address
thread:get(name) - get the value of a global in the thread's env
thread:set(name, value) - set the value of a global in the thread's env
thread:stop() - stop the thread
```



**运行阶段**

```
function init(args)  --由测试线程调用，只会在进入运行阶段时，调用一次。支持从启动wrk的命令中，获取命令行参数；
function delay()  --在每次发送request之前调用，如果需要delay，那么delay相应时间；
function request()  --用来生成请求；每一次请求都会调用该方法，所以注意不要在该方法中做耗时的操作；
function response(status, headers, body)  --在每次收到一个响应时调用；为提升性能，如果没有定义该方法，那么wrk不会解析headers和body；
```



** 结束阶段**

```
function done(summary, latency, requests)  --在整个测试过程中只会调用一次，可从参数给定的对象中，获取压测结果，生成定制化的测试报告。
```



### 5.3 自定义脚本中可访问的变量和方法

全局变量：wrk

```
wrk = {
    scheme  = "http",
    host    = "localhost",
    port    = nil,
    method  = "GET",
    path    = "/",
    headers = {},
    body    = nil,
    thread  = <userdata>,
  }
```



全局方法：wrk.fomat wrk.lookup wrk.connect

```
function wrk.format(method, path, headers, body)  --根据参数和全局变量wrk，生成一个HTTP rquest string。
function wrk.lookup(host, service)  --给定host和service（port/well known service name），返回所有可用的服务器地址信息。
function wrk.connect(addr)  --测试与给定的服务器地址信息是否可以成功创建连接
```



### 5.4 压测实例

压测命令：

​	wrk -t8 -c200 -d30s --latency -s test.lua http://www.bing.com



test.lua是用lua写的压测脚本，下面是压测脚本的实例。　

#### 5.4.1 请求前延时

每次请求前，延迟 10ms:

```
function delay()
   return 10
end
```



#### 5.4.2 post方法

```
wrk.method = "POST"
wrk.headers["S-COOKIE2"]="a=2&b=Input&c=10.0&d=20191114***"
wrk.body = "recent_seven=20191127_32;20191128_111"
wrk.headers["Host"]="api.shouji.**.com"
function response(status,headers,body)
        if status ~= 200 then --将服务器返回状态码不是200的请求结果打印出来
                print(body)
        --      wrk.thread:stop()
        end
end
```



#### 5.4.3 发送json

```
request = function()
    local headers = { }
    headers['Content-Type'] = "application/json"
    body = {
        mobile={"1533899828"},
        params={code=math.random(1000,9999)}
    }
    local cjson = require("cjson")
    body_str = cjson.encode(body)
    return wrk.format('POST', nil, headers, body_str)
end
```



#### 5.4.4 读取文件

实现随机header-cookie

```
idArr = {}
falg = 0
wrk.method = "POST"
wrk.body = "a=1"
function init(args)
        for line in io.lines("integral/cookies.txt") do
                print(line)
                idArr[falg] = line
                falg = falg+1
        end
        falg = 0
end

--wrk.method = "POST"
--wrk.body = "a=1"
--wrk.path = "/v1/points/reading"

request = function()
        parms = idArr[math.random(0,4)] --随机传递文件中的参数
        --parms = idArr[falg%(table.getn(idArr)+1)] 循环传递文件中的参数
        wrk.headers["S-COOKIE2"] = parms
        falg = falg+1
        return wrk.format()
end
```



#### 5.4.5 随机参数

wrk创建数组并初始化，拼接随机参数

```
idArr = {};
function init(args)
        idArr[1] = "1";
        idArr[2] = "2";
        idArr[3] = "3";
        idArr[4] = "4";
end

request = function()
        parms = idArr[math.random(1,4)]
        path = "/v1/points/reading?id="..parms
        return wrk.format("GET",path)
end
```



#### 5.4.6 MD5校验

```
md5 = require "md5"
request = function()
	local headers = {}
	local time = os.time()
	headers["Scan-Id"] = "6512ea34f32663528144a310496ca71a"
	headers["Scan-Tt"] = time
	headers["Scan-Sign"] = md5.sumhexa(time)

	return wrk.format('GET', nil, headers, nil)
end
```



#### 5.4.7 认证

请求的接口需要先进行认证，获取 token 后，再发起请求。

下面的脚本表示，在 token 为空的情况下，先请求 `/auth` 接口来认证，获取 token, 拿到 token 以后，将 token 放置到请求头中，再请求真正需要压测的 `/test` 接口。执行过程，第一次执行path的值是`\auth`,响应中判断token为空，获取到token,并设置全局头部信息和pathd值`/test`, 第二次请求的时候，就带着认证信息了

```
token = nil
path  = "/auth"

request = function()
   return wrk.format("GET", path)
end

response = function(status, headers, body)
   if not token and status == 200 then
      token = headers["X-Token"]
      path  = "/test"
      wrk.headers["X-Token"] = token
   end
end
```



#### 5.4.8 HTTP pipeline

通过在 init 方法中将三个 HTTP请求拼接在一起，实现每次发送三个请求，以使用 HTTP pipeline。·

```
init = function(args)
   local r = {}
   r[1] = wrk.format(nil, "/?foo")
   r[2] = wrk.format(nil, "/?bar")
   r[3] = wrk.format(nil, "/?baz")

   req = table.concat(r)
end

request = function()
   return req
end
```



#### 5.4.9 自定义结果

```
rimap = {
    "/v3/goods/detail", 
    "/v3/goods/status",
    "/v3/goods/sku_detail",
    "/v3/coupons/lists",
    "/v3/pages/recommend",
    "/v3/carts",
}

methodmap = {
    "POST", 
    "POST",
    "POST",
    "GET",
    "GET",
    "GET",
}

params = {
      [[{"id":2}]], 
      [[{"spu_id":2,"type":1}]],
      [[{"id":2,"sku_id":7}]], -- 双中括号里面不转译
      "page=1&size=100",  
      "",
      "",
}

math.randomseed(os.time())

init = function()
    local r = {}
    local path = ""   -- 局部变量（不加local 是全局变量）
    local method = "get" -- 默认get

    -- header 头
    wrk.headers["Hash"]= "85280aa135bbd0108dd6aa424565a"
    wrk.headers["Token"]= ""
   
    for i, v in ipairs(urimap) do -- 键从1 开始 非 0
        path = v    -- 路径
        method = methodmap[i]  -- method

        if method == "POST" then
            wrk.headers["content-type"]= "application/json" --POST 参数json格式
            wrk.body = params[i]
        end

        if method == "GET" and  params[i] ~= "" then
            path = v .. "?" ..params[i]
        end

        io.write(method, "---", params[i], "----", path, "\n") -- 打印请求方式（1个线程会打印一次），参数，路径（不含域名）
        r[i] =  wrk.format(method, path)    
    end 

    req = table.concat(r)
end

request = function()
      return req
end
    
response = function(status, headers, body)  
    if status ~= 200 then
        print("status:", status)
        print("error:", body)
        wrk.thread:stop()
    else 
       -- print("body:", body)   
    end
end  

done = function(summary, latency, requests)

    local durations=summary.duration / 1000000    -- 执行时间，单位是秒
    local errors=summary.errors.status            -- http status不是200，300开头的
    local requests=summary.requests               -- 总的请求数
    local valid=requests-errors                   -- 有效请求数=总请求数-error请求数
  
    io.write("Durations:       "..string.format("%.2f",durations).."s".."\n")
    io.write("Requests:        "..summary.requests.."\n")
    io.write("Avg RT:          "..string.format("%.2f",latency.mean / 1000).."ms".."\n")
    io.write("Max RT:          "..(latency.max / 1000).."ms".."\n")
    io.write("Min RT:          "..(latency.min / 1000).."ms".."\n")
    io.write("Error requests:  "..errors.."\n")
    io.write("Valid requests:  "..valid.."\n")
    io.write("QPS:             "..string.format("%.2f",valid / durations).."\n")
    io.write("--------------------------\n")
  
end
```

```
wrk -t1 -c1000 -d30s -T10s -s category.lua --latency https://shop-api-crs.chi.com  

-t 开启多少线程
-c 连接数量 
-T 时间参数，支持时间单位 (2s, 2m, 2h)
-s Lua脚本路径  
--latency 压测结束完，打印统计信息  

返回：

Running 15s test @ https://shop-api-crs.chi.com 
  1 threads and 10 connections. 
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   124.50ms  173.08ms   1.11s    47.38%
    Req/Sec   252.53    111.40   363.00     79.86%
  Latency Distribution
     50%  381.93ms
     75%    0.00us
     90%    0.00us
     99%    0.00us
  3674 requests in 15.08s, 2.51MB read  --（15.08秒内共处理完成了 3674个请求，读取了2.51MB数据）
Requests/sec:    243.56    -- （平均每秒处理完成请求）
Transfer/sec:    170.29KB  -- （平均每秒读取数据）
Durations:       15.08s     -- 执行时间，单位是秒
Requests:        3674   -- 总的请求数
Avg RT:          124.50ms  --平均值
Max RT:          1110.714ms  -- 最大值
Min RT:          78.324ms   -- 最小值
Error requests:  0       -- Error请求数
Valid requests:  3674  -- 有效请求数
QPS:             243.56   --QPS
```

