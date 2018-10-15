# message
主题：介绍MessageChannel API及其用法

看某个JS库源码的时候发现了MessageChannel

MessageChannel接口是信道通信API的一个接口，它允许我们创建一个新的信道并通过信道的两个MessagePort属性来传递数据

简单来说，MessageChannel创建了一个通信的管道，这个管道有两个口子，每个口子都可以通过postMessage发送数据，而一个口子只要绑定了onmessage回调方法，就可以接收从另一个口子传过来的数据。

一个简单的例子：

   var ch = new MessageChannel();
   var p1 = ch.port1;
   var p2 = ch.port2;
   p1.onmessage = function(e){console.log("port1 receive " + e.data)}
   p2.onmessage = function(e){console.log("port2 receive " + e.data)}
   p1.postMessage("你好世界");
   p2.postMessage("世界你好")；
MessageChannel用法很简单，但是功能却不可小觑。例如当我们使用多个web worker并想要在两个web worker之间实现通信的时候，MessageChannel就可以派上用场：

main.html

  <script>
        var w1 = new Worker("worker1.js");
        var w2 = new Worker("worker2.js");
        var ch = new MessageChannel();
        w1.postMessage("initial port",[ch.port1]);
        w2.postMessage("initial port",[ch.port2]);
        w2.onmessage = function(e){
            console.log(e.data);
        }
  <script>
worker1.js

  var port;
   onmessage = function(e){
    if(e.data == "initial port"){
        port = e.ports[0];
    }else{
        setTimeout(function(){
            port.postMessage("this is from worker1")
        },2000)
      }
  }
worker2.js

    var port;
    onmessage = function(e){
    if(e.data == "initial port"){
        port = e.ports[0];
        port.onmessage = function(e){
            postMessage(e.data)
        }
       }
   }
在上面这个例子中，首先通过web worker的postMessage方法把两个MessageChannel的port传递给两个web woker，然后就可以通过每个port的postMessage方法传递数据了。由于在worker中无法通过console.log打出传递的数据，因此我们通过给w2绑定onmessage回调函数来验证传递是否成功。最终我们可以看到控制台中输出

this is from worker1

而传递的路径为：

w1=> ch1 => ch2 => w2

根据canisue的数据显示，目前大多数浏览器都对MessageChannel有很好的支持，所以不妨在下一个项目里面使用一下这个API吧！
