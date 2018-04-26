> ### 前端如何防范XSS

##### XSS ( Cross Site Scripting ) 

跨站脚本攻击, 是一种攻击者向用户的浏览器注入恶意代码脚本的攻击。

XSS攻击的种类：

1. 持续型XSS攻击

攻击者输入恶意代码并通过评论或表单提交等方式将其提交到网站, 而网站的后端却对用户的数据不做任何处理进而直接存入数据库, 当其他用户访问该网站并且需要请求网站的评论数据时网站后端直接从数据库中取出带有恶意代码的数据返回给用户页面此时用户就会遭受到攻击, 比如`while { alert( 'Attack!' ) }` 这样的恶意脚本。

2. 反射型XSS攻击

用户向网站发送get请求某个网页的url, 但用户点开的是带攻击的url类似于:

````javascript
/* http://xxx?name=<script>alert('233')</script>   <-- evil url
   ctx.render('index.html', { name:name });
   --index.html--
   <html>
   <div>${name}</div>  
   </html>
*/
````

3. 基于DOM的XSS攻击

它是反射型攻击的一种, 此时服务器返回的页面是正常的, 只是页面在执行JS时会将恶意脚本一并执行。

````javascript
/*
http://xxx?name=<script>alert('233')</script>     <-- evil url
--index.js-- 
var name = getparm('name');
document.querySelector('div').innerHTML = name;
*/
````

防止XSS注入的一些方法：

1. 对用户输入进行转义

````javascript
function htmlEncode(html) {
    var sub = document.createElement('div');
    sub.textContent != null ? sub.textContent = html : sub.innerText = html;
    var output = sub.innerHTML;
    sub = null;
    return output;
}

//既然有转义当然也有反转义的方法

function htmlDecode(text) {
    var sub = document.createElement('div');
    sub.innerHTML = text;
    var output = sub.textContent || sub.innerText;
    sub = null;
    return output;
}
````

2. 对数据进行编码

````javascript
/*
 @ escape()
 
 @ encodeURI()
 
 @ encodeURIComponent()
 -- 对应的解码方法 --
 @ unescape()
 
 @ unencodeURI()
 
 @ unencodeURIComponent()
*/
````

3. CSP( Content Security Policy )

CSP的实质就是白名单制度, 开发者明确告诉客户端, 哪些外部资源可以加载和执行, 等同于提供白名单。它的实现和执行全部由浏览器完成, 开发者只需提供配置, CSP大大增强了网页的安全性。

有两种方法可以启用CSP, 一种是通过设置HTTP头信息的`Content-Security-Policy` 字段, 另外一种是通过网页中的`<meta>` 标签。

````javascript
<meta http-equiv="Content-Security-Policy" content="script-src 'self'; object-src 'none'; style-src cdn.example.org third-party.org; child-src https:">
````

CSP通过设置限制选项来提供我们所需的白名单：

````javascript
/*
 @ script-src: 外部脚本
 
 @ style-src: 样式表
 
 @ img-src: 图像
 
 @ media-src: 媒体文件 ( 音频和视频 )
 
 @ font-src: 字体文件
 
 @ object-src: 插件 ( 比如Flash )
 
 @ child-src: 框架
 
 @ connect-src: HTTP 连接 ( 通过 XHR、WebSockets、EventSource等 )
 
 ......
*/
````

` default-src` 用来设置上面每个选项的默认值:

````javascript
Content-Security-Policy: default-src 'self'     // 限制所有的外部资源, 只能从当前域名加载
````

其他的一些限制：

````javascript
/*
 @ block-all-mixed-content：HTTPS 网页不得加载 HTTP 资源（浏览器已经默认开启）
 
 @ upgrade-insecure-requests：自动将网页上所有加载外部资源的 HTTP 链接换成 HTTPS 协议
 
 @ sandbox：浏览器行为的限制，比如不能有弹出窗口等
*/
````

使用`report-uri` 还可以记录XSS注入的行为并报告给指定的`url`：

````javascript
Content-Security-Policy: default-src 'self'; ...; report-uri https://retroastro.com;
````

设置多个值, 用空格分隔：

````javascript
Content-Security-Policy: script-src 'self' https://host1.com https://host2.com
````

`script-src` 中的一些特殊值：

