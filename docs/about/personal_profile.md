---
title: 个人主页
---

<center><font  color= #518FC1 size=6 class="ml3">AstonWang‘s Blog</font></center>
<script src="https://cdnjs.cloudflare.com/ajax/libs/animejs/2.0.2/anime.min.js"></script>

 
<center><font  color= #518FC1 size=2 class="ml3">一切看似逝去的，都不曾离开，你所给予的爱与温暖，让我执着地守护在这里</font></center>
<script src="https://cdnjs.cloudflare.com/ajax/libs/animejs/2.0.2/anime.min.js"></script>




<div id="rcorners2" >
  <div id="rcorners1">
    <i class="fa fa-calendar" style="font-size:100"></i>
    <body>
      <font color="#4351AF">
      编程是一种技艺，一种需要用心学习的技艺
        <p class="p1"></p>
<script defer>
    //格式：2020年04月12日 10:20:00 星期二
    function format(newDate) {
        var day = newDate.getDay();
        var y = newDate.getFullYear();
        var m =
            newDate.getMonth() + 1 < 10
                ? "0" + (newDate.getMonth() + 1)
                : newDate.getMonth() + 1;
        var d =
            newDate.getDate() < 10 ? "0" + newDate.getDate() : newDate.getDate();
        var h =
            newDate.getHours() < 10 ? "0" + newDate.getHours() : newDate.getHours();
        var min =
            newDate.getMinutes() < 10
                ? "0" + newDate.getMinutes()
                : newDate.getMinutes();
        var s =
            newDate.getSeconds() < 10
                ? "0" + newDate.getSeconds()
                : newDate.getSeconds();
        var dict = {
            1: "一",
            2: "二",
            3: "三",
            4: "四",
            5: "五",
            6: "六",
            0: "天",
        };
        //var week=["日","一","二","三","四","五","六"]
        return (
            y +
            "年" +
            m +
            "月" +
            d +
            "日" +
            " " +
            h +
            ":" +
            min +
            ":" +
            s +
            " 星期" +
            dict[day]
        );
    }
    var timerId = setInterval(function () {
        var newDate = new Date();
        var p1 = document.querySelector(".p1");
        if (p1) {
            p1.textContent = format(newDate);
        }
    }, 1000);
</script>
      </font>
    </body>
    <!-- <b><span id="time"></span></b> -->
  </div>
</div> 

<!-- <img class="img1" src="https://s2.loli.net/2024/02/01/AgiGpYk38C6ctJV.jpg"> -->


<!-- <div id="rcorners3" >
  <img class="img1" src="https://s1.imagehub.cc/images/2024/02/02/79cb7379982d1c7bb0ae7163985609c4.jpeg"  width="170" height="200" alt="个人头像" align="left" style="margin-right: 20px; "/>
  <div>
    <p style="font-size: 40px">Wcowin</p>
    <p style="font-size: 18px">A college student in CQ</p>
  </div>
</div>
    -->


!!! pied-piper1 "About Site"
    本站名为：空谷幽兰，取自许巍歌曲《空谷幽兰》，始于20190417，重建于20231201，此站点为个人知识库，经历了几次更迭，并决心从重建之日开始，坚持长期主义，认真记录，思考总结。不为别的，只为在这漫长而又短暂的一生中，留一丝浅浅的记忆...（如果错误，欢迎指正！）

!!! pied-piper1 "About Me <img src="../pics/personal.jpg" alt="arv-anshul" style="width: 30px; border-radius: 50%;" />"
    - [x] Hey, I'm AstonWang! 一个咖啡重度爱好者...
    - [x] 目前主要从事后端+机器学习工程化方向的工作...


!!! pied-piper1 "About 一言"    
    - [x] 我认清时间的时候，只剩下一寸光阴，这是我的个人主页，到处是岁月痕迹...
    - [x] 很多时候技术人觉得实现功能重要，事实上可持续维护才是最重要的... 



## 現在の技能

!!! pied-piper1 "About Skill"
    - [x] 持续精进服务端开发技能，致力于发展成为一名全沾的艺人...  
    - [x] 持续关注机器学习+云原生领域的技术...  

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

<!-- <img class="img1" src="https://s2.loli.net/2024/02/01/AgiGpYk38C6ctJV.jpg"> -->

<!-- ## 联系我

<a href="https://muselink.cc/Wcowin" target="_blank">
  <img class="img1" src="https://s1.imagehub.cc/images/2024/02/02/3d5a68d9ca0da9137d927bda1a0b41e7.jpeg"  >
  <center>
    <div style="color:orange; 
    color: #999;
    padding: 2px;">我的名片</div>
  </center>  
</a>

<figure markdown >
  ![Image title](https://s1.imagehub.cc/images/2024/02/02/43c746351261969a02bda7d743199604.jpeg){.img1}
  <figcaption>公众号</figcaption>
</figure> -->

<!-- ## 个人简历
[个人简历(在线)](https://cv.devtool.tech/preview/538d1d22-c3a3-4611-9ffb-be6d8fbf0e8c) -->

<!-- ## 个人技能
本人擅长 Ai、Fw、Fl、Br、Ae、Pr、Id、Ps 等软件的安装与卸载。  
精通 CSS、JavaScript、PHP、ASP、C、C++、C#、Java、Ruby、Perl、Lisp、Python、Objective-C、ActionScript、Pascal 等单词的拼写。  
熟悉 Windows、Linux、OS X、Android、iOS、WP8 、harmony、hyper等系统的开关机。 -->

<!-- ## 个人荣誉
![IMG_9007.jpeg](https://s2.loli.net/2024/02/03/RH5jOlZqdITAcw8.jpg){loading=lazy  class="img1"  }   -->


<!-- <head>

<script>
function _howxm(){_howxmQueue.push(arguments)}
window._howxmQueue=window._howxmQueue||[];
_howxm('setAppID','14429fca-cac1-4551-a472-b046a96ebb75');
(function(){var scriptId='howxm_script';
if(!document.getElementById(scriptId)){
var e=document.createElement('script'),
t=document.getElementsByTagName('script')[0];
e.setAttribute('id',scriptId);
e.type='text/javascript';e.async=!0;
e.src='https://static.howxm.com/sdk.js';
t.parentNode.insertBefore(e,t)}})();
</script>

</head> -->

<!-- ## 须知
如果你在浏览博客的过程中发现了任何问题，欢迎前往 GitHub 的[代码仓库](https://github.com/Wcowin/Wcowin.github.io)提交 Issues 或直接修改相关文件后提交 Pull Requests。如果你有其他事情想要咨询，可以通过下方按钮使用邮件联系我,请不要滥用博客的评论功能发表与主题无关言论。 -->

[Send Email :fontawesome-solid-paper-plane:](mailto:sswss5@aliyun.com>){.md-button}
<!-- <a target="_blank"  href="mailto:wangkewen821@gmail.com""><button class="buttonxuan2" style="vertical-align:middle" ><span>Send Email:fontawesome-solid-paper-plane: </span></button></a> -->
<!-- 
<chat-bot platform_id="d19a99ed-b684-4d64-8c70-7663d974af17" user_id="325b3ae2-0317-4c5f-9f9b-c4ce0e51e36b" chatbot_id="8eedef48-41ef-4f78-97d9-71e8197a452d"><a href="https://www.chatsimple.ai/?utm_source=widget&utm_medium=referral">[chatbot]</a></chat-bot><script src="https://cdn.chatsimple.ai/chat-bot-loader.js" defer></script> -->
