# dingding
#!/usr/bin/env python3
# -*- coding: utf-8-*-

import time
import hmac
import hashlib
import base64
import requests
import dingtalk

import json

from urllib import parse


# 获取北京市天气等信息,发送消息到钉钉

def SendMessage():

    
    host = 'https://jisuqgtq.market.alicloudapi.com'
    path = '/weather/query'
    method = 'GET'
    appcode = 'ca56f7df97bf4033ab41de817c641068'
    querys = 'city=北京市&citycode=131'
    url = host + path + '?' + querys

    headers = {'Authorization': 'APPCODE ca56f7df97bf4033ab41de817c641068',
               'Content-Type': 'application/json; charset=UTF-8'}

    res = requests.get (url, headers=headers)

    reson = res.text

    respond = json.loads (reson)

    city = respond['result']['city']
    date = respond['result']['date']
    week = respond['result']['week']
    weather = respond['result']['weather']
    temp = respond['result']['temp']
    winddirect = respond['result']['winddirect']
    windpower = respond['result']['windpower']
    ziwaixian = respond['result']['index'][2]['iname'] + respond['result']['index'][2]['ivalue']
    kongqizhishu = respond['result']['index'][5]['iname'] + respond['result']['index'][5]['ivalue']

    #     return city,date,week,weather,temp,winddirect,windpower,ziwaixian,kongqizhishu

    # 时间戳
    timestamp = str (round (time.time () * 1000))
    # 钉钉加签
    secret = 'SECbb31344613f5730dad068081f967e65390c2368f230f5d4c357dd844ea50d193'
    secret_enc = secret.encode ('utf-8')
    string_to_sign = '{}\n{}'.format (timestamp, secret)
    string_to_sign_enc = string_to_sign.encode ('utf-8')
    hmac_code = hmac.new (secret_enc, string_to_sign_enc, digestmod=hashlib.sha256).digest ()
    sign = urllib.parse.quote_plus (base64.b64encode (hmac_code))
    print (timestamp)
    print (sign)
    # 钉钉机器人中的Webhook+sign+timestamp
    url = 'https://oapi.dingtalk.com/robot/send?access_token=849a0e39e5e07f7d176ebd7b27ca8a75ae7d550865ea5179b5be2fb57206f131&timestamp={0}&sign={1}'.format (
        timestamp, sign)

    headers = {"Content-Type": "application/json;charset=utf-8"}

    data1 = {

        "msgtype": "markdown", "markdown": {"title": "北京天气",
            "text": "北京天气" + "\n >" + city + ", \t" + date + ", \t" + week + ",\t" + weather + ", \t" + "平均气温" + temp + ",\t" + winddirect + ", \t" + windpower + ", v" + ziwaixian + ", \t" + kongqizhishu + "\n > ![screenshot](https://mat1.gtimg.com/pingjs/ext2020/tianqi/mobilev2/982194185a079fad9fd6d7eac312b378.png)\n > @樊舒 发布[天气](http://weathernew.pae.baidu.com/weathernew/pc?query=%E5%8C%97%E4%BA%AC%E5%A4%A9%E6%B0%94&srcid=4982&city_name=%E5%8C%97%E4%BA%AC&province_name=%E5%8C%97%E4%BA%AC) \n"},
        "at": {"atMobiles": [

        ], "atUserIds": [

        ], "isAtAll": False}}

    data = json.dumps (data1)

    print (type (data))

    response = requests.post (url, data=data, headers=headers)

    print (response.text)


if __name__ == '__main__':
    # 程序运行时调用方法
    SendMessage ()
