---
title: My Friends
layout: page
---

<style>
.post ul{
    list-style: none;
    padding: 0;
    flex-wrap: wrap;
    display: flex;
    justify-content: space-around;
}
.post ul>li {
    width: 20%;
    position: relative;
    text-align: center;
    margin: 2%;
    min-width: 130px;
}
.post ul>li img {
    width: 60%;
    margin: 20% 20% 5% 20%;
    border-radius: 50%;
    border: 1px solid #bb2222;
}
.post ul>li>a {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    display: inline-block;
    width: 100%;
    font-size: .8em;
}
.post ul>li br {
    display: none;
}
</style>

- ![adlered](media/files/friends/adlered.png)  
  [adlered](https://www.stackoverflow.wiki "贼拉正经的技术博客~")  
- ![csfwff](media/files/friends/csfwff.png)  
  [csfwff](https://www.sszsj.cc/ "鼠鼠在碎觉，请勿打扰~")  
- ![Gakkiyomi](media/files/friends/Gakkiyomi.png)  
  [Gakkiyomi](http://gakkiyomi.me/ "为往圣继绝学~")  
- ![Yui](media/files/friends/Yui.gif)  
  [Yui](https://www.lingmx.com/ "戏演一生美如画")  
- ![Lemon](media/files/friends/Lemon.png)  
  [Lemon](https://chenxiaohui.eu.org/ "我年华虚度，空有一身疲惫")  
- ![iwpz](media/files/friends/iwpz.png)    
  [iwpz](https://iwpz.net/ "和平哥")  
- ![stillwarter](media/files/friends/stillwarter.gif)  
  [stillwarter](https://stillwarter.github.io/ "可爱猫猫")

<script>
    Array.from(document.querySelectorAll('.post ul>li')).map(e => e.onmouseover = (ev) => {
        let target = ev.target
    }, false)
</script>