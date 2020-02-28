---
title: pmAPI
date: 2019-11-28 02:19:01
tags: 检测到数值超过阈值，通过mail和telegram通知
---

/*

使用golang语言，

通过调用 https://aqicn.org/json-api/doc/ 提供的接口，接口数据两小时更新一次

判断空气质量（pm2.5）大于50，发报警通知到telegram的一个channel或者发报警到自己的邮箱。

*/

/*

json数据示例

{

​    "status": "ok",

​    "data": {

​        "aqi": 68,

​        "idx": 1437,

​        "attributions": [{

​                "url": "https://china.usembassy-china.org.cn/embassy-consulates/shanghai/air-quality-monitor-stateair/",

​                "name": "U.S. Consulate Shanghai Air Quality Monitor"

​            },

​            {

​                "url": "http://www.semc.gov.cn/",

​                "name": "Shanghai Environment Monitoring Center(上海市环境监测中心)"

​            },

​            {

​                "url": "http://106.37.208.233:20035/emcpublish/",

​                "name": "China National Urban air quality real-time publishing platform (全国城市空气质量实时发布平台)"

​            },

​            {

​                "url": "https://waqi.info/",

​                "name": "World Air Quality Index Project"

​            }

​        ],

​        "city": {

​            "geo": [

​                31.2047372,

​                121.4489017

​            ],

​            "name": "Shanghai (上海)",

​            "url": "https://aqicn.org/city/shanghai"

​        },

​        "dominentpol": "pm25",

​        "iaqi": {

​            "co": {

​                "v": 4.6

​            },

​            "h": {

​                "v": 89.3

​            },

​            "no2": {

​                "v": 17.9

​            },

​            "o3": {

​                "v": 23.6

​            },

​            "p": {

​                "v": 1024.2

​            },

​            "pm10": {

​                "v": 21

​            },

​            "pm25": {

​                "v": 68

​            },

​            "so2": {

​                "v": 2.6

​            },

​            "t": {

​                "v": 9.7

​            },

​            "w": {

​                "v": 0.6

​            }

​        },

​        "time": {

​            "s": "2019-11-27 16:00:00",

​            "tz": "+08:00",

​            "v": 1574870400

​        },

​        "debug": {

​            "sync": "2019-11-27T17:55:33+09:00"

​        }

​    }

}

*/

## 程序

package  main

import (

​    "io/ioutil"

​    "log"

​    "net/http"

​    "encoding/json"

​    "gopkg.in/gomail.v2"

)



//检查错误

func checkErr(err error) {

​    if err != nil {

​        log.Fatalf("get: %v", err)

​    }

}



//获取数据

func Getapi() interface{} {

​    token := "643ded16//////////////////////59620b3"

​    city := "shanghai"

​    url := "https://api.waqi.info/feed/" + city + "/?token=" + token

​    res, err := http.Get(url)

​    checkErr(err)

​    defer res.Body.Close()



​    data, err := ioutil.ReadAll(res.Body)

​    checkErr(err)



​    result := map[string]interface{} {}

​    json.Unmarshal(data, &result)



​    return result

}



//利用gomail第三方库发送邮件

func SentMail(alert string) {

​    m := gomail.NewMessage()

​    m.SetHeader("From", "@@@@@@@@@com")

​    m.SetHeader("To", "@@@@@@@@@@.com")

​    m.SetHeader("Subject", "ALERT")

​    m.SetBody("text/html", alert)



​    d := gomail.NewDialer("smtp.qq.com", 465, "@qq.com", "assword")



​    // Send the email to foxmail

​    if err := d.DialAndSend(m); err != nil {

​        panic(err)

​    }

}



//利用telegram bot api发送消息到channel

func SentChannel(text string) {

​    token := "1046537098:AAFWX7DzSPVl8wnGVNtVJeVR6g-LMBWEsxA"

​    chatId := "-1001407192363"

​    url := "https://api.telegram.org/bot" + token + "/sendMessage?chat_id=" + chatId + "&text=" + text

​    res, err := http.Get(url)

​    checkErr(err)

​    defer res.Body.Close()

}



func main() {

​    resp := Getapi()

​    result := resp.(map[string]interface{})



​    //city := result["data"].(map[string]interface{})["city"].(map[string]interface{})["name"]

​    //currtime := result["data"].(map[string]interface{})["time"].(map[string]interface{})["s"]

​    pm25 := result["data"].(map[string]interface{})["iaqi"].(map[string]interface{})["pm25"].(map[string]interface{})["v"]



​    pm25_2 := pm25.(float64)

​    if int(pm25_2) > 50 {

​        alert := "警告：上海pm25数值大于50"

​        SentMail(alert)

​        SentChannel(alert)

​    }

}

### 运行结果示意：

![avatar](C:\Users\Elixir\Desktop\Sn1601.png)

<img src="C:\Users\Elixir\Desktop\Sn1666.jpg" alt="avatar" style="zoom:50%;" />

### 参考

1 《Go入门指南》 https://github.com/unknwon/the-way-to-go_ZH_CN

2  GoDoc   https://godoc.org/gopkg.in/gomail.v2#example-package

3  Telegram-Bot-API  https://core.telegram.org/bots/api
