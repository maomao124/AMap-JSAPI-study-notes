



[toc]







# 概述

地图 JS API 2.0 是高德开放平台免费提供的第四代 Web 地图渲染引擎， 以 WebGL 为主要绘图手段，本着“更轻、更快、更易用”的服务原则，广泛采用了各种前沿技术，交互体验、视觉体验大幅提升，同时提供了众多新增能力和特性





## 官网

* 官网：https://ditu.amap.com/
* 开放平台：https://lbs.amap.com/
* 文档：https://lbs.amap.com/api/javascript-api-v2/summary
* 示例：https://lbs.amap.com/demo/list/js-api-v2
* 手册：https://lbs.amap.com/api/javascript-api-v2/documentation





<iframe id="doc2" onload="changeHash()" src="https://a.amap.com/jsapi/static/doc/20230922/index.html?v=2" style="font-family: PingFangSC-Regular, &quot;Microsoft YaHei&quot;, &quot;Open Sans&quot;, Arial, &quot;Hiragino Sans GB&quot;, 微软雅黑, STHeiti, &quot;WenQuanYi Micro Hei&quot;, SimSun, sans-serif; box-sizing: border-box; border-top: none; border-right: none; border-bottom: 1px solid rgb(240, 240, 240); border-left: none; border-image: initial; width: 1220px; height: calc(-48px + 100vh); border-radius: 4px; box-shadow: none; margin-bottom: 28px;"></iframe>









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





### 第四步：获取key和密钥

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



### 第一步：创建项目

![image-20240218175903646](img/高德地图JSAPI学习笔记/image-20240218175903646.png)



### 第二步：安装依赖

```sh
npm i @amap/amap-jsapi-loader --save
```



```sh
PS D:\程序\2024Q1\vue-integration-amap> npm i @amap/amap-jsapi-loader --save
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@2.3.3 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@2.3.3: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

+ @amap/amap-jsapi-loader@1.0.1
added 1 package from 1 contributor and audited 971 packages in 3.294s

95 packages are looking for funding
  run `npm fund` for details

found 2 moderate severity vulnerabilities
  run `npm audit fix` to fix them, or `npm audit` for details
PS D:\程序\2024Q1\vue-integration-amap>
```



可以看到package.json多了一项：

![image-20240301173925308](img/高德地图JSAPI学习笔记/image-20240301173925308.png)





### 第三步：新建文件

在sr/view目录下新建一个vue文件

![image-20240301172245841](img/高德地图JSAPI学习笔记/image-20240301172245841.png)





### 第四步：创建地图容器

```vue
<template>
    <div id="container"></div>
</template>

<script>
export default {
    
}
</script>

<style scoped>

</style>

```





### 第五步：设置样式

```vue
<template>
    <div id="container"></div>
</template>

<script>
export default {
    
}
</script>

<style scoped>
#container {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 800px;
}
</style>

```





### 第六步：导入依赖

```vue
<template>
    <div id="container"></div>
</template>

<script>
import AMapLoader from '@amap/amap-jsapi-loader';

export default {

}
</script>

<style scoped>
#container {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 800px;
}
</style>

```





### 第七步：完善代码

```vue
<template>
    <div id="container"></div>
</template>

<script>
import AMapLoader from '@amap/amap-jsapi-loader';

export default {
    mounted()
    {
        window._AMapSecurityConfig = {
            serviceHost: `http://127.0.0.1:8089/_AMapService`
        }
        this.initAMap();
    },
    unmounted()
    {
        this.map?.destroy();
    },
    methods:
        {
            /**
             * 初始化高德地图
             */
            initAMap()
            {
                AMapLoader.load({
                    key: "318d115b92dd7beb6e26260a2e208256", // 申请好的Web端开发者Key，首次调用 load 时必填
                    version: "2.0", // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
                    // 需要使用的的插件列表，如比例尺'AMap.Scale'等
                    plugin: [
                        'AMap.Autocomplete', // 输入提示插件
                        'AMap.PlaceSearch', // POI搜索插件
                        'AMap.Scale', // 右下角缩略图插件 比例尺
                        'AMap.OverView', // 地图鹰眼插件
                        'AMap.ToolBar', // 地图工具条
                        'AMap.Geolocation', // 定位控件，用来获取和展示用户主机所在的经纬度位置
                        'AMap.MarkerClusterer',
                        'AMap.Geocoder'
                    ],
                })
                    .then((AMap) =>
                    {
                        this.map = new AMap.Map("container", {
                            // 设置地图容器id
                            viewMode: "3D", // 是否为3D地图模式
                            zoom: 12, // 初始化地图级别
                            center: [111.397428, 23.90923], // 初始化地图中心点位置
                        });
                    })
                    .catch((e) =>
                    {
                        console.log(e);
                    });
            },
        },
}
</script>

