---
layout: post
title: Linux命令-MySql json column数据去重处理
date: 2015-11-15 14:55:29
tags: [Linux命令]
published: True

---
##先说需求
公司某项目数据存储在MySql中值是一个Json字符串
例如:

```json
{
    "main_position": {
        "name_en": " Centre Back "
    }, 
    "side_position": [
        {
            "name_en": "Defensive Midfield "
        }, 
        {
            "name_en": "Left-Back "
        }
    ]
}
```
现要将name_en这个字段做去重处理,看看到底有多少种类型

##解决方法

1.写代码,通过程序去读这个字段值,然后做去重处理

2.通过linux命令做去重处理

1和2都能实现此功能,但是从速度上来说(不是性能哦,具体性能方面我没有做过测试),2这种方式比较简便,那我们按照第2种方式来实现

##具体实现

1.将数据库字段值导出到一个文本文件

```bash
mysql -uroot -p -h192.168.9.42 -e "select position from table_name where position is not null" db_name > data.txt
```

导出的文件内容如下

```
position
{"main_position":{"name_en":" Centre Back "}}
{"main_position":{"name_en":" Keeper "}}
{"main_position":{"name_en":" Keeper "}}
{"main_position":{"name_en":" Centre Back "},"side_position":[{"name_en":"Right-Back "}]}
{"main_position":{"name_en":" Keeper "}}
{"main_position":{"name_en":" Keeper "}}
{"main_position":{"name_en":" Keeper "}}
{"main_position":{"name_en":" Keeper "}}
{"main_position":{"name_en":" Keeper "}}
......
......
......
```

2.逐行解析json字符串

linux处理json命令有好多,我这里选择[jq](http://stedolan.github.io/jq/),具体安装方法详见这里[安装方法](https://stedolan.github.io/jq/download/)

注:jq版本大于等于1.4,不然filter中?号用不了

```bash
awk ' NR != 1 {print $0}' data.txt | jq ".main_position.name_en, .side_position[]?.name_en" |sed -e 's/^"*\s*//g' -e 's/\s*"*$//g'|sort -u
```

解释一下:

* awk ' NR != 1 {print $0}' data.txt 主要是去逐行读取刚才从数据库中导出的数据,从第二行开始读

* jq ".main_position.name_en, .side_position[]?.name_en" 将main_position下的name_en字段值和side_position数组下的name_en值解析出来

* sed -e 's/^"*\s*//g' -e 's/\s*"*$//g' 将解析出值中的双引号和前后空格替换为空字符串

* sort -u 排序去重

##Show Time

看一下最终执行结果

![结果](http://7xnxev.com1.z0.glb.clouddn.com/2015-11-15/2015-11-15%2016%3A10%3A52.png)

比写代码快多了吧
