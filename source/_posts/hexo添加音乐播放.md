---
title: hexo添加音乐播放
date: 2023-11-16 13:39:24
categories:
  - Tools
tags: [blog]
---


{% aplayerlist %}
{
    "narrow": false,
    "autoplay": true,
    "mode": "random",
    "showlrc": 3,
    "mutex": true,
    "theme": "#e6d0b2",
    "preload": "auto",
    "listmaxheight": "513px",
    "width": "50%",
    "music": [
        {
            "title": "Something Just Like This",
            "author": "Coldplay",
            "url": "/music/SomethingJustLikeThis.mp3",
            "pic": "/music/Coldplay.jpg",
            "lrc": "/music/SomethingJustLikeThis-Coldplay.lrc"
        },
        {
            "title": "发如雪",
            "author": "周杰伦",
            "url": "/music/发如雪.mp3",
            "pic": "/music/Coldplay.jpg",
            "lrc": "/music/发如雪-周杰伦.lrc"
        },
        {
            "title": "Viva La Vida",
            "author": "Coldplay",
            "url": "/music/VivaLaVida.mp3",
            "pic": "/music/Coldplay.jpg",
            "lrc": "/music/VivaLaVida-Coldplay.lrc"
        },
        {
          "title": "我记得",
          "author": "赵雷",
          "url": "/music/我记得.mp3",
          "pic": "/music/我记得.jpg",
          "lrc": "/music/我记得-赵雷.lrc"
        },
        {
          "title": "晚婚",
          "author": "江蕙",
          "url": "/music/晚婚.mp3",
          "pic": "/music/晚婚.jpg",
          "lrc": "/music/晚婚-江蕙.lrc"
        }
    ]
}
{% endaplayerlist %}


安装[hexo-tag-aplayer](https://github.com/MoePlayer/hexo-tag-aplayer) `npm i hexo-tag-aplayer`,在 `source/music` 目录中添加下载的歌曲，歌词，封面。这里给一个免费的[音乐下载网站](https://slider.kz/),在文章中写入如下内容即可:  

```
{% aplayer title author url [picture_url, narrow, autoplay, width:xxx, lrc:xxx] %}
```

下面给出一个播放列表的示例。<!-- more -->

```

{% aplayerlist %}
{
    "narrow": false,
    "autoplay": true,
    "mode": "random",
    "showlrc": 3,
    "mutex": true,
    "theme": "#e6d0b2",
    "preload": "auto",
    "listmaxheight": "513px",
    "width": "50%",
    "music": [
        {
            "title": "Something Just Like This",
            "author": "Coldplay",
            "url": "/music/SomethingJustLikeThis.mp3",
            "pic": "/music/Coldplay.jpg",
            "lrc": "/music/SomethingJustLikeThis-Coldplay.lrc"
        },
        {
            "title": "Viva La Vida",
            "author": "Coldplay",
            "url": "/music/VivaLaVida.mp3",
            "pic": "/music/Coldplay.jpg",
            "lrc": "/music/VivaLaVida-Coldplay.lrc"
        },
        {
          "title": "我记得",
          "author": "赵雷",
          "url": "/music/我记得.mp3",
          "pic": "/music/我记得.jpg",
          "lrc": "/music/我记得-赵雷.lrc"
        },
        {
          "title": "晚婚",
          "author": "江蕙",
          "url": "/music/晚婚.mp3",
          "pic": "/music/晚婚.jpg",
          "lrc": "/music/晚婚-江蕙.lrc"
        }
    ]
}
{% endaplayerlist %}
```