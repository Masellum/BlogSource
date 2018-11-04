---
title: 四零四 
date: 2018-08-03 15:22:56
layout: page
comments: false
permalink: /404.html
---


你好，失落的灵魂，疲惫的旅行者，你似乎来到了我都没有想到过的地方。


让我把你送回去吧。

页面将在<span id="jumpTo"></span>秒内自动跳转回[首页](https://blog.masellum.me/)。
<blockquote>
::before
<p id="hitokoto">一言获取中...</p>
</blockquote>
<!-- 以下写法，选取一种即可 -->

<!-- 现代写法，推荐 -->
<!-- 兼容低版本浏览器 (包括 IE)，可移除 -->
<script src="https://cdn.jsdelivr.net/npm/bluebird@3/js/browser/bluebird.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/whatwg-fetch@2.0.3/fetch.min.js"></script>
<!--End-->
<script>
  fetch('https://v1.hitokoto.cn')
    .then(function (res){
      return res.json();
    })
    .then(function (data) {
      var hitokoto = document.getElementById('hitokoto');
      hitokoto.innerText = data.hitokoto + "\n——「" + data.from + "」"; 
    })
    .catch(function (err) {
      console.error(err);
    })
</script>

<script type="text/javascript">
function countDown(secs, surl) {
    var jumpTo = document.getElementById('jumpTo');
    jumpTo.innerHTML = secs;
    if (--secs > 0) {
        setTimeout("countDown(" + secs + ",'" + surl + "')", 1000);
    } else {
        location.href = surl;
    }
}
</script>
<script type="text/javascript">countDown(10,'/');</script>