<style scoped>
#container {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 800px;
}
</style>

```







### 第八步：添加点

```vue
<template>
    <div>
        <div id="container"></div>
        <div>
            <button @click="addPoint">添加点</button>
            <button @click="clear">清空</button>
        </div>
    </div>
</template>

<script>
import AMapLoader from '@amap/amap-jsapi-loader';

export default {
    mounted()
    {
        window._AMapSecurityConfig = {
            serviceHost: `http://127.0.0.1:8089/_AMapService`
        }
        this.initAMap();
    },
    unmounted()
    {
        this.map?.destroy();
    },
    methods:
        {
            /**
             * 初始化高德地图
             */
            initAMap()
            {
                AMapLoader.load({
                    key: "318d115b92dd7beb6e26260a2e208256", // 申请好的Web端开发者Key，首次调用 load 时必填
                    version: "2.0", // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
                    // 需要使用的的插件列表，如比例尺'AMap.Scale'等
                    plugin: [
                        'AMap.Autocomplete', // 输入提示插件
                        'AMap.PlaceSearch', // POI搜索插件
                        'AMap.Scale', // 右下角缩略图插件 比例尺
                        'AMap.OverView', // 地图鹰眼插件
                        'AMap.ToolBar', // 地图工具条
                        'AMap.Geolocation', // 定位控件，用来获取和展示用户主机所在的经纬度位置
                        'AMap.MarkerClusterer',
                        'AMap.Geocoder'
                    ],
                })
                    .then((AMap) =>
                    {
                        this.map = new AMap.Map("container", {
                            // 设置地图容器id
                            viewMode: "3D", // 是否为3D地图模式
                            zoom: 12, // 初始化地图级别
                            center: [111.397428, 23.90923], // 初始化地图中心点位置
                        });
                    })
                    .catch((e) =>
                    {
                        console.log(e);
                    });
            },
            addPoint()
            {
                const marker = new AMap.Marker({
                    position: [100 + Math.random() * 22, 22 + Math.random() * 20]
                })
                this.map.add(marker);
            },
            clear()
            {
                this.map.clearMap();
            }
        },
}
</script>

<style scoped>
#container {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 800px;
}
</style>

```



![image-20240301175949887](img/高德地图JSAPI学习笔记/image-20240301175949887.png)







## vue3



### 第一步：创建项目

![image-20240313142441847](img/高德地图JSAPI学习笔记/image-20240313142441847.png)



![image-20240313142456088](img/高德地图JSAPI学习笔记/image-20240313142456088.png)





### 第二步：安装依赖

```sh
npm i @amap/amap-jsapi-loader --save
```



```sh

```



### 第三步：新建文件

在sr/view目录下新建一个vue文件







### 第四步：创建地图容器

```vue
<template>
    <div id="container"></div>
</template>

<script setup>

</script>

<style scoped>

</style>

```





### 第五步：设置样式

```vue
<template>
    <div id="container"></div>
</template>

<script setup>

</script>

<style scoped>
#container {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 800px;
}
</style>

```





### 第六步：导入依赖

```vue
<template>
    <div id="container"></div>
</template>

<script setup>
import AMapLoader from '@amap/amap-jsapi-loader';

</script>

<style scoped>
#container {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 800px;
}
</style>

```





### 第七步：完善代码

```vue
<template>
    <div id="container"></div>
</template>

<script setup>
import AMapLoader from '@amap/amap-jsapi-loader';
import {onBeforeMount, onMounted, onUnmounted} from 'vue'

onBeforeMount(()=>
{
    window._AMapSecurityConfig = {
        serviceHost: `http://127.0.0.1:8089/_AMapService`
    }
})

onMounted(()=>
{
    initAMap();
})

onUnmounted(()=>
{
    this.map?.destroy();
})

/**
 * 初始化高德地图
 */
function initAMap()
{
    AMapLoader.load({
        key: "318d115b92dd7beb6e26260a2e208256", // 申请好的Web端开发者Key，首次调用 load 时必填
        version: "2.0", // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
        // 需要使用的的插件列表，如比例尺'AMap.Scale'等
        plugin: [
            'AMap.Autocomplete', // 输入提示插件
            'AMap.PlaceSearch', // POI搜索插件
            'AMap.Scale', // 右下角缩略图插件 比例尺
            'AMap.OverView', // 地图鹰眼插件
            'AMap.ToolBar', // 地图工具条
            'AMap.Geolocation', // 定位控件，用来获取和展示用户主机所在的经纬度位置
            'AMap.MarkerClusterer',
            'AMap.Geocoder'
        ],
    })
        .then((AMap) =>
        {
            this.map = new AMap.Map("container", {
                // 设置地图容器id
                viewMode: "3D", // 是否为3D地图模式
                zoom: 12, // 初始化地图级别
                center: [111.397428, 23.90923], // 初始化地图中心点位置
            });
        })
        .catch((e) =>
        {
            console.log(e);
        });
}

