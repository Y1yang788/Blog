# LFSR
#### 概念了解
LFSR是指线性反馈移位寄存器，给定一定的输出，将该输出的线性代数再用作输入的移位寄存器，异或运算是最常见的单比特线性代数：
对寄存器的某些位进行异或操作后作为输入,再对寄存器中的各比特进行整体移位.该结构具有结构简单,运行速度快的特点,常被应用于伪随机数和伪随机噪声的生成中.同时,该原件常与流密码相关部分联合使用.
#### 原理
LFSR是FSR的一种，还有一种是NFSR(非线性)
![alt text](https://pic.imgdb.cn/item/676ceddfd0e0a243d4ea7fa7.png)
函数可表示为:![](https://pic.imgdb.cn/item/676cee04d0e0a243d4ea7faa.png)

---

这里给不了解布尔运算的做一个基础补充：
布尔运算分为与运算，或运算，异或运算和非运算
#### 与运算：
![](https://pic.imgdb.cn/item/676cee17d0e0a243d4ea7fab.png)
#### 或运算
![](https://pic.imgdb.cn/item/676cee22d0e0a243d4ea7fb1.png)
#### 异或运算
![](https://pic.imgdb.cn/item/676cee2bd0e0a243d4ea7fb7.png)
#### 非运算
![](https://pic.imgdb.cn/item/676cee35d0e0a243d4ea7fba.png)

---
- an+1=c1an⊕c2an-1⊕...⊕cna1
- an+2=c1an+1⊕c2an⊕...⊕cna2
- ...
- an+i=c1an+i-1⊕c2an+i-2⊕...⊕cnai(i=1,2,3...)
  
例如:
![](https://pic.imgdb.cn/item/676cee4bd0e0a243d4ea7fbc.png)
题型:1.已知反馈函数，输出序列，求逆推出初始状态
```
from flag import flag
assert flag.startswith("flag{")
assert flag.endswith("}")
# 作用：判断字符串是否以指定字符 开头或结尾
assert len(flag)==25

def lfsr(R,mask):
    output = (R << 1) & 0xffffff #将R向左移动1位，bin(0xffffff)='0b111111111111111111111111'
    i=(R&mask)&0xffffff          #按位与运算符&：参与运算的两个值,如果两个相应位都为1,则该位的结果为1,否则为0
    lastbit=0
    while i!=0:
        lastbit^=(i&1)           #按位异或运算，得到输出序列
        i=i>>1
    output^=lastbit              #将输出值写入 output的后面
    return (output,lastbit)

R=int(flag[5:-1],2)  #flag为二进制数据
mask    =   0b1010011000100011100

f=open("key","ab")   #以二进制追加模式打开
for i in range(12):
    tmp=0
    for j in range(8):
        (R,out)=lfsr(R,mask)
        tmp=(tmp << 1)^out
    f.write(chr(tmp))   #将lfsr输出的序列每8个二进制为一组，转化为字符，共12组
f.close()
```
思路:
![](https://pic.imgdb.cn/item/676cee57d0e0a243d4ea7fbf.png)
第一种方法:
```
from Crypto.Util.number import*

f = open('key.txt','rb').read()
r = bytes_to_long(f)
bin_out = bin(r)[2:].zfill(12*8)
R = bin_out[:19]    #获取输出序列中与掩码msk长度相同的值
print(R)
mask = '1010011000100011100'  #顺序 c_n,c_n-1,。。。,c_1
key =  '0101010100111000111'

R = ''
for i in range(19):
    output = 'x'+key[:18]
    out = int(key[-1])^int(output[-3])^int(output[-4])^int(output[-5])^int(output[-9])^int(output[-13])^int(output[-14])^int(output[-17])
    R += str(out)
    key = str(out)+key[:18]

print('flag{'+R[::-1]+'}')
```
第二种方法：猜seed
```
from Crypto.Util.number import*
import os,sys
os.chdir(sys.path[0])

f = open('key.txt','rb').read()
c = bytes_to_long(f)
bin_out = bin(c)[2:].zfill(12*8)   #将key文本内容转换为 2 进制数，每个字节占 8 位

R = bin_out[0:19]  #取输出序列的前19位
mask = 0b1010011000100011100

def lfsr(R,mask):
    output = (R << 1) & 0xffffffff
    i=(R&mask)&0xffffffff
    lastbit=0
    while i!=0:
        lastbit^=(i&1)
        i=i>>1
    output^=lastbit
    return (output,lastbit)

#根据生成规则，初始状态最后一位拼接输出序列
#我们可以猜测seed的第19位（0或1），如果seed19+R[:18]输出值等于R[:19]，那么就可以确定seed值了
def decry():
    cur = bin_out[0:19]      #前19位 2 进制数
    res = ''
    for i in range(19):
        if lfsr(int('0'+cur[0:18],2),mask)[0] == int(cur,2):
            res += '0'
            cur = '0'+cur[0:18]
        else:
            res += '1'
            cur = '1' + cur[0:18]
    return int(res[::-1],2)

r = decry()
print(bin(r))
```
第三种方法:
```
import os,sys
os.chdir(sys.path[0])
from Crypto.Util.number import *
key = '0101010100111000111'
mask = 0b1010011000100011100

R = ""
index = 0
key = key[18] + key[:19]
while index < 19:
    tmp = 0
    for i in range(19):
        if mask >> i & 1:
            tmp ^= int(key[18 - i])
    R += str(tmp)
    index += 1
    key = key[18] + str(tmp) + key[1:18]

print (R[::-1])
```
