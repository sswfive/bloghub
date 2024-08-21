---
title: 个人主页
---
<center><font  color= #518FC1 size=6 class="ml3">AstonWang‘s Blog</font></center>
<script src="https://cdnjs.cloudflare.com/ajax/libs/animejs/2.0.2/anime.min.js"></script>

 
<center><font  color= #518FC1 size=3 class="ml4">一切看似逝去的，都不曾离开，你所给予的爱与温暖，让我执着地守护在这里</font></center>
<script src="https://cdnjs.cloudflare.com/ajax/libs/animejs/2.0.2/anime.min.js"></script>


!!! pied-piper1 "About Site"
    本站名为：空谷幽兰，取自许巍歌曲《空谷幽兰》，始于20190417，重建于20231201，此站点为个人知识库，经历了几次更迭，并决心从重建之日开始，坚持长期主义，认真记录，思考总结。不为别的，只为在这漫长而又短暂的一生中，留一丝浅浅的记忆...（如果错误，欢迎指正！）

!!! pied-piper1 "About Me <img src="../pics/personal.jpg" alt="arv-anshul" style="width: 30px; border-radius: 50%;" />"
    - [x] Hey, I'm AstonWang! 一个咖啡重度爱好者...
    - [x] 目前主要从事后端+机器学习工程化方向的工作...


!!! pied-piper1 "About 一言"    
    - [x] 我认清时间的时候，只剩下一寸光阴，这是我的个人主页，到处是岁月痕迹...
    - [x] 很多时候技术人觉得实现功能重要，事实上可持续维护才是最重要的... 

<style>
.skill {
  margin-bottom: 10px;
}

.skill-name {
  font-weight: bold;
  margin-bottom: 5px;
}

.skill-bar {
  background-color: #ddd;
  height: 20px;
  border-radius: 5px;
}

.skill-level {
  background-color: #4CAF50;
  height: 100%;
  border-radius: 5px;
}
</style>

<div class="skill">
  <div class="skill-name">Python</div>
  <div class="skill-bar">
    <div class="skill-level" style="width: 75%;"></div>
  </div>
</div>

<div class="skill">
  <div class="skill-name">Go</div>
  <div class="skill-bar">
    <div class="skill-level" style="width: 50%;"></div>
  </div>
</div>

<div class="skill">
  <div class="skill-name">数据分析</div>
  <div class="skill-bar">
    <div class="skill-level" style="width: 50%;"></div>
  </div>
</div>

<div class="skill">
  <div class="skill-name">机器学习</div>
  <div class="skill-bar">
    <div class="skill-level" style="width: 35%;"></div>
  </div>
</div>

<div class="skill">
  <div class="skill-name">Shell</div>
  <div class="skill-bar">
    <div class="skill-level" style="width: 25%;"></div>
  </div>
</div>


<!-- 可选一言 -->
<center>
<font  color= #608DBD size=5>
<p id="hitokoto">
  <a href="#" id="hitokoto_text" target="_blank"></a>
</p>
<script>
  fetch('https://v1.hitokoto.cn')
    .then(response => response.json())
    .then(data => {
      const hitokoto = document.querySelector('#hitokoto_text')
      hitokoto.href = `https://hitokoto.cn/?uuid=${data.uuid}`
      hitokoto.innerText = data.hitokoto
    })
    .catch(console.error)
</script>
</font>
</center>

[Send Email :fontawesome-solid-paper-plane:](mailto:sswss5@aliyun.com>){.md-button}