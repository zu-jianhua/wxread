## attention     

1.只需要完成签到将num次数200->2，num每次为30秒，200即100min。<br>
2.对于issue中提出的“阅读时间没有增加”，“增加时间与刷的时间不对等”建议替换capture.py中的【headers】、【cookies】字段，【data】字段保留。


## 序

开发这个脚本主要是为了挑战赛刷时长和天数，由于本人偶尔看书有时来不及签到导致入场费打了水漂。网上找了一些相关代码，发现高赞的自动阅读需要挂虚拟器模拟或者用ADB模拟，实现一点也不优雅。于是抓了一下官网接口，分析了官网的源码JS逆向了一部分代码，编写了Python挂机脚本兼有自动更新Cookie与推送到微信的功能。只需要一次部署，长时间挂机。

## 脚本介绍

针对微信读书阅读挑战赛编写的脚本：

1. 能进行刷阅读时长，且时长默认计入排行榜、挑战赛等。（指定200分钟）
2. 可部署服务器每天定时运行脚本并推送到微信。
3. 一次抓包，长时间使用。对于Cookie更新问题给出了自动获取Cookie更新值的解决方案。
4. 比较市面上的ADB调试器、自动阅读器，本脚本实现了轻量化编写，部署服务器即可运行，无需更多环境条件。
5. 脚本JS逆向分析各接口请求，分析各字段的拼接方式，并对字段进行加密、计算处理使得服务器能够成功响应（`{'succ': 1, 'synckey': 2060028311}`，表示数据字段正常）。

## 操作步骤（v3.0）

1、脚本逻辑还是比较简单的，main.py与push.py代码不需要改动。在微信阅读官网 [微信读书 (qq.com)](https://weread.qq.com/) 搜索【三体】点开阅读点击下一页进行抓包，抓到`read`接口 `https://weread.qq.com/web/book/read`，如果返回格式正常（如：

```
json复制代码{
  "succ": 1,
  "synckey": 564589834
}
```

右键复制为Bash格式，然后在 [Convert curl commands to Python (curlconverter.com)](https://curlconverter.com/python/) 转化为Python脚本，复制需要的headers与cookies字段替换到`capture.py`（data字段保留），运行`main.py`即可，依赖自行安装。

2、服务器运行，在你的服务器上有Python运行环境即可，使用`cron`定义自动运行。（如：

```
bash复制代码
0 2 * * * /www/server/pyporject_evn/wxread_venv/bin/python3 /root/wxread/main.py >> /root/wxread/logs/$(date +\%y-\%m.\%d)_sout.log 2>&1
```

意思为：【在每天两点时刻使用python所在位置编译器运行某个路径下的main.py脚本，同时将输出按每天的日期格式输出到对应日志中】可供参考）。

3、Pushplus推送，更改你的Token即可。
## 截图展示

#### 1、运行结果

![image-20241004115421978](pic/image-20241004115421978.png)

#### 2、接口抓取

![image-20241004115513846](pic/image-20241004115513846.png)

#### 3、JS逆向

![image-20241004115545324](pic/image-20241004115545324.png)

#### 3、显示成效(测试近一个月全部正常运行)

![352c71c4cdd2e16e84cb9239499573a1](pic/352c71c4cdd2e16e84cb9239499573a.jpg)

#### 4、服务器自动运行指令

![image-20241004120026766](pic/image-20241004120026766.png)

#### 5、完成推送


![5ed32774727aadb47aeb32ca21db8342](pic/5ed32774727aadb47aeb32ca21db8342.jpg)


### 字段解释

- `appId`: `"wbxxxxxxxxxxxxxxxxxxxxxxxx"` ✔
  - 应用的唯一标识符。

- `b`: `"ce032b305a9bc1ce0b0dd2a"` ✔
  - 书籍或章节的唯一标识符。

- `c`: `"0723244023c072b030ba601"` ✔
  - 内容的唯一标识符，可能是页面或具体段落。

- `ci`: `60` ✔
  - 章节或部分的索引。

- `co`: `336` ✔
  - 内容的具体位置或页码。

- `sm`: `"[插图]威慑纪元61年，执剑人在一棵巨树"` ✔
  - 当前阅读的内容描述或摘要。

- `pr`: `65` ✔
  - 页码或段落索引。

- `rt`: `88` ✔
  - 阅读时长或阅读进度。

- `ts`: `1727580815581` ✔
  - 时间戳，表示请求发送的具体时间（毫秒级）。

- `rn`: `114`
  - 随机数或请求编号，用于标识唯一的请求。

- `sg`: `"bfdf7de2fe1673546ca079e2f02b79b937901ef789ed5ae16e7b43fb9e22e724"`
  - 安全签名，用于验证请求的合法性和完整性。

- `ct`: `1727580815` ✔
  - 时间戳，表示请求发送的具体时间（秒级）。

- `ps`: `"xxxxxxxxxxxxxxxxxxxxxxxx"` ✔
  - 用户标识符或会话标识符，用于追踪用户或会话。

- `pc`: `"xxxxxxxxxxxxxxxxxxxxxxxx"` ✔
  - 设备标识符或客户端标识符，用于标识用户的设备或客户端。

- `s`: `"fadcb9de"`
  - 校验和或哈希值，用于验证请求数据的完整性。

## github action部署
- `WXREAD_HEADERS`: 微信读书请求头
- `WXREAD_COOKIES`: 微信读书cookies
- `READ_NUM`: (可选) 阅读时长, 每一次代表30秒，比如你想刷1个小时这里填120，你只需要签到这里填2次。默认200
- `PUSH_METHOD`: (可选) 推送方式，可选`pushplus`或`telegram`
- `PUSHPLUS_TOKEN`: (可选) pushplus token
- `TELEGRAM_BOT_TOKEN`: (可选) telegram bot token
- `TELEGRAM_CHAT_ID`: (可选) telegram chat id