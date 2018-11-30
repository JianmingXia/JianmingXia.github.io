---
title: API Blueprint教程及Demo
date: 2018/11/30 21:00:00
tags:
  - API Blueprint
categories: 技术杂集
---

## 说在前面
这篇文章本应是年初的输出了，后由于工作的原因一拖再拖。当时之所以想到使用API Blueprint的原因是由于之前的后端接口没有一个规范的API文档，最初做曲奇的时候，几个小伙伴的角色都是全栈工程师，而且由于职责明确，基本上都是自己写出的后端服务，自己写客户端去调用。而随着时间的发展，安卓、IOS开发同学以及新鲜血液的加入，以及一些小小的人员变动，导致客户端不清楚服务端的返回到底是什么样，也因此引发了一些线上问题，这时api blueprint浮现在我们的视野中。完善的规范能够免除自定义接口规范的痛苦，不仅可以按照API Blueprint规范去书写API文档，而且还可以结合gitlab hook输出最终的API文档，诱惑不要太大。

最近我又重新开始了后端开发，自然也要书写好API文档了，于是又将忘记了快一年的API Blueprint捡起来了。

<!-- more -->
> API blueprint 以下称 APIB

## 基础定义
### Metadata
```plain
FORMAT: 1A

# Polls

Polls is a simple API allowing consumers to view polls and vote in them.
```

APIB从一个Metadata开始，format关键字表示API蓝图的版本

### API Name & Description
APIB 中的第一个标题作为API的名称，标题以一个或多个#开头，在本例中是“Polls”。

* #的数量将决定标题的级别
* 标题后面是API的描述

### Resource Groups
使用Group关键字开头的标题，可以用于创建一组相关资源：

```plain
# Group Questions

Resources related to questions in the API.
```

### Resource
Resource作为Resource Groups的子集存在，The heading specifies the URI used to access the resource inside of square brackets at the end of the heading

```plain
## Question Collection [/questions]
```

### Actions
Action是用资源中的子标题指定的，该子标题后面跟着HTTP方法名

```plain
### List All Questions [GET]
```

每个action至少包括一个响应：必须要有的状态码以及可能包含的body。比如返回200状态码及一个JSON body：

```plain
+ Response 200 (application/json)

        [
            {
                "question": "Favourite programming language?",
                "published_at": "2014-11-11T08:40:51.620Z",
                "url": "/questions/1",
                "choices": [
                    {
                        "choice": "Swift",
                        "url": "/questions/1/choices/1",
                        "votes": 2048
                    }, {
                        "choice": "Python",
                        "url": "/questions/1/choices/2",
                        "votes": 1024
                    }, {
                        "choice": "Objective-C",
                        "url": "/questions/1/choices/3",
                        "votes": 512
                    }, {
                        "choice": "Ruby",
                        "url": "/questions/1/choices/4",
                        "votes": 256
                    }
                ]
            }
        ]
```

Note：返回的参数类型在状态码之后，无需设置Content-Type header

另外，action可以支持多个：

```plain
### Create a New Question [POST]

You may create your own question using this action. It takes a JSON object
containing a question and a collection of answers in the form of choices.

+ question (string) - The question
+ choices (array[string]) - A collection of choices.

+ Request (application/json)

            {
                "question": "Favourite programming language?",
                "choices": [
                    "Swift",
                    "Python",
                    "Objective-C",
                    "Ruby"
                ]
            }
```

Post请求返回一个201的状态码，另外还有HTTP headers 及 Body：

```plain
+ Response 201 (application/json)

    + Headers

            Location: /questions/1

    + Body

                {
                    "question": "Favourite programming language?",
                    "published_at": "2014-11-11T08:40:51.620Z",
                    "url": "/questions/1",
                    "choices": [
                        {
                            "choice": "Swift",
                            "url": "/questions/1/choices/1",
                            "votes": 0
                        }, {
                            "choice": "Python",
                            "url": "/questions/1/choices/2",
                            "votes": 0
                        }, {
                            "choice": "Objective-C",
                            "url": "/questions/1/choices/3",
                            "votes": 0
                        }, {
                            "choice": "Ruby",
                            "url": "/questions/1/choices/4",
                            "votes": 0
                        }
                    ]
                }
```

__此时还可以继续下一个Resource：__

```plain
## Question [/questions/{question_id}]
```

