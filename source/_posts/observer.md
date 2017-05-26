---
title: JavaScript 观察者模式
date: 2017-05-12 09:26:19
tags: method
---

观察者（observer）模式又叫订阅/发布（subscriber/publisher）模式。浏览器的事件监听就是这种模式的实例。订阅者 A 订阅发布者 B 的特定动作，当发布者 B 的特定动作发生后，订阅者 A 便会收到通知，然后订阅者 A 就可以进行相应的处理。

<!-- more -->

下面以该模式实现一个小游戏，游戏规则为：若干名游戏者分别敲击键盘某个按键，看谁在 30 秒内敲击键盘次数最多。游戏过程中的比分会实时显示在记分牌上。

首先，定义一个通用的发布者对象：

```
var publisher = {
    // 订阅者
    subscribers : {
        // 消息类型 ：订阅者组成的数组
        any : []
    },
    // 新增订阅者（消息类型、以及相应的回调函数）
    subscribe : function(type, fn, context){
        var subscribers = this.subscribers;
        type = type || 'any';
        fn = typeof fn === 'function' ? fn : context[fn];
        if (typeof subscribers[type] === 'undefined'){
            subscribers[type] = [];
        }
        subscribers[type].push({
            fn : fn,
            context : context || this
        });
    },
    // 删除订阅者
    unsubscribe : function(type, fn, context){
        this.notice('unsubscribe',type,fn,context);
    },
    // 发布消息（会通知订阅者）
    publish : function(type,arg){
        this.notice('publish',type,arg);
    },
    // 根据消息类型，触发订阅者的回调函数
    notice : function(action,type,arg,context){
        var pubtype = type || 'any',
            subscribers = this.subscribers[pubtype],
            i,
            fn,
            context,
            max = subscribers ? subscribers.length : 0;

        for (i = 0;i < max;i += 1){
            fn = subscribers[i].fn;
            context = subscribers[i].context;
            
            if (action === 'publish'){
                fn.call(context,arg);
            } else {
                fn === arg && 
                context === context &&
                (subscribers.splice(i,1))
            }
        }
    }
};
```

将一个普通对象转化成发布者对象：

```
function makePublisher(o){
    var i;
    for (i in publisher){
        publisher.hasOwnProperty(i) && 
        typeof publisher[i] === "function" &&
        (o[i] = publisher[i])
    }
    o.subscribers = {any : []};
}
```

然后，定义玩家构造函数和实例方法：

```
function Player(name,key){
    this.points = 0;
    this.name = name;
    this.key = key;
    // 发布 newplayer 类型消息
    this.publish('newplayer',this);
}

Player.prototype.play = function(){
    this.points += 1;
    // 发布 play 类型消息
    this.publish('play',this);
}
```

定义记分牌：

```
var scoreboard = {
    element : document.getElementById("scoreboard"),
    update : function(score){
        var i, msg = '';
        for (i in score){
            if (score.hasOwnProperty(i)){
                msg += '<p><b>' + i + '<\/b>: ';
                msg += score[i];
                msg += '<\/p>';
            }
        }
        this.element.innerHTML = msg;
    }
};
```

定义游戏对象：

```
var game = {
    keys : {},
    addPlayer : function(player){
        var key = player.key.toString().charCodeAt(0);
        this.keys[key] = player;
    },
    handleKeypress : function(e){
        e = e || window.event;
        if (game.keys[e.which]){
            game.keys[e.which].play();
        }
    },
    handlePlay : function(player){
        var i,
            players = this.keys,
            name = '',
            points = 0,
            score = {};

        for (i in players){
            if (players.hasOwnProperty(i)){
                name = players[i].name;
                points = players[i].points;
                score[name] = points;
            }
        }
        // 发布 update 类型消息
        this.publish('update',score);
    }
};
```

新建发布者：

```
makePublisher(Player.prototype);
makePublisher(game);
```

事件订阅：

```
// 对所有的 Player 实例，当其发布消息 newplayer 时，
// 会触发 game.addPlayer() 方法
Player.prototype.subscribe("newplayer","addPlayer",game);

// 对所有的 Player 实例，当其发布消息 play 时，
// 会触发 game.handlePlay() 方法
Player.prototype.subscribe("play","handlePlay",game);

// 对 game 对象，当其发布消息 update 时，
// 会触发 scoreboard.update() 方法
game.subscribe("update",scoreboard.update,scoreboard);

// 对 window 对象，当其发送键盘按压事件时，
// 会触发 game.handleKeypress() 方法
window.onkeypress = game.handleKeypress;
```

新建游戏者：

```
var palyername,key;
while (1) {
    playername = prompt("Add player (name)");
    if (!playername){
        break;
    }
    while (1){
        key = prompt("Key for " + playername + "?");
        if (key){
            break;
        }
    }
    new Player(playername,key);
}
```

30 秒后终止游戏：

```
setTimeout(function(){
    window.onkeypress = null;
    alert('Game over!');
}, 30000);
```

程序执行流程如下：
(1) 动态创建多个 player 对象。用户在对话框输入用户名和对应的按键，并确认后，调用 new Player() 方法新建游戏者。重复执行此操作和依次创建多个游戏者。当用户取消输入用户名时，终止创建游戏者，进入游戏。
(2) new Player() 方法执行时，会发布 【**newplayer**】 类型消息。
(3) 消息 【**newplayer**】 会触发 game.addPlayer() 方法。也就是说，当用户依次输入用户名后，game 对象便会在其 keys 属性中保存各个用户名及对应按键信息。
(4) 游戏过程中，用户 player01 按下游戏前设置的按键，就会触发 game.handleKeypress() 方法。
(5) game.handleKeypress() 方法执行时，会触发 player01.play() 方法。
(6) player01.play() 方法会发布 【**play**】 类型消息。
(7) 消息 【**play**】 会触发 game.handlePlay() 方法。
(8) game.handlePlay() 方法会发布 【**update**】 类型消息。
(9) 消息 【**update**】 会触发 scoreboard.update() 方法。
(9) scoreboard.update() 方法更新记分牌。
(10) 30 秒时间到，游戏结束！


参考：
[1] 《JavaScript 模式》