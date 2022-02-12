---
title: Go语言使用阿里云实现短信发送/验证服务（gin框架）
date: 2021-12-16 11:57:24
author:
top: false
hide: false
cover: false
img: medias/featureimages/36.jpg
password:
toc: false
mathjax: false
summary: Go语言使用阿里云实现短信发送/验证服务（gin框架）
categories: Golang
tags:
- Golang
- AliYun
- Gin框架
---
# go语言使用阿里云实现短信发送/验证服务（gin框架）

### 官方示例

## 短信批量发送结果查询 CodeSample 

该项目为通过 SendSms 发送短信并查询发送的结 CodeSample，生成的代码可以通过安装[各语言的包](https://darabonba.api.aliyun.com/module/alibabacloud/CS20151215)来进行测试。

### 先决条件

- 在您开始之前，您需要注册阿里云帐户并获取您的 [凭证](https://usercenter.console.aliyun.com/#/manage/ak)

### 使用的 API

- SendBatchSms 批量发送短信，可以参考：[文档](https://help.aliyun.com/document_detail/101414.html)
- QuerySendDetails 查询短信的发送情况，可以参考：[文档](https://help.aliyun.com/document_detail/102352.html)

### 输入参数

```sh
<phoneNumbers>: 接收短信的手机号码，多个用英文逗号隔开
<signNameJson>: 短信签名名称，eg: "阿里云"
<templateCode>: 短信模板CODE
<templateParamJson>: 短信模板变量对应的实际值，eg：{"code":"1234"}
```

### 返回示例

QuerySendDetails：

```sh
{
	"TotalCount":1,
	"Message":"OK",
	"RequestId":"819BE656-D2E0-4858-8B21-B2E477085AAF",
	"SmsSendDetailDTOs":{
		"SmsSendDetailDTO":{
			"SendDate":"2019-01-08 16:44:10",
			"OutId":123,
			"SendStatus":3,
			"ReceiveDate":"2019-01-08 16:44:13",
			"ErrCode":"DELIVERED",
			"TemplateCode":"SMS_122310183",
			"Content":"【阿里云】验证码为：123，您正在登录，若非本人操作，请勿泄露",
			"PhoneNum":15298356881
		}
	},
	"Code":"OK"
}
```



### 本人测试代码如下：

##### 用户信息结构体：

```go
type UserInfo struct {
	Uid              int64
	Account          string
	Password         string
	Phone            string
	Email            string
	VerificationCode string
}
```

##### 短信发送部分代码：

```go
// GetSMSverification 通过短信发送验证码
func GetSMSVerification(c *gin.Context) {
	data := Models.UserInfo{}
	err := c.BindJSON(&data)
	if err != nil {
		Utils.HandleErr(c, err, "获取数据失败")
		return
	}

	user := &Models.UserInfo{Phone: data.Phone}
	isExist := &data
	isExist.QueryPhone(user.Phone)
	//发送验证码
	request := Models.ALiYunCommunicationRequest{}
	randcode := Utils.GenerateRandNum(6)
    //需获取AliYun的短信服务签名和短信模板
	req := request.SetParamsValue(data.Phone, signName, randcode, TemplateParam)
	err = req.SendReq()
	Utils.HandleErr(c, err, "发送失败")

	c.JSON(http.StatusOK, gin.H{
		"verification": randcode,
		"result":       "发送成功",
	})

	//记录本地验证码
	user.VerificationCode = randcode

	//给定过期时间
	timer := time.NewTimer(60 * time.Second)
	select {
	case <-timer.C:
		user.VerificationCode = "expired"
	}
	timer.Stop()
}
```



##### 短信验证部分代码：

```go
// LoginbySMS  通过SMS用户登录
func LoginbySMS(c *gin.Context) {
	data := Models.UserInfo{}
	err := c.BindJSON(&data)
	if err != nil {
		Utils.HandleErr(c, err, "获取数据失败")
		return
	}
	if len(data.VerificationCode) == 0 {
		c.JSON(http.StatusBadRequest, gin.H{
			"verification": nil,
			"result":       "获取数据失败",
		})
	}

	user := &Models.UserInfo{VerificationCode: data.VerificationCode} //填入的验证码
	isExist := &data
	isExist.QueryPhone(data.Phone)

	if len(data.VerificationCode) == 0 {
		c.JSON(http.StatusBadRequest, gin.H{
			"verification": user.VerificationCode,
			"result":       "验证码输入错误",
		})
	}
	if user.VerificationCode == "expired" {
		c.JSON(http.StatusBadRequest, gin.H{
			"verification": user.VerificationCode,
			"result":       "验证码过期",
		})
	}

	//填入的验证码与数据库的验证码进行比较
	if user.VerificationCode == data.VerificationCode {
		SetCookie(c, data.Uid)
	} else {
		c.JSON(http.StatusBadRequest, gin.H{
			"verificationCode": user.VerificationCode,
			"result":           "验证码错误",
		})
	}
}

```



## 结论：

因为短信签名需要完整的网站和App才能申请成功，不能看到实际短信情况，但在postman测试是成功返回，待网站建设完好后，再展示短信结果。。。