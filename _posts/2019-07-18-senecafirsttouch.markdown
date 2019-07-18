---
layout: post
title: seneca 初探
date: 2019-07-18 12:00:00 +0800
tags: 技术 js
---

## 概况

Seneca是一个nodejs下的老牌的微服务工具集了。

Seneca的最大的特色可能就是足够灵活，但是推广的最大的障碍也是太灵活了。

Seneca是基于模式的一种微服务工具集，使用上真的是挺简单的，看了一天文档就知道是怎么玩的了。

但是使用下来，有那么几个问题。

1. 官方基本已经停止功能上的开发了。新版本开发缓慢。
2. 官方插件很久都没有更新过，虽然自己写插件也是可以的，但是没有官方插件还是没那么方便。
3. 文档和版本不匹配，文档是下一个版本的，npm仓库上的最新版本都没文档新。
4. example实在是太老了，例子还是正式版本以下的一个版本，没有使用新版的example，非常不适合上手。
5. 官方推荐的demo，Seneca的版本都没固化……试版本试了半天……简直了
6. 因为技术框架太小众，资料非常的少。

结论就是，这个玩意虽然很灵活，也有人在用，但是实在是太小众了。从设计模式角度来说，框架本身还行。
但是不适合大型互联网项目，比较适合小项目。虽然，通过插件设计，可以实现各种功能，功能上问题不大。
整个框架使用了类似actor模式的设计思路，感觉就是微服务版的多线程程序的感觉。
程序中所有的耦合关系，基本都是通过“消息”来实现的，万事万物皆消息。
基于nodejs的性能，这种actor模式的设计，不一定会有非常高的效率。但是最大的问题还是掌握这个模式的人，太少。

## 入门

Seneca的入门其实很快，比如写一个Seneca微服务的步骤：

1. 用实例化方法Seneca()创建一个实例。
2. 最Seneca实例进行配置
3. 写一些函数，这些函数是一些功能函数，用来处理逻辑
4. 定义消息匹配模式，然后把消息模式和功能函数绑定
5. 启动脚本，即可通过给服务发送能匹配上的消息，来触发绑定再模式上的功能函数。
6. 如果不符合需求，可以自行编写插件（中间件）去处理一些通用逻辑，编写插件用的设计思想，其实就是AOP。

## 示例代码

一个使用user plugin的最简单的Seneca微服务的例子

```js
require('seneca')()
  .use('user')
  .listen(10101)
  .ready(function(){
    this.act({role:'user',cmd:'register',nick:'u1',name:'U1',password:'u1'})
  })
```

一个简单的添加模式匹配的例子

```js
var seneca = require('seneca').()

seneca
  .add("compare:project",function(msg,done){//增加方法：项目比较
    done(errorHandler(msg),comparePrj())
  })
  .add("compare:version",function(msg,done){//增加方法：版本比较
    done(errorHandler(msg),compareVer(msg))
  })
  .add("compare:measure",function(msg,done){//增加方法：不同模型下的测算比较
    done(errorHandler(msg),compareModleCal(msg))
  })
  .listen(10502)//简单配置，方法接收一个端口号，服务监听9000端口
```

编写一个plugin的例子

```js


function service(options) {
    this.add(`${$module},if:list`, (msg, done) => {
        done(null, db.users)
    })
    this.add(`${$module},if:load`, (msg, done) => {
        const { id } = msg.args.params
        done(null, db.users.find(v => Number(id) === v.id))
    })
    this.add(`${$module},if:edit`, (msg, done) => {
        let { id } = msg.args.params
        id = +id
        const { name } = msg.args.body
        const index = db.users.findIndex(v => v.id === id)
        if (index !== -1) {
            db.users.splice(index, 1, {
                id,
                name
            })
            done(null, db.users)
        } else {
            done(null, { success: false })
        }
    })
    this.add(`${$module},if:create`, (msg, done) => {
        const { name } = msg.args.body
        db.users.push({
            id: ++userCount,
            name
        })
        done(null, db.users)
    })
    this.add(`${$module},if:delete`, (msg, done) => {
        let { id } = msg.args.params
        id = +id
        const index = db.users.findIndex(v => v.id === id)
        if (index !== -1) {
            db.users.splice(index, 1)
            done(null, db.users)
        } else {
            done(null, { success: false })
        }
    })
}

module.exports = {
    restService : service
}
```

可以使用client来配置跨模块的调用,其实就是把特定模式匹配到其他模块的上，然后，其他模块接到消息，再进行进一步的匹配。

```js
var seneca = require('seneca')()
var confit = {}
const Promise = require('bluebird')

const act = Promise.promisify(seneca.act, { context: seneca })

seneca
.add()//增加一些项目方法的模式匹配，这里省略
//……
.client({//把所有带参数 modul：config的消息，都匹配到config server 服务上
    port: '10011',
    pin: 'modul:config'
})

act("modul:config").then(res=>{
    config = res
    console.log("---------------already get config from config center-----------------")
    console.log(config)
    console.log("---------------------------------------------------------------------")
}).catch(err=>{
    errorHandler(err)
})

var errorHandler = (err)=>{
    console.log(err)
    //……处理异常
}

```

## 总结

如果你看完这些还没看懂，那么……请放弃这个框架。这个框架本身没有银弹，也没有语法糖。

没有特别完备的生态，也没有过多的功能。如果你不想使用一个重型框架（比如spring cloud）来书写你的程序，那么Seneca是适合你的。

但是，你打算写个功能完备的微服务，那么请不要使用Seneca。这绝对不是一个好的选择。

用Seneca设计微服务，不太需要很强的技术能力，不过真的需要很好的设计能力。