### URI Parameters
> 详见：[https://github.com/apiaryio/api-blueprint/blob/master/API%20Blueprint%20Specification.md#def-uriparameters-section](https://github.com/apiaryio/api-blueprint/blob/master/API%20Blueprint%20Specification.md#def-uriparameters-section)

URI 参数使用参数列表来描述URI，比如：

```plain
+ Parameters
    + question_id (number) - ID of the Question in the form of an integer
```



## 合理的文档结构



![image.png | left | 378x743](https://cdn.nlark.com/yuque/0/2018/png/92822/1543580361790-46b4c82c-04a2-45d7-9a2b-26c444bd5237.png "")


## DEMO
按照[文章](https://apiblueprint.org/documentation/tutorial.html)的描述写出的APIB 文档：

### 文档
```plain
FORMART: 1A

# Polls

Polls is a simple API allowing consumers to view polls and vote in them.

# Group Questions

Resources related to questions in the API.

## Question Collection [/questions]

### List All Questions [GET]

+ Response 200 (application/json)

        [
            {
                "question": "Favourite programming language?",
                "published_at": "2014-11-11T08:40:51.620Z",
                "url": "/questions/1",
                "choices": [
                    {
                        "choice": "Swift",
                        "url": "/questions/1/choices/1",
                        "votes": 2048
                    }, {
                        "choice": "Python",
                        "url": "/questions/1/choices/2",
                        "votes": 1024
                    }, {
                        "choice": "Objective-C",
                        "url": "/questions/1/choices/3",
                        "votes": 512
                    }, {
                        "choice": "Ruby",
                        "url": "/questions/1/choices/4",
                        "votes": 256
                    }
                ]
            }
        ]
        
### Create a New Question [POST]

You may create your own question using this action. It takes a JSON object
containing a question and a collection of answers in the form of choices.

+ question (string) - The question
+ choices (array[string]) - A collection of choices.

+ Request (application/json)

            {
                "question": "Favourite programming language?",
                "choices": [
                    "Swift",
                    "Python",
                    "Objective-C",
                    "Ruby"
                ]
            }
            
+ Response 201 (application/json)

    + Headers

            Location: /questions/1

    + Body

                {
                    "question": "Favourite programming language?",
                    "published_at": "2014-11-11T08:40:51.620Z",
                    "url": "/questions/1",
                    "choices": [
                        {
                            "choice": "Swift",
                            "url": "/questions/1/choices/1",
                            "votes": 0
                        }, {
                            "choice": "Python",
                            "url": "/questions/1/choices/2",
                            "votes": 0
                        }, {
                            "choice": "Objective-C",
                            "url": "/questions/1/choices/3",
                            "votes": 0
                        }, {
                            "choice": "Ruby",
                            "url": "/questions/1/choices/4",
                            "votes": 0
                        }
                    ]
                }

## Question [/questions/{question_id}]

+ Parameters
    + question_id (number) - ID of the Question in the form of an integer

### View a Questions Detail [GET]

+ Response 200 (application/json)

            {
                "question": "Favourite programming language?",
                "published_at": "2014-11-11T08:40:51.620Z",
                "url": "/questions/1",
                "choices": [
                    {
                        "choice": "Swift",
                        "url": "/questions/1/choices/1",
                        "votes": 2048
                    }, {
                        "choice": "Python",
                        "url": "/questions/1/choices/2",
                        "votes": 1024
                    }, {
                        "choice": "Objective-C",
                        "url": "/questions/1/choices/3",
                        "votes": 512
                    }, {
                        "choice": "Ruby",
                        "url": "/questions/1/choices/4",
                        "votes": 256
                    }
                ]
            }
            
### Delete [DELETE]

+ Response 204
```

### 效果



![image.png | left | 827x369](https://cdn.nlark.com/yuque/0/2018/png/92822/1543582453287-2ad3b073-66ba-427f-9e44-667c625c888d.png "")



## 说在后面
看了下上次博客写的时间，是10月26号，距今天已经一个多月了，如果是以往，我现在内心肯定又是慌的要死，感叹时间蹉跎之类的。但是这次不会，自己明白这段时间自己在做什么，这一生还很长，如果不想随随便便地过完这一生，那就只能珍惜时间，好好学习——深刻相信，知识会给我想要的东西。

## 资料
* [https://apiblueprint.org/](https://apiblueprint.org/)
* [https://apiblueprint.org/documentation/tutorial.html](https://apiblueprint.org/documentation/tutorial.html)
* [https://github.com/apiaryio/api-blueprint/blob/master/API%20Blueprint%20Specification.md](https://github.com/apiaryio/api-blueprint/blob/master/API%20Blueprint%20Specification.md)
* [https://github.com/apiaryio/api-blueprint/tree/master/examples](https://github.com/apiaryio/api-blueprint/tree/master/examples)
* [https://github.com/apiaryio/api-blueprint/blob/master/Glossary%20of%20Terms.md](https://github.com/apiaryio/api-blueprint/blob/master/Glossary%20of%20Terms.md)