````javascript
/*
 @ nonce: 每次HTTP回应给出一个授权token，页面内嵌脚本必须有这个token，才会执行。
 
 @ hash: 列出允许执行的脚本代码的Hash值，页面内嵌脚本的哈希值只有吻合的情况下，才能执行。
*/
````

````javascript
 /* nonce值的例子：
 
    Content-Security-Policy: script-src 'nonce-EDNnf03nceIOfn39fn3e9h3sdfa'

    <script nonce=EDNnf03nceIOfn39fn3e9h3sdfa>
        // some code
    </script>
  
    设置的nonce值要与内嵌脚本中的nonce相等才会执行代码
 */
````

````javascript
/*  hash值的例子：

    Content-Security-Policy: script-src 'sha256-qznLcsROx4GACP2dm0UCKCzCG-HiZ1guq6ZZDob_Tng='
    
    <script> //somecode </script>
    
    服务器给出一个允许执行代码片段的hash值, 对应的代码才会执行, 因为hash值相等。
*/
````

>  ### JavaScript的各种宽高属性

网上看到有一篇博客讲的挺好挺全面的, 有需要的可以去看看：

### [戳这里看JS中的各种宽高图解](http://blog.poetries.top/2016/12/13/js-props/) 

* ##### 自己写的一个小demo

##### 场景：实现bilibili右侧导航栏的切换导航效果。

````html
// HTML
<main>
    <div class="container">
        <div class="part">Live</div>
        <div class="part">Comics</div>
        <div class="part">Drama</div>
        <div class="part">Native</div>
        <div class="part">Music</div>
        <div class="part">Dance</div>
        <div class="part">Game</div>
        <div class="part">Science</div>
        <div class="part">Life</div>
        <div class="part">Kichiku</div>
        <div class="part">Advertise</div>
        <div class="part">Entertain</div>
        <div class="part">Fashion</div>
        <div class="part">TVshows</div>
        <div class="part">Film</div>
        <div class="part">Movies</div>
    </div>
</main>
<aside class="elevator-module">
    <div class="nav-list">
        <div class="item active" data-id=0>直播</div>
        <div class="item" data-id=1>动画</div>
        <div class="item" data-id=2>番剧</div>
        <div class="item" data-id="3">国创</div>
        <div class="item" data-id="4">音乐</div>
        <div class="item" data-id="5">舞蹈</div>
        <div class="item" data-id="6">游戏</div>
        <div class="item" data-id="7">科技</div>
        <div class="item" data-id="8">生活</div>
        <div class="item" data-id="9">鬼畜</div>
        <div class="item" data-id="10">广告</div>
        <div class="item" data-id="11">娱乐</div>
        <div class="item" data-id="12">时尚</div>
        <div class="item" data-id="13">TV剧</div>
        <div class="item" data-id="14">影视</div>
        <div class="item" data-id="15">电影</div>
    </div>
    <div class="s-line"></div>
    <div class="back-top">
        <div class="circle"></div>
    </div>
</aside>

````

````css
// CSS
.elevator-module{
    position:fixed;
    top:20px;
    left:50%;
    margin-left:590px;
    transition:all .4s ease;
    z-index:10;
}
.elevator-module .nav-list{
    position:relative;
    background-color:#f6f9fa;
    border:1px solid #e5e9ef;
    overflow:hidden;
    border-radius:4px;
}
.elevator-module .nav-list .item{
    width:48px;
    height:32px;
    line-height:32px;
    text-align:center;
    transition:all .3s;
    cursor:pointer;
}
.elevator-module .nav-list .item.on, .elevator-module .nav-list .item:hover{
    background-color:#00a1d6;
    color:#fff;
}
.nav-list .item.active{
    color:#fff;
    background:#00a1d6;
}
.elevator-module .s-line{
    position:relative;
    border-left:1px solid #ddd;
    border-right:1px solid #ddd;
    height:9px;
    width:30px;
    margin:0 auto;
}
.elevator-module .back-top:hover{
    background-color:#00a1d6;
    border-color:#00a1d6;
}
.elevator-module .back-top{
    position:relative;
    display:block;
    cursor:pointer;
    height:50px;
    background-color:#f6f9fa;
    overflow:hidden;
    border: 1px solid #e5e9ef;
    border-radius:4px;
    text-align: center;
}
.circle{
    display:inline-block;
    width:30px;
    height:30px;
    margin-top:10px;
    border-radius:50%;
    background:#ffafc9;
}
.part{
    width:800px;
    height:600px;
    margin:20px auto;
    background:#00a1d6;
    text-align:center;
    line-height:600px;
    font-size:13em;
    font-family:Comic Sans MS;
    color:#fff;
    font-weight:700;
    border-radius:4px;
}
.container{
    width:1160px;
    margin:0 auto;
    position:relative;
}
````

