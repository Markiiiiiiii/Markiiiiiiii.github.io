---
title: tgbotapi创建Bot时的代理设置
date: 2021-01-12 11:51:33
tags:
- go
categories:
- 编程
---
## 使用go-telegram-bot-api的代理设置
代码如下：
```
proxyUrl, err := url.Parse("http://127.0.0.1:8001") //设置代理http或sock5
	myClient := &http.Client{Transport: &http.Transport{Proxy: http.ProxyURL(proxyUrl)}} //使用代理
	bot, err := tgbotapi.NewBotAPIWithClient("Telegram Token", myClient)
```
#### 注意 
不能使用tgbotapi的NewBotAPI方法，该方法的传参只能是一个string类型的Token。
源码如下：
```
func NewBotAPI(token string) (*BotAPI, error) {
	return NewBotAPIWithClient(token, APIEndpoint, &http.Client{})
}
```
实际上还是调用的NewBotWithClient(),源码如下。

```
func NewBotAPIWithClient(token, apiEndpoint string, client HttpClient) (*BotAPI, error) {
	bot := &BotAPI{
		Token:           token,
		Client:          client,
		Buffer:          100,
		shutdownChannel: make(chan interface{}),

		apiEndpoint: apiEndpoint,
	}

	self, err := bot.GetMe()
	if err != nil {
		return nil, err
	}

	bot.Self = self

	return bot, nil
}
```