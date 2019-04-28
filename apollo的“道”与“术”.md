<html lang="en"><head>
    <meta charset="UTF-8">
    <title></title>
<style id="system" type="text/css">h1,h2,h3,h4,h5,h6,p,blockquote {    margin: 0;    padding: 0;}body {    font-family: "Helvetica Neue", Helvetica, "Hiragino Sans GB", Arial, sans-serif;    font-size: 13px;    line-height: 18px;    color: #737373;    margin: 10px 13px 10px 13px;}a {    color: #0069d6;}a:hover {    color: #0050a3;    text-decoration: none;}a img {    border: none;}p {    margin-bottom: 9px;}h1,h2,h3,h4,h5,h6 {    color: #404040;    line-height: 36px;}h1 {    margin-bottom: 18px;    font-size: 30px;}h2 {    font-size: 24px;}h3 {    font-size: 18px;}h4 {    font-size: 16px;}h5 {    font-size: 14px;}h6 {    font-size: 13px;}hr {    margin: 0 0 19px;    border: 0;    border-bottom: 1px solid #ccc;}blockquote {    padding: 13px 13px 21px 15px;    margin-bottom: 18px;    font-family:georgia,serif;    font-style: italic;}blockquote:before {    content:"C";    font-size:40px;    margin-left:-10px;    font-family:georgia,serif;    color:#eee;}blockquote p {    font-size: 14px;    font-weight: 300;    line-height: 18px;    margin-bottom: 0;    font-style: italic;}code, pre {    font-family: Monaco, Andale Mono, Courier New, monospace;}code {    background-color: #fee9cc;    color: rgba(0, 0, 0, 0.75);    padding: 1px 3px;    font-size: 12px;    -webkit-border-radius: 3px;    -moz-border-radius: 3px;    border-radius: 3px;}pre {    display: block;    padding: 14px;    margin: 0 0 18px;    line-height: 16px;    font-size: 11px;    border: 1px solid #d9d9d9;    white-space: pre-wrap;    word-wrap: break-word;}pre code {    background-color: #fff;    color:#737373;    font-size: 11px;    padding: 0;}@media screen and (min-width: 768px) {    body {        width: 748px;        margin:10px auto;    }}</style><style id="custom" type="text/css"></style></head>
<body marginheight="0"><p>Google了下配置中心，结果大概有两类：<br>1. 携程Apollo的介绍。<br>2. 各类配置中心的对比。   

</p>
<p>很实用！从技术选型，到具体实践，一站式服务！可惜都是在讲“术”，“道”在哪里？<br>然后文章我还是撸了一把，术与道 都要有嘛~ 发现下面一个好TM长的博客，来自于国内java中间件大户 阿里。"道"在这里。<br><a href="http://jm.taobao.org/2016/09/28/an-article-about-config-center/">http://jm.taobao.org/2016/09/28/an-article-about-config-center/</a><br>博客是2016年写的，3年过去了，配置中心已经有很多优秀的开源作品，但这篇<br>文章从一个很优雅的名词开始“系统运行时飞行姿态的调整”。老实讲，我是因为这个词才继续看下去的，多么优雅的描述啊，“飞行姿态的调整”，让我联想到飞机的若干飞行场景:<br>   "747穿越大西洋，突然遇到风暴，要调整航线，绕过风暴中心"<br>   "现在是高峰期，马尼拉机场没有足够的跑道，所以你坐的飞机要在马尼拉湾上空盘旋飞行40分钟，排队等待降落"

</p>
<p>我们系统不也有类似的场景么？

</p>
<pre><code>“秒杀活动的流量比我们预估的要高很多，系统要奔溃了，赶紧打开限流开关！让流量在外排队”  ---调整飞行姿态
“什么？坑爹的服务商临时通知明天变更域名？”  ---调整飞行姿态
“今天网络拥堵，需要修改接口的超时时间！”    ---调整飞行姿态
“我靠，生产的配置写错了，立马改掉！”    ---调整飞行姿态</code></pre>
<p>系统要优雅地“调整飞行姿态”，靠什么？ <strong>动态配置</strong>！
配置我懂，啥是<strong>动态</strong>的配置呢？那静态的配置是啥样？一句话解释：   

</p>
<pre><code>系统在“飞行”过程中，通过改变配置来调整“飞行姿态”，这种配置叫动态配置！   </code></pre>
<p> 这样就很容易理解什么是静态：飞行中不需要改变的配置。

</p>
<h5>国内早期知名的配置中心产品Diamond为什么出现在阿里？</h5>
<pre><code>        因为阿里是最早对配置中心产生刚性需求的大厂。</code></pre>
<h5>阿里是太阳系最早的么阿里是太阳系最早的么？</h5>
<pre><code>嘿嘿，不是！ 可以考证到的结果是：1985年就有人提出了动态配置中心的概念了。Jeff Kramer和Jeff Magee两位老哥在IEEE TRANSACTIONS ON SOFTWARE ENGINEERING 上发表了一篇名为 Dynamic Configuration for Distributed Systems的Paper。领先我国好多年啊！！</code></pre>
<h5>阿里对配置中心的刚性需求是啥？</h5>
<pre><code>    1.稀疏的变更  --线上的配置偶尔需要改变一下！
    2.快速的传播  --我改变了配置，系统的飞行姿势要立马改变啊！
    3.多环境的配置隔离 --生产的配置归生产，测试的配置归测试，不要搞错啊！</code></pre>
<p>好了！那篇好TM长的博客撸完了！
下面撸撸这些讲“术”的文章吧。

</p>
<p>“感谢各大厂开源，我们小厂不用花钱买服务了，更不用自研了！”<br>“什么？google一下，发现这么多免费的产品，我们选哪个啊？”<br>来来来，我们先撸一下需求吧! 我们需要配置中心哪些特性，尤其是刚性需求？<br>或者说配置中心有哪些特性是我们需要的？   

</p>
<p>1.静态配置管理。<br>2.动态配置管理。<br>3.多环境配置的管理与隔离。<br>4.配置的快速传播。<br>以上4点刚需。在现在复杂的分布式系统环境下，需求更多了。   

</p>
<pre><code>“要是配置中心挂了，我们取不到配置了怎么办？”   
    --需求：应用可以读本地缓存的配置，配置中心要发出报警信息，告诉运维我挂了，赶紧来维护。
“我系统代码要回滚，配置也要回滚！” --需求：回滚。
"我们现在要灰度发布，配置也要支持！"   --需求：灰度。
"我们这边不光有java的系统 ，还有.net系统"   --多语言支持
"配置要有权限管理，生产配置不是谁都能看的"  --权限管理
"配置中心要高可用"  --冗余部署, 多数据中心
"要高性能，我们有上千个系统，都接入的话，你能抗得住？"  --高性能
"对我系统侵入性小，而且接入要尽可能简单！" --简单，侵入小
"最好还有个体验良好的用户操作界面！"   --admin系统的需求
"我们公司对配置的安全性要求更，需要有审计"  --审计需求</code></pre>
<p>好了，民意调查结束，对大家需求我们了解的差不多了，来对比下各产品吧。
从贴图上看，Apollo与我们需求最贴近！就选Apollo吧！
<a href="https://github.com/flysnow911/DailySpringFrameWork/blob/master/doc/1514121668514028415%20(1"><img src="https://github.com/flysnow911/DailySpringFrameWork/blob/master/doc/1514121668514028415%20(1" alt="横向对比">.png "横向对比")</a>.png "横向对比")
     "我还需要接入Ldap！ 携程: 请下载源码，自行开发！或者等待！








</p>
<p>Edit By <a href="http://mahua.jser.me">MaHua</a></p>
</body></html>