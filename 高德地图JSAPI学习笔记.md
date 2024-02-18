



[toc]







# 概述

地图 JS API 2.0 是高德开放平台免费提供的第四代 Web 地图渲染引擎， 以 WebGL 为主要绘图手段，本着“更轻、更快、更易用”的服务原则，广泛采用了各种前沿技术，交互体验、视觉体验大幅提升，同时提供了众多新增能力和特性





## 官网

* 官网：https://ditu.amap.com/
* 开放平台：https://lbs.amap.com/
* 文档：https://lbs.amap.com/api/javascript-api-v2/summary
* 示例：https://lbs.amap.com/demo/list/js-api-v2









# 成为开放者

## 概述

想要使用高德地图，必须先注册成为开发者，获取key

安全密钥机制旨在提升用户对 key 的安全有效管理，降低明文传输被窃取的风险。 2021年12月02日后创建的 key 必须配备安全密钥一起使用



## 步骤

### 第一步：登录高德开放平台控制台

https://console.amap.com/

![image-20240218152957863](img/高德地图JSAPI学习笔记/image-20240218152957863.png)



选择登录方式





### 第二步：注册为开发者

登录后显示此界面

![image-20240218153110971](img/高德地图JSAPI学习笔记/image-20240218153110971.png)



选择认证方式

![image-20240218153228709](img/高德地图JSAPI学习笔记/image-20240218153228709.png)



成为个人开发者即可

![image-20240218153330246](img/高德地图JSAPI学习笔记/image-20240218153330246.png)





![image-20240218153711528](img/高德地图JSAPI学习笔记/image-20240218153711528.png)





![image-20240218153724993](img/高德地图JSAPI学习笔记/image-20240218153724993.png)





### 第三步：创建key

进入**应用管理**，**创建新应用**

https://console.amap.com/dev/key/app

![image-20240218153839486](img/高德地图JSAPI学习笔记/image-20240218153839486.png)





![image-20240218153853912](img/高德地图JSAPI学习笔记/image-20240218153853912.png)







![image-20240218153917827](img/高德地图JSAPI学习笔记/image-20240218153917827.png)





![image-20240218154030018](img/高德地图JSAPI学习笔记/image-20240218154030018.png)



点击添加key

![image-20240218154052161](img/高德地图JSAPI学习笔记/image-20240218154052161.png)





![image-20240218154246081](img/高德地图JSAPI学习笔记/image-20240218154246081.png)





#### 第四步：获取key和密钥

创建成功后，可获取 key 和安全密钥

![image-20240218154415422](img/高德地图JSAPI学习笔记/image-20240218154415422.png)











## 流量限制

![image-20240218152746708](img/高德地图JSAPI学习笔记/image-20240218152746708.png)













# JS API 安全密钥使用

## 概述

**在2021年12月02日以后申请的 key 需要配合你的安全密钥一起使用**

为了增强安全性，建议将安全密钥存储在服务器端，并通过服务器端生成地图 JS API 的请求 URL。这样可以避免将密钥直接以明文的方式暴露给 Web 端代码中，减少密钥被滥用的风险，如果你只是想便捷开发可以使用明文方式设置

秘钥使用有**代理服务器转发**和**明文方式设置**两种方式





## 通过代理服务器转发

以Nginx反向代理为例，参考以下三个location配置，进行反向代理设置，分别对应自定义地图、海外地图、Web 服务，其中自定义地图和海外地图如果没有使用到相关功能也可以不设置。需要将以下配置中的**「你的安全密钥」**六个字替换为你刚刚获取的securityJsCode安全密钥

```sh
server {
  listen       80;             #nginx端口设置，可按实际端口修改
  server_name  127.0.0.1;      #nginx server_name 对应进行配置，可按实际添加或修改
  # 自定义地图和海外地图如果没有使用到相关功能可以不设置，但是如果需要设置顺序要与示例一致
  # 自定义地图服务代理
  location /_AMapService/v4/map/styles {
    set $args "$args&jscode=你的安全密钥";
    proxy_pass https://webapi.amap.com/v4/map/styles;
  }
  # 海外地图服务代理
  location /_AMapService/v3/vectormap {
    set $args "$args&jscode=你的安全密钥";
    proxy_pass https://fmap01.amap.com/v3/vectormap;
  }
  # Web服务API 代理
  location /_AMapService/ {
    set $args "$args&jscode=你的安全密钥";
    proxy_pass https://restapi.amap.com/;
  }
}
```





JS API 使用`<script>`标签同步加载增加代理服务器设置脚本，设置代理服务器域名或地址，将下面示例代码中的**「你的代理服务器域名或地址」**替换为你的代理服务器域名或 ip 地址，其中_AMapService为代理请求固定前缀，不可省略或修改



脚本同步加载为例：

```html
<div id="container"></div>
<script type="text/javascript">
  window._AMapSecurityConfig = {
    serviceHost: "你的代理服务器域名或地址/_AMapService",
    //例如 ：serviceHost:'http://1.1.1.1:80/_AMapService',
  };
</script>
<script
  type="text/javascript"
  src="https://webapi.amap.com/maps?v=2.0&key=你申请的key值"
></script>
<script type="text/javascript">
  //地图初始化应该在地图容器div已经添加到DOM树之后
  var map = new AMap.Map("container", {
    zoom: 12,
  });
</script>
```





## **通过明文方式设置**

JS API 安全密钥以明文方式设置，不建议在生产环境使用，不安全

JS API 使用`<script>`标签同步加载增加安全密钥设置脚本，并将**「你申请的安全密钥」**替换为你的安全密钥



```html
<div id="container"></div>
<script type="text/javascript">
  window._AMapSecurityConfig = {
    securityJsCode: "「你申请的安全密钥」",
  };
</script>
<script
  type="text/javascript"
  src="https://webapi.amap.com/maps?v=2.0&key=你申请的key值"
></script>
<script type="text/javascript">
  //地图初始化应该在地图容器div已经添加到DOM树之后
  var map = new AMap.Map("container", {
    zoom: 12,
  });
</script>
```









# 入门程序

## 第一步：配置nginx

配置文件如下，需要替换秘钥

```sh
#配置运行Nginx进程生成的worker进程数
worker_processes 2;
#配置Nginx服务器运行对错误日志存放的路径
error_log ./logs/error.log;
#配置Nginx服务器允许时记录Nginx的master进程的PID文件路径和名称
pid ./logs/nginx.pid;
#配置Nginx服务是否以守护进程方法启动
#daemon on;



events{
	#设置Nginx网络连接序列化
	accept_mutex on;
	#设置Nginx的worker进程是否可以同时接收多个请求
	multi_accept on;
	#设置Nginx的worker进程最大的连接数
	worker_connections 1024;
	#设置Nginx使用的事件驱动模型
	#use epoll;
}


http{
	#定义MIME-Type
	include mime.types;
	default_type application/octet-stream;

server {
        listen          8089;
        server_name     localhost;

          # 自定义地图和海外地图如果没有使用到相关功能可以不设置，但是如果需要设置顺序要与示例一致
  # 自定义地图服务代理
  location /_AMapService/v4/map/styles {
    set $args "$args&jscode=你的安全密钥";
    proxy_pass https://webapi.amap.com/v4/map/styles;
  }
  # 海外地图服务代理
  location /_AMapService/v3/vectormap {
    set $args "$args&jscode=你的安全密钥";
    proxy_pass https://fmap01.amap.com/v3/vectormap;
  }
  # Web服务API 代理
  location /_AMapService/ {
    set $args "$args&jscode=你的安全密钥";
    proxy_pass https://restapi.amap.com/;
  }
   }
}
```





## 第二步：检查配置

命令：

```sh
nginx -t
```





```sh
PS D:\opensoft\nginx-1.24.0\conf> cd ..
PS D:\opensoft\nginx-1.24.0> ls


    目录: D:\opensoft\nginx-1.24.0


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2023/5/17     14:24                cache
d-----         2023/5/12     14:54                conf
d-----         2023/4/11     23:31                contrib
d-----         2023/4/11     23:31                docs
d-----         2023/5/18     14:11                file
d-----          2023/5/3     14:45                html
d-----         2023/5/12     14:54                key
d-----         2023/5/19     14:10                logs
d-----         2023/5/19     13:58                static
d-----         2023/4/29     13:29                temp
-a----         2023/4/11     23:29        3811328 nginx.exe


PS D:\opensoft\nginx-1.24.0> .\nginx.exe -t
nginx: the configuration file D:\opensoft\nginx-1.24.0/conf/nginx.conf syntax is ok
nginx: configuration file D:\opensoft\nginx-1.24.0/conf/nginx.conf test is successful
PS D:\opensoft\nginx-1.24.0>
```





