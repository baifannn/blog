---
title: Redis02-发布和订阅
date: 2020-03-24 21:54:24
tags:
- redis
categories:
- redis学习
---

发布和订阅又称pub/sub，订阅者listener负责订阅频道channel，发送者publisher负责向频道发布二进制字符串消息（binary string message）
redis提供了5个发布与订阅的命令

| 命令         | 用例和描述                                                   |
| ------------ | ------------------------------------------------------------ |
| SUBSCRIBE    | SUBSCRIBE channel [channel..]订阅一个或者多个频道            |
| UBSUBSCRIBE  | SUBSCRIBE [channel..]退订一个或者多个频道，如果没指定频道，那么退订所有频道 |
| PUBLISH      | PUBLISH channel message 向指定频道发布消息                   |
| PSUBSCRIBE   | PSUBSCRIBE pattern 订阅与给定模式相匹配的所有频道            |
| PUNSUBSCRIBE | PUNSUBSCRIBE [pattern] 订一个或者多个频道，如果没指定频道，那么退订所有频道 |

python展示：

```python
def publisher(n):
  time.sleep(1)
  for i in xrange(n):
    conn.publish('channel',1)
    time.sleep(1)

def run_pubsub():
  threading.Thread(target=publisher,arges={3,}).start()
  pubsub=conn.pubsub()
  pubsub.subscribe(['channel'])
  count=0
  for item in pubsub.listen():
    print item
    count +=1
    if count==4:
      pubsub.unsubscribe()
    if count==5：
    	break
rub_pubsub()
{'pattern':None,'type':'subscribe','channel':'channel','data':1L}
{'pattern':None,'type':'message','channel':'channel','data':'0'}
{'pattern':None,'type':'message','channel':'channel','data':'1'}
{'pattern':None,'type':'message','channel':'channel','data':'2'}
{'pattern':None,'type':'unsubscribe','channel':'channel','data':0L}
```

风险

1. 旧版reids速度不够快，会因为积压导致redis输出缓冲体积变大，导致redis变慢，甚至崩溃，新版redis不会出现这个问题
2. 数据传输的可靠性。发布订阅模式中断线，会丢失断线期间发送的所有消息