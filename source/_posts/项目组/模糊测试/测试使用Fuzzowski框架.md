---
title: 测试使用fuzzowski框架
tags:
  - - fuzz
  - - socket
categories:
  - [项目组, 模糊测试]
abbrlink: 931e55d9
date: 2020-11-01 15:08:32
---

fuzzowski是一个模糊测试器，目前主要支持LPD、IPP、BACnet、Modbus协议。本次测试使用的Modbus部分内容进行修改测试，实际上是通过测定规则生成数据通过tcp不断进行发包。

### Socket脚本

```python
import socket

HOST = '0.0.0.0'
PORT = 6678
count = 1

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind((HOST, PORT))
s.listen(10)

print('Server start at: %s:%s' %(HOST, PORT))
print('wait for connection...')

while True:
    conn, addr = s.accept()
    data = conn.recv(1024)

    f = open("test1.txt",'a')
    f.write("{:<5d}:".format(count))

    for c in data:
        f.write('%02X' % ord(c))
        f.write(' ')

    f.write('\n')
    f.close()

    print("{:<5d}:Connected by{}".format(count, addr))

    count+=1
    conn.send(data)
    conn.close()

```

### Fuzzowski设置

先使用简单的`python3 -m fuzzowski 127.0.0.1 6789 -f modbus`

经过输出测试，发现变动一个`word`型会产生**140**种十六进制数据，变动一个`byte`型会产生**116**种十六进制数据，为固定产生非随机生成。

通过测试modbus的read_coil模块，主要为`/fuzzowski-master/fuzzowski/fuzzers/modbus/modbus.py`中如下代码：

```python
s_initialize("modbus_read_coil")
with s_block("modbus_head"):
    s_word(0x0001,name='transId',fuzzable=True)
    s_word(0x0000,name='protoId',fuzzable=False)
    s_word(0x06,name='length')
    s_byte(0xff,name='unit Identifier',fuzzable=False)
    with s_block('pdu'):
        s_byte(0x01,name='funcCode read coil memory',fuzzable=False)
        s_word(0x0000,name='start address')
        s_word(0x0000,name='quantity')

s_initialize('read_holding_registers')
with s_block("modbus_head"):
    s_word(0x0001,name='transId',fuzzable=False)
    s_word(0x0002,name='protoId',fuzzable=False)
    s_word(0x06,name='length')
    s_byte(0xff,name='unit Identifier',fuzzable=False)
    with s_block('read_holding_registers_block'):
        s_byte(0x01,name='read_holding_registers')
        s_word(0x0000,name='start address')
        s_word(0x0000,name='quantity')
```

其中，fuzzable默认为True。设定为True的字段会依次产生变动生成数据，而非交叉产生，即一个数据包仅会有一个字段被fuzz。

举例修改上述模块为：

```python
s_initialize("modbus_read_coil")
with s_block("modbus_head"):
    s_byte(0x68,name='funcCode read coil memory1',fuzzable=False)
    s_byte(0x04,name='funcCode read coil memory2',fuzzable=False)
    s_byte(0x07,name='funcCode read coil memory3',fuzzable=True)
    s_byte(0x00,name='funcCode read coil memory4',fuzzable=False)
    s_byte(0x00,name='funcCode read coil memory5',fuzzable=False)
    s_byte(0x00,name='funcCode read coil memory6',fuzzable=False)
    
s_initialize('read_holding_registers')
with s_block("modbus_head"):
    s_word(0x0001,name='transId',fuzzable=False)
    # s_word(0x0002,name='protoId',fuzzable=False)
    # s_word(0x06,name='length')
    # s_byte(0xff,name='unit Identifier',fuzzable=False)
    # with s_block('read_holding_registers_block'):
    #     s_byte(0x01,name='read_holding_registers')
    #     s_word(0x0000,name='start address')
    #     s_word(0x0000,name='quantity')
```

即只会生成`68 04 07 [00] 00 00`其中会对第4字节的fuzz数据，共112条。

通过`python3 -m fuzzowski 127.0.0.1 6789 -f modbus -tn -rt 1 -r read_coil`调用modbus的read_coil模块（实际上只是生成测试数据发送tcp包，设置为不处理响应）。

![](测试使用fuzzowski框架/modbus-fuzz.png)