</script>

<style scoped>
#container {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 800px;
}
</style>

```









## 封装

在src/utils下创建`amapUtils.ts`

```typescript
import AMapLoader from '@amap/amap-jsapi-loader';


const defaultPlugin: Array<string> = [
    'AMap.Autocomplete', // 输入提示插件
    'AMap.PlaceSearch', // POI搜索插件
    'AMap.Scale', // 右下角缩略图插件 比例尺
    'AMap.OverView', // 地图鹰眼插件
    'AMap.ToolBar', // 地图工具条
    'AMap.Geolocation', // 定位控件，用来获取和展示用户主机所在的经纬度位置
    'AMap.MarkerClusterer',
    'AMap.Geocoder'
];

/**
 * 初始化地图
 * @param that vue视图的this指针
 * @param pluginList 插件列表
 * @param key 高德地图的key
 */
export function initAMap(that: any, pluginList: Array<string>, key: string): any
{
    window._AMapSecurityConfig = {
        serviceHost: `http://127.0.0.1:8089/_AMapService`
    }
    const plugin = pluginList ? pluginList : defaultPlugin
    AMapLoader.load({
        key: key ? key : "318d115b92dd7beb6e26260a2e208256", // 申请好的Web端开发者Key，首次调用 load 时必填
        version: "2.0", // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
        // 需要使用的的插件列表，如比例尺'AMap.Scale'等
        plugin: plugin,
    })
        .then((AMap: any) =>
        {
            const map = new AMap.Map("container", {
                // 设置地图容器id
                viewMode: "3D", // 是否为3D地图模式
                zoom: 12, // 初始化地图级别
                center: [111.397428, 23.90923], // 初始化地图中心点位置
            });
            that.map = map;
        })
        .catch((e: Error) =>
        {
            console.log('加载失败：', e);
        });
}

/**
 * 销毁地图
 * @param map 高德地图容器对象
 */
export function destroy(map: any): void
{
    map?.destroy();
}

```





在src目录下创建`env.d.ts`

```typescript
declare module "@amap/amap-jsapi-loader";
```



在src目录下创建`global.d.ts`

```typescript

// src/global.d.ts
export {}

declare global {
    interface Window {
        _AMapSecurityConfig: any;
    }
}

declare const window: any;

```



使用示例：

```vue
<template>
    <div id="container"></div>
</template>

<script>
import {initAMap, destroy} from '../utils/amapUtils'

export default {
    name: "View1",
    data()
    {
        return {
            map: null,
        }
    },
    methods: {},
    created()
    {
        console.log("初始化")
        initAMap(this);
    },
    beforeDestroy()
    {
        console.log("销毁")
        destroy(this.map)
    },
}
</script>

<style scoped>
#container {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 90vh;
}
</style>

```













# 入门

## 图层

### 概述

JS API 支持官方图层、三方图层、数据图层等

实时交通路况图层`AMap.TileLayer.Traffic`



1. 标准图层 [TileLayer](https://lbs.amap.com/api/jsapi-v2/documentation#tilelayer)
2. 卫星图层 [TileLayer.Satellite](https://lbs.amap.com/api/jsapi-v2/documentation#Satellite)
3. 路网图层 [TileLayer.RoadNet](https://lbs.amap.com/api/jsapi-v2/documentation#RoadNet)
4. 实时交通图层 [TileLayer.Traffic](https://lbs.amap.com/api/jsapi-v2/documentation#Traffic)
5. 楼块图层 [Buildings](https://lbs.amap.com/api/jsapi-v2/documentation#buildings)
6. 室内图层 [IndoorMap](https://lbs.amap.com/api/jsapi-v2/documentation#indoormap)



### 使用

创建图层

```js
const layer = new AMap.createDefaultLayer({
  zooms: [3, 20], //可见级别
  visible: true, //是否可见
  opacity: 1, //透明度
  zIndex: 0, //叠加层级
});
```



加载图层

```js
const map = new AMap.Map("container", {
  viewMode: "2D", //默认使用 2D 模式
  zoom: 11, //地图级别
  center: [116.397428, 39.90923], //地图中心点
  layers: [layer], //layer为创建的默认图层
});
```



```vue
<template>
    <div>
        <div id="container"></div>
        <button @click="addLayer">加载图层</button>
    </div>
