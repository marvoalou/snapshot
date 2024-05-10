---
title: Hexo
abbrlink: 37733
date: 2024-03-01 15:41:37
background: url(https://w.wallhaven.cc/full/2y/wallhaven-2yw1qx.jpg)
tags: 知识
---

## Hexo 美化记录

页脚养鱼：
footer.styl
```styl
background-color: alpha($dark-black, .1)

#footer-wrap
   position: absolute
   padding: 1.2rem 1rem 1.4rem
   color: $light-grey
   text-align: center
   left: 0
   right: 0
   top:0
   bottom: 0

   #footer
     if hexo-config('footer_bg') != false
       &:before
       position: absolute
       width: 100%
       height: 100%
       background-color: alpha($dark-black, .1)
       content: ''
```
js

```js
- <script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.min.js"></script>
- <script src="https://cdn.jsdelivr.net/gh/xiabo2/CDN@latest/fishes.js"></script>
```

固定宽度：
```css
/* 鱼塘固定宽度 */
canvas:not(#ribbon-canvas), #web_bg {
    margin-bottom: -0.5rem;
    display: block;
    width: 100%;
    height: 160px
}
```

开启懒加载：
```zsh
npm install hexo-lazyload-image --save
```