## 第三步：启动

前台启动：

```sh
PS D:\opensoft\nginx-1.24.0> ./nginx
```





## 第四步：HTML页面准备



```html
<!DOCTYPE html>


<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>高德地图入门程序</title>
    <style>
        html,
        body,
        #container {
            width: 100%;
            height: 100%;
        }
    </style>
</head>
<body>
<div id="container"></div>
</body>

<script>

    window._AMapSecurityConfig = {
        serviceHost: 'http://127.0.0.1:8089/_AMapService',
    };

</script>

<script
        type="text/javascript"
        src="https://webapi.amap.com/maps?v=2.0&key=318d115b92dd7beb6e26260a2e208256"
></script>

<script>
    //地图初始化应该在地图容器div已经添加到DOM树之后
    var map = new AMap.Map("container", {
        zoom: 12,
    });
</script>
</html>
```



![image-20240218163032784](img/高德地图JSAPI学习笔记/image-20240218163032784.png)







另一种写法：

```html
<script type="text/javascript">
  window._AMapSecurityConfig = {
    securityJsCode: "「你申请的安全密钥」",
  };
</script>
<script src="https://webapi.amap.com/loader.js"></script>
<script type="text/javascript">
  AMapLoader.load({
    key: "「你申请的应用Key」", //申请好的Web端开发者 Key，调用 load 时必填
    version: "2.0", //指定要加载的 JS API 的版本，缺省时默认为 1.4.15
  })
    .then((AMap) => {
      const map = new AMap.Map("container");
    })
    .catch((e) => {
      console.error(e); //加载错误提示
    });
</script>
```







## 第五步：添加标记点Marker

```html
<!DOCTYPE html>


<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>高德地图入门程序</title>
    <style>
        html,
        body,
        #container {
            width: 100%;
            height: 100%;
        }
    </style>
</head>
<body>
<div id="container"></div>
</body>

<script>

    window._AMapSecurityConfig = {
        serviceHost: 'http://127.0.0.1:8089/_AMapService',
    };

</script>

<script
        type="text/javascript"
        src="https://webapi.amap.com/maps?v=2.0&key=318d115b92dd7beb6e26260a2e208256"
></script>

<script>
    //地图初始化应该在地图容器div已经添加到DOM树之后
    var map = new AMap.Map("container", {
        zoom: 12,
    });

    const marker =new AMap.Marker({
        position:[116.25789,39.2456]
    })
    map.add(marker);
    const marker2 =new AMap.Marker({
        position:[112.58756,23.57865]
    })
    map.add(marker2);

</script>
</html>
```





![image-20240218163549633](img/高德地图JSAPI学习笔记/image-20240218163549633.png)







## 第六步：添加折线

```html
<!DOCTYPE html>


<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>高德地图入门程序</title>
    <style>
        html,
        body,
        #container {
            width: 100%;
            height: 100%;
        }
    </style>
</head>
<body>
<div id="container"></div>
</body>

<script>

    window._AMapSecurityConfig = {
        serviceHost: 'http://127.0.0.1:8089/_AMapService',
    };

</script>

<script
        type="text/javascript"
        src="https://webapi.amap.com/maps?v=2.0&key=318d115b92dd7beb6e26260a2e208256"
></script>

<script>
    //地图初始化应该在地图容器div已经添加到DOM树之后
    var map = new AMap.Map("container", {
        zoom: 12,
    });

    const marker = new AMap.Marker({
        position: [116.25789, 39.2456]
    })
    map.add(marker);
    const marker2 = new AMap.Marker({
        position: [112.58756, 23.57865]
    })
    map.add(marker2);

    const line = [[115.2578, 26.2568], [115.257, 33.2586], [114.257, 39.25777], [112.258, 26.255465]];
    const polyline = new AMap.Polyline({
        path: line, //设置线覆盖物路径
        strokeColor: "#3366FF", //线颜色
        strokeWeight: 5, //线宽
        strokeStyle: "solid", //线样式
    });
    map.add(polyline);

</script>
</html>

```



![image-20240218164635968](img/高德地图JSAPI学习笔记/image-20240218164635968.png)









# 整合vue

## vue2