</template>

<script>
import {initAMap, destroy} from '../utils/amapUtils'

export default {
    name: "View2",
    data()
    {
        return {
            map: null,
        }
    },
    methods: {
        addLayer()
        {
            const layer = new AMap.createDefaultLayer({
                zooms: [3, 20], //可见级别
                visible: true, //是否可见
                opacity: 1, //透明度
                zIndex: 0, //叠加层级
            });
            this.map.setLayers(layer);
        },
    },
    created()
    {
        console.log("初始化")
        initAMap(this);
    },
    beforeDestroy()
    {
        console.log("销毁")
        destroy(this.map)
    },
}
</script>

<style scoped>
#container {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 90vh;
}
</style>

```







### **实时交通路况图层**

**AMap.TileLayer.Traffic**

| **参数/方法** | **说明**     | **类型** | **参数值描述**           | **默认值** |
| ------------- | ------------ | -------- | ------------------------ | ---------- |
| autoRefresh   | 是否自动刷新 | Boolean  | true \| false            | false      |
| interval      | 刷新间隔     | Number   | 自动更新数据间隔的毫秒数 | 180        |



添加图层使用map.add(traffic)，移除路况图层：map.remove(traffic)



```vue
<template>
    <div>
        <div id="container"></div>
        <button @click="addLayer">加载图层</button>
        <button @click="addTraffic">创建实时交通路况图层</button>
        <button @click="showTraffic">显示路况图层</button>
        <button @click="hideTraffic">隐藏路况图层</button>
    </div>
</template>

<script>
import {initAMap, destroy} from '../utils/amapUtils'

export default {
    name: "View2",
    data()
    {
        return {
            map: null,
            traffic: null,
        }
    },
    methods: {
        addLayer()
        {
            const layer = new AMap.createDefaultLayer({
                zooms: [3, 20], //可见级别
                visible: true, //是否可见
                opacity: 1, //透明度
                zIndex: 0, //叠加层级
            });
            this.map.setLayers(layer);
        },
        addTraffic()
        {
            if (this.traffic)
            {
                return;
            }
            const traffic = new AMap.TileLayer.Traffic({
                autoRefresh: true, //是否自动刷新，默认为false
                interval: 60, //刷新间隔，默认180s
            });
            this.traffic = traffic;
            this.map.add(traffic);
        },
        showTraffic()
        {
            if (this.traffic)
            {
                this.traffic.show();
            }
        },
        hideTraffic()
        {
            if (this.traffic)
            {
                this.traffic.hide();
            }
        }
    },
    created()
    {
        console.log("初始化")
        initAMap(this);
    },
    beforeDestroy()
    {
        console.log("销毁")
        destroy(this.map)
    },
}
</script>

<style scoped>
#container {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 90vh;
}
</style>
```





![image-20240319145059408](img/高德地图JSAPI学习笔记/image-20240319145059408.png)



显示后：

![image-20240319145112008](img/高德地图JSAPI学习笔记/image-20240319145112008.png)







### 卫星与路网图层

```vue
<template>
    <div>
        <div id="container"></div>
        <button @click="addTraffic">创建卫星与路网图层</button>
        <button @click="showTraffic">显示图层</button>
        <button @click="hideTraffic">隐藏图层</button>
    </div>
</template>

<script>
import {initAMap, destroy} from '../utils/amapUtils'

export default {
    name: "View3",
    data()
    {
        return {
            map: null,
            traffic: null,
        }
    },
    methods: {
        addTraffic()
        {
            if (this.traffic)
            {
                return;
            }
            const traffic = new AMap.TileLayer.Satellite();
            //创建路网图层
            var roadNet = new AMap.TileLayer.RoadNet();
            this.traffic = traffic;
            this.map.setLayers([traffic, roadNet]);
        },
        showTraffic()
        {
            if (this.traffic)
            {
                this.traffic.show();
            }
        },
        hideTraffic()
        {
            if (this.traffic)
            {
                this.traffic.hide();
            }
        }
    },
    created()
    {
        console.log("初始化")
        initAMap(this);
    },
    beforeDestroy()
    {
        console.log("销毁")
        destroy(this.map)
    },
}
</script>

<style scoped>
#container {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 90vh;
}
</style>

```



![image-20240319154347709](img/高德地图JSAPI学习笔记/image-20240319154347709.png)





![image-20240319154416612](img/高德地图JSAPI学习笔记/image-20240319154416612.png)





