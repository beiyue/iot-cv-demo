---
layout: post
title:  "数据存储与展现"
toc: true


---

通过IoT Core Rule Egine连接其他aws服务.本实验将数据存入ElasticSearch并通过Kibana进行展现

## 1. 部署ElasticSearch

### 1.1 通过ElasticSearch建立ES域

- 在AWS官网通过搜索进入Elasticsearch Service。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/1.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/1.png)

- 在Elasticsearch Service首页选择“创建新域”。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/2.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/2.png)

- 选择部署类型为“开发和测试”，Elasticsearch 选择版本为“6.7”，点击下一步。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/3.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/3.png)

- 配置Elasticsearch域名为“iot-lab”， 实例类型选择“r5.large.elasticsearch“，实例数量选择为1。如果有更小的实例也可以选择。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/4.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/4.png)

- 其余配置如果与图片中相同，则无需更改（关闭专用主实例和加密），检查完毕后直接点击下一步。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/5.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/5.png)

<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/6.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/6.png)

- 网络配置选择“公有访问权限”， 访问策略选择允许从特定IP访问域，一个窗口会弹出，您需要在窗口输入您的登陆电脑的公有IP或者对应的CIDR形式，公网IP可通过这个网站获取: [What Is My IP](https://www.whatismyip.com/)
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/6a.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/6a.png)

<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/7.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/7.png)

- 此时您的访问策略应该显示为如下形式, "aws:SourceIp":之后是您的IP地址，这里通常不需要进一步调整了。确认之后点击下一步。

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": "es:*",
      "Resource": "arn:aws-cn:es:cn-north-1:XXXXXXXXXXXX:domain/iot-lab/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "XXXXXXXXXXXX"
        }
      }
    }
  ]
}
```

- 检查之前的配置，点击确认，您将在控制面看见您的新域。稍等十分钟之后，您的域状态将变成绿色的有效。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/8.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/8.png)

- 点击“iot-lab“，您将进入域的详细页面。在此处您将看到Kibana链接，点击Kibana链接以访问Kibana网页，如果无法进入网页，请再次检查您的IP地址是否正确，如果有问题请修改访问策略。可使用CIDR表示方法达到更宽松的条件。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/9.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/9.png)

<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/10.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/10.png)

### 1.2 在Elasticsearch集群上建立索引

- 在这里我们使用HTTP请求来建立索引，请在终端输入如下命令，

```sh
curl -H 'Content-Type: application/json' -i -X PUT -d '{
  "mappings": {
    "car": {
      "properties": {
        "timestamp": {
          "type": "long",
          "copy_to": "datetime"
        },
        "datetime": {
          "type": "date",
          "store": true
        },
        "location": {
          "type": "geo_point"
        },
        "battery":{
        "type": "float"
        },
        "temperature":{
        "type": "short"
        },
        "pressure":{
        "type": "short"
        },
        "rpm":{
        "type": "short"
        }
      }
    }
  }
}
' 'https://search-iot-lab-xxxxxxxxxxxxxxxxxxxxx.cn-north-1.es.amazonaws.com.cn/cars'
```

请把最下面一行的

```sh
https://search-iot-lab-xxxxxxxxxxxxxxxxxxxxx.cn-north-1.es.amazonaws.com.cn/cars
```

替换为您的集群的终端节点，在“iot-lab”的详细页面可以找到：
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/11.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/11.png)

如果成功您将在终端收到状态“ {“acknowledged”:true)”。如果不成功，请检查Kibana能否进入。如果不能进入，请调整访问策略，比如使用xxx.xxx.xxx.xxx/16代替IP。

## 2. 设置规则将数据存入ES

### 2.1 创建IAM角色

- 打开AWS IAM服务，选择侧边栏的角色，点击创建角色
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/20.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/20.png)

- 在AWS产品中选择IoT，点击下一步：权限
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/21.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/21.png)

- 点击下一步：标签
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/22.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/22.png)

- 点击下一步：审核
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/23.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/23.png)

- 添加角色名称为：“iot-es-action-role“，点击创建角色
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/24.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/24.png)

### 2.2 创建IoT规则

- 进入IoT服务，进打开侧边栏的行动，点击右上角的创建
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/25.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/25.png)

- 添加规则名称为ElasticsearchLab
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/26.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/26.png)

- 在规则查询语句中输入

```SQL
SELECT *, timestamp() as timestamp FROM 'connectedcar/#'
```

<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/27.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/27.png)

- 点击添加操作，选择将消息发送到Amazon Elasticsearch Service，点击配置操作
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/28.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/28.png)

- 选择资源为"iot-lab", ID输入“${newuuid()}”，索引输入“cars“， 类型输入“car”。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/29.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/29.png)

- 选择角色为iot-es-action-role。 
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/30.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/30.png)

- 反复检查没有遗漏后，请创建规则。如果显示SQL错误而无法创建，请先将FROM 'connectedcar/#' 语句删除，在创建之后再更新规则
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/31.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/31.png)

<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/32.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/32.png)

## 3. 设置Kibana进行数据可视化

### 3.1 Kibana index pattern创建

- 进入Kibana节点，节点可从您的Elasticsearch控制面板找到。节点的格式如下

```sh
https://search-iot-lab-xxxxxxxxxxxxx.us-east-1.es.amazonaws.com/_plugin/kibana/
```

<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/33.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/33.png)

- 进入Kibana的discover侧边栏，选择Create index pattern，并在index pattern中输入“cars”。匹配成功后会显示Success，之后请点击Next step
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/40.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/40.png)

- 如果无法找到create index pattern，请点击Kibana的图标，然后选择Index patterns。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/32a.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/32a.png)

- 从下拉菜单中选择Time Filter field name为datetime。点击create index pattern。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/41.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/41.png)

- 您将看见刚刚创建的index pattern: cars*。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/42.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/42.png)

### 3.2 数据可视化

- 接下来进入Visualize侧边栏，选择create a visualization。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/43.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/43.png)

- 选择Line作为我们要创建的图的类型。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/44.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/44.png)

- 选择cars*。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/45.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/45.png)

- 打开Metrics Y-Axis的下拉菜单，在Aggregation中选择average，Field选择engine_tempterature,然后点击add metrics。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/46.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/46.png)

- 打开Buckets X-Axis的下拉菜单，在Aggregation中选择Date Histogram，Field选择datetime。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/47.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/47.png)

- 点击蓝色的开始按钮就将生成图像。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/48.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/48.png)

- 右上角的蓝色Refresh按钮可以刷新图像来预览最新数据。
<a data-fancybox="gallery" href="https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/49.png">
</a>

![Get started](https://iot-demo-resource.s3-ap-southeast-1.amazonaws.com/dashboard/49.png)
