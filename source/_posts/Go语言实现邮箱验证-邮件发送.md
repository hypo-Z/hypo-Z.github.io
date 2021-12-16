---
title: Go语言实现邮箱验证/邮件发送
date: 2021-12-15 16:02:27
author:
top: false
hide: false
cover: false
password:
toc: false
mathjax: false
summary: Go语言实现邮箱验证/邮件发送
categories: Golang
tags:
- 随笔
- 文章
- Gin框架
- SMTP
---

# Go语言实现邮箱验证/邮件发送

### 1.首先设置邮箱的smtp

SMTP服务器就是邮件代收发服务器，由邮件服务商提供，常见的SMTP服务器端口号：
QQ邮箱：SMTP服务器地址：smtp.qq.com（端口：587）
雅虎邮箱: SMTP服务器地址：smtp.yahoo.com（端口：587）
163邮箱：SMTP服务器地址：smtp.163.com（端口：25）
126邮箱: SMTP服务器地址：smtp.126.com（端口：25）
新浪邮箱: SMTP服务器地址：smtp.sina.com（端口：25）

登录邮箱账户 设置开启SMTP 并获取授权码

我使用的是QQ邮箱进行测试

![qqmail](https://hypo-pictrue-1308430808.cos.ap-shanghai.myqcloud.com/hypo.ltd-%E6%96%87%E4%BB%B6%E8%AE%BF%E9%97%AE%E5%AD%98%E5%82%A8/qqmail_test.jpg)

QQ邮箱默认关闭SMTP服务，将IMAP/SMTP服务打开，跟着流程做后你会得到令牌密码。

### 代码实现如下：

```go
func SendToMail(rand, to string) error {

	subject := "动态验证码"
	user := "user@qq.com"
	password := "**********" //输入刚得到的令牌
	host := "smtp.qq.com:587"
	body := "您的动态验证码为：" + rand + "，您正在进行密码重置操作，如非本人操作，请忽略本邮件！"

	sendUserName := "senderName" //发送邮件的人名称
	fmt.Println("send email")
	hp := strings.Split(host, ":")
	auth := smtp.PlainAuth("", user, password, hp[0])
	var content_type string
	mailtype := ""
	if mailtype == "html" {
		content_type = "Content-Type: text/" + mailtype + "; charset=UTF-8"
	} else {
		content_type = "Content-Type: text/plain" + "; charset=UTF-8"
	}

	msg := []byte("To: " + to + "\r\nFrom: " + sendUserName + "<" + user + ">" + "\r\nSubject: " + subject + "\r\n" + content_type + "\r\n\r\n" + body)
	send_to := strings.Split(to, ";")
	err := smtp.SendMail(host, auth, user, send_to, msg)
	fmt.Println("send email")
	if err != nil {
		fmt.Println("Send mail error!")
		fmt.Println(err)
	} else {
		fmt.Println("Send mail success!")
	}
	return err
}

func main(){
	//发送验证码
	randcode := Utils.GenerateRandNum(6)//随机6位数
	err = Models.SendToMail(randcode,user.Email)
    log.Printf("验证码发送失败,err:%s",err)
	c.JSON(http.StatusOK, gin.H{
		"verification": randcode,
		"result":       "发送成功",
	})

}
```
成功发送：

![success](/home/zhb/pictrue/qqmail_test_success.jpg)




### 遇到的几个坑

- smtp.SendMail 后出现 EOF 失败

**解决方法：** QQmail 的 465 端口用于通过 TLS 连接，但 SendMail 需要普通的旧 TCP。尝试连接到端口 587。SendMail 将在可用时自动升级到 TLS（在本例中就是这种情况）。

还需要解决的问题：

- 这个只能单人进行验证，如果多个人同时验证会造成验证码记录覆盖，导致用户输入验证码错误。

代解决。。。