````javascript
// JavaScript
var ElevatorModule = {
    init() {
        this.scrollEvent();
    },
    scrollEvent() {

        var elevator = document.querySelector('.elevator-module');
        var navList = document.querySelector('.nav-list');
        var items = document.querySelectorAll('.nav-list .item');
        var modules = document.querySelectorAll('.part');
        var backtop = document.querySelector('.back-top');

        window.addEventListener('scroll', slipper);
        navList.addEventListener('click', moveTo);
        backtop.addEventListener('click', fly);


        function slipper() {

            for ( var item of items ) {

                var rect = item.getBoundingClientRect();

                var scrollTop = document.documentElement.scrollTop;

                var item_top = rect.top + scrollTop; //获取右侧导航每个分区上方到页面最顶部的距离

                var fetch = getLocation();

                var id = item.getAttribute('data-id');

                if ( item_top >= fetch[id] - 100 ) {  //移到哪个分区右侧的hover就停留在哪

                    items.forEach((el) => {
                        el.classList.remove('active'); 
                    })

                    item.classList.add('active');

                }

            }

        }

        var flag = true;

        //定位到指定分区
        function moveTo() {

            var e = window.event;

            items.forEach((item) => {
                item.classList.remove('active');
            })

            if ( e.target && e.target.nodeName == 'DIV' ) {

                var id = e.target.getAttribute('data-id');

                e.target.classList.add('active');

                var fetch = getLocation();

                if ( flag ) {

                    AnimationFrame(fetch[id]);

                    id ? flag = false : flag = true; 
                    //设置节流效果, 即在运动的过程中不允许再次触发AnimationFrame函数

                }

            }

        }
        //scrollTop过渡
        function AnimationFrame(future_top) {

            let current_top = document.documentElement.scrollTop;

            var timer = setInterval( () => {

                if ( current_top > future_top ) {

                    current_top -= 60;
                    document.documentElement.scrollTop = current_top;

                } else if ( current_top < future_top ) {

                    current_top += 60;
                    document.documentElement.scrollTop = current_top;

                }
                
                if ( current_top >= future_top - 30 && current_top <= future_top + 30 ) { 
                     //当移动到指定的范围内时直接定位到对应的位置
                  
                    document.documentElement.scrollTop = future_top;
                    clearInterval(timer);
                    flag = true;

                }
            },10)
        }
        //获取pageY
        function getLocation() {

            var arr = [];

            modules.forEach((module) => {
                var thing = pageY(module);
                arr.push(thing);
            })

            return arr;

        }
        //获取页面中间每个分块上方到页面最顶部的距离
        function pageY(el) {

            if ( el.offsetParent ) {

                return el.offsetTop + pageY(el.offsetParent);

            } else {

                return el.offsetTop;

            }

        }
        //平滑返回页面顶部
        function fly() {

            cancelAnimationFrame(timer);

            var timer = requestAnimationFrame( function fn() {

                var top = window.pageYOffset;

                if ( top > 0 ) {

                    document.documentElement.scrollTop = top - 80;

                    timer = requestAnimationFrame(fn);

                } else {
                    cancelAnimationFrame(timer);
                }
            })

        }

    }
}
ElevatorModule.init();
````

> ### PS

一些需要注意的常见js事件属性：

* #####event.stopPropagation()


终止事件在传播过程的捕获、目标处理或起泡阶段进一步传播。调用该方法后，该节点上处理该事件的处理程序将被调用，事件不再被分派到其他节点。

* ##### event.preventDefault()

该方法将通知 Web 浏览器不要执行与事件关联的默认动作（如果存在这样的动作）。

* ##### 不论鼠标指针穿过被选元素或其子元素，都会触发 mouseover 事件。对应mouseout



* ##### 只有在鼠标指针穿过被选元素时，才会触发 mouseenter 事件。对应mouseleave




###### 参考文章：

https://www.cnblogs.com/caizhenbo/p/6836390.html  　XSS

https://www.cnblogs.com/zzgblog/p/5819807.html  　转义方法

http://www.ruanyifeng.com/blog/2016/09/csp.html 　　CSP



