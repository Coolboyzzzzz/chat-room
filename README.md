# 网络聊天室

## 1.利用H5的WebScoket和后端node的ws包实现一个简易的聊天室

现在的网站上进行实时通讯的功能需求日益增长，今天就用H5的WebScoket和node的ws实现一个网络实时的聊天室

## 2.程序的截图

![image-20220319150435279](https://github.com/Coolboyzzzzz/FrontEndInterView/blob/main/image/image-20220319150435279.png?raw=true)

![image-20220319150454216](https://github.com/Coolboyzzzzz/FrontEndInterView/blob/main/image/image-20220319150454216.png?raw=true)

做出了一个简易的网上实时的聊天

## 3.代码

首先我们需要一个服务端作为我们的信息中转站

下面是seriver端的代码

```js
const ws = require('ws') 
const experss = require('express')
const app = experss()
app.use('/', experss.static('./public'))
var Wbs = new ws.Server({ host: '127.0.0.1', port: '8080' }, () => {
    console.log('成功')
}) //创建服务器
var clients = [] //连接的客户端
var ulits = [] //用于保存用户的名字
Wbs.on('connection', (client) => { //服务器监听客户端连接服务器的事件
    console.log('连接成功')
    clients.push(client) //把这个客户端保存在一个数组中
    client.on('message', (msg) => { //监听这个客户端发送的消息
        var msg = JSON.parse(msg)
        if (msg.text) { //如果发送msg对象有text属性那么给列表上所有的客户端进行广播
            for (var i = 0; i < clients.length; i++) {
                clients[i].send(JSON.stringify(msg))
            }
        } else { //如果msg对象没有text属性就代表又新人加入聊天室，把新人的名字进行广播
            ulits.push(msg)
            for (var i = 0; i < clients.length; i++) {
                clients[i].send(JSON.stringify(ulits))
            }
        }
    })
    client.on('close', () => { //监听客户端断开连接
        for (var i = 0; i < clients.length; i++) {
            if (client == clients[i]) { //确定是客户端断开的连接在存入数组中的位置，进行删除
                clients.splice(i, 1)
                ulits.splice(i, 1)//以及删除他的信息
                break//若找到直接跳出循环，有利于程序优化
            }
        }
        for (var j = 0; j < clients.length; j++) {
            clients[j].send(JSON.stringify(ulits))//最后把最新的用户列表广播所有客户端
        }
    })
})
app.listen('8090', () => {
    console.log('app启动成功')
})
```



搞定了server端我们还需要页面

下面是js的主要的核心逻辑代码，还有部分css以及调用的包在这里就不展示了，都放到了源码文件夹下



```js
<body>

    <div class="wrap">
        <!-- 头部 Header 区域 -->
        <div class="header">
            <h3>
                <div id='name'></div>
            </h3>
            <div id='rs'></div>
            <img src="img/person01.png" alt="icon" />
        </div>
        <!-- 中间 聊天内容区域 -->
        <div class="main">
            <ul id='talkList' class="talk_list" style="top: 0px;">
                <li class="left_word">
                </li>
                <li class="right_word">
                </li>
            </ul>
            <div class="drag_bar" style="display: none;">
                <div class="drager ui-draggable ui-draggable-handle" style="display: none; height: 412.628px;"></div>
            </div>
        </div>
        <!-- 底部 消息编辑区域 -->
        <div class="footer">
            <img src="img/person02.png" alt="icon" />
            <input id='content' type="text" placeholder="说的什么吧..." class="input_txt" />
            <input id='send' type="button" value="发 送" class="input_sub" />
        </div>
    </div>
</body>
<script type="text/javascript" src="js/scroll.js"></script>
<script>
    var usersname = window.sessionStorage.getItem('usersname')//利用H5的sessionStorage临时存储用户名称每一次都会先读本地时候有缓存
    while (!usersname || !usersname.trim()) {
        usersname = prompt('请输入用户名') //检测用户名是否输入的合法
    }
    window.sessionStorage.setItem('usersname', usersname)  //利用setItem存储缓存

    var name = document.getElementById('name')
    name.innerHTML = `${usersname}` //显示用户名
    var wss = new WebSocket('ws://qiu839169472.e2.luyouxia.net:27647') //建立连接  这里是持续的TCP连接
    wss.onopen = function () { //刚建立连接触发的事件
        wss.send(JSON.stringify({
            'uname': usersname //首先提交用户名
        }))
        wss.onmessage = function (msg) { //这是返回消息的时候触发的事件
            msg = JSON.parse(msg.data) //获取真实的消息

            if (msg.text) { //这里实现的逻辑是 msg是一个对象如果服务器返回的msg对象没有text属性说明是聊天室知识新增了人并没有人发消息，如果含有text属性说明是有人发送了消息
                var li = document.createElement('li') //创建一个DOM元素
                msg.uname += ''
                li.innerHTML = "<div class='tx tx1'>" + msg.uname[0].toUpperCase() + "</div> <span>" + msg.text + "</span>" //给DOM元素添加要显示的文本
                msg.uname == usersname ? li.className = 'right_word' : li.className = 'left_word' //这里判断发送的消息是自己发的还是别人发的，分别添加不同的类名
                talkList.appendChild(li) //把li元素追加到聊天框中
            } else { //msg没有text属性说明知识有人加入了聊天室，则更新列表
                rs.innerHTML = ''
                for (var i of msg) {
                    var div = document.createElement('div') //创建一个元素
                    i = i.uname + ''
                    div.innerHTML = i.slice(0, 1).toUpperCase()//渲染用户头像
                    div.className = 'tx'//给用户头像添加样式
                    rs.appendChild(div)//追加到列表
                }
            }
        }
        wss.onclose = function () {
            console.log('客户端关闭')
        }
    }
    send.onclick = submit
    document.onkeydown = function (event) { //给聊天框添加enter事件，自动发送
        if (event.keyCode == 13) return submit();
    }

    function submit() {  //这是发送消息的处理函数
        var text = document.getElementById('content')
        wss.send(JSON.stringify({
            'uname': usersname,
            'text': text.value

        }))
        text.value = '' //发送完消息清空输入框
        // 初始化右侧滚动条
        // 这个方法定义在scroll.js中
        resetui()//每发送完消息就滚动到最后一个位置
    }
</script>
```

