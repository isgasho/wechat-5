## wechat 开发框架

[![GoDoc](http://godoc.org/github.com/liujianping/wechat?status.png)](http://godoc.org/github.com/liujianping/wechat)

该框架仅为抛砖之作, 没有实现的微信公众号的全部api接口, 仅实现部分接口包括:

-	[获取接口调用凭据](http://mp.weixin.qq.com/wiki/2/88b2bf1265a707c031e51f26ca5e6512.html)
-	[接收消息](http://mp.weixin.qq.com/wiki/17/fc9a27730e07b9126144d9c96eaf51f9.html)
-	[发送消息](http://mp.weixin.qq.com/wiki/18/c66a9f0b5aa952346e46dc39de20f672.html)
-	[自定义菜单](http://mp.weixin.qq.com/wiki/6/95cade7d98b6c1e1040cde5d9a2f9c26.html)
-	[用户管理](http://mp.weixin.qq.com/wiki/17/c807ee0f10ce36226637cebf428a0f6d.html)

### 项目依赖

-	[api](http://github.com/liujianping/api)
-	[iris](http://github.com/kataras/iris) 

###  快速开始

客户端快速开发指南:

````go
	
	import "github.com/liujianping/wechat"
	import "github.com/liujianping/wechat/entry"

	api := wechat.NewClient("appid", "appsecret")

	// 获取令牌
	var token entry.Token
	if err := api.Access(&token); err != nil {

	}

	// 获取用户信息
	var user_info entry.UserInfo
	if err := api.GetUserInfo("open_id", "zh_CN", &user_info); err != nil {

	}

	// 更多接口(参考其它微信项目的对象定义即可轻松实现)
	...

````

服务端(支持多应用)快速开发指南:

````go

	package main

	import (
		"log"

		"github.com/liujianping/wechat"
		"github.com/liujianping/wechat/entry"
	)

	func DemoHandle(app *wechat.Application, request *entry.Request) (interface{}, error) {
		log.Printf("demo msg (%v)\n", request)
		return nil, nil
	}

	func EchoHandle(app *wechat.Application, request *entry.Request) (interface{}, error) {
		switch strings.ToLower(request.MsgType) {
		case entry.MsgTypeEvent:
			switch strings.ToLower(request.Event) {
			case entry.EventSubscribe:
				log.Printf("user (%s) subscribed", request.FromUserName)

				var user_info entry.UserInfo
				if err := app.Api().GetUserInfo(request.FromUserName, entry.LangZhCN, &user_info); err != nil {
					return nil, err
				}

				text := entry.NewText(request.FromUserName,
					request.ToUserName,
					time.Now().Unix(),
					fmt.Sprintf("亲爱的(%s), 谢谢您的关注!", user_info.NickName))
				return text, nil

			case entry.EventUnSubscribe:
				log.Printf("user (%s) unsubscribed", request.FromUserName)
			case entry.EventScan:
				log.Printf("user (%s) scan", request.FromUserName)
			case entry.EventLocation:
				log.Printf("user (%s) location", request.FromUserName)
			case entry.EventClick:
				log.Printf("user (%s) menu click (%s)", request.FromUserName, request.EventKey)
			case entry.EventView:
				log.Printf("user (%s) menu view (%s)", request.FromUserName, request.EventKey)
			}
		case entry.MsgTypeText:
			//! echo
			text := entry.NewText(request.FromUserName, request.ToUserName, time.Now().Unix(), request.TextContent)
			return text, nil
		case entry.MsgTypeImage:
		case entry.MsgTypeVoice:
		case entry.MsgTypeVideo:
		case entry.MsgTypeMusic:
		case entry.MsgTypeNews:
		}
		return nil, nil
	}

	func main() {
		demo := wechat.NewApplication("/demo", "demo_secret", "appid", "secret", false)

		btn11 := entry.NewButton("link").URL("http://github.com/liujianping/wechat")
		btn12 := entry.NewButton("click").Event("event_click")
		demo.Menu(entry.NewMenu(btn11, btn12))


		echo := wechat.NewApplication("/echo", "echo_secret", "appid", "secret", false)

		btn21 := entry.NewButton("link").URL("http://github.com/liujianping/wechat")
		btn22 := entry.NewButton("click").Event("event_click")
		echo.Menu(entry.NewMenu(btn21, btn22))

		serv := wechat.NewServer(":8080")
		
		serv.Application(demo, DemoHandle)
		serv.Application(echo, EchoHandle)

		serv.Start()
	}

````
### 微信测试号

[申请](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login)

### 例子

参考 [demo](https://github.com/liujianping/wechat/blob/master/demo/demo.go) 实现.

### 老版本

[README](https://github.com/liujianping/wechat/blob/v0.1/README.md)



