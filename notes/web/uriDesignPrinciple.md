---
layout: post
notes: true
subtitle: "【技术博客】URI设计原则，你设计的API做到了么"
comments: false
author: "Guy Levin（架构师之路译）"
date: 2017-09-24 00:00:00

---

原文：[http://blog.restcase.com/7-rules-for-rest-api-uri-design/](http://blog.restcase.com/7-rules-for-rest-api-uri-design/)

译文：[http://www.sohu.com/a/167341480_178889](http://www.sohu.com/a/167341480_178889)

*   目录
{:toc }

# 咱们设计的REST API真的nice么

**优雅型：** http://api.exapmle.com/louvre/da-vinci/mona-lisa

卢浮宫/达芬奇/蒙娜丽莎

**中庸型：** http://58.com/bj/ershou/310976

北京/二手频道/帖子ID

**谢特型：** http://api.example.com/68dd0-a9d3-11e0-9f1c

不知道什么鬼

# 1. URI的末尾不要添加"/"

多一个斜杠，语义完全不同，究竟是目录，还是资源，还是不确定而多做一次301跳转？

**负面case：** http://api.canvas.com/shapes/

**正面case：** http://api.canvas.com/shapes

# 2. 使用"-"提高URI的可读性

目的是使得URI便于理解，用“-”来连接单词

**正面case：** http://api.example.com/blogs/my-first-post

# 3. 禁止在URL中使用"_"

目的是提高可读性，"_"可能被文本查看器中的下划线特效遮蔽

**负面case：** http://api.example.com/blogs/my_first_post

# 4. 禁止使用大写字母

RFC 3986中规定URI区分大小写，但别用大写字母来为难程序员了，既不美观，又麻烦

**负面case：** http://api.example.com/My-Folder/My-Doc

**正面case：** http://api.example.com/my-folder/my-doc

# 5. 不要在URI中包含扩展名

应鼓励REST API客户端使用HTTP提供的格式选择机制Accept request header

**正面case：** http://58.com/bj/ershou/310976

**一个case：** http://58.com/bj/ershou/310976x.shtml

# 6. 建议URI中的名称使用复数

**正面case：** http://api.college.com/students/3248234/courses

**负面case：** http://api.college.com/student/3248234/course

最后，给后端研发工程师一个建议：清晰优雅的 RESTful API是为调用者编写的，别无脑随意定义一些shit一样的URI给移动/前端工程师使用，小心生命有危险。