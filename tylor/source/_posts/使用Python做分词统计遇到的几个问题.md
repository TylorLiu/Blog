---
title: 使用Python做分词统计遇到的几个问题
date: 2020-09-04 18:49:22
tags: Python
category: blog
---
### 背景
工作中需要对几百条问题原因进行分类汇总，分析出主要是哪些方面做的不到位，并进行针对性的改进。最开始想通过Excel统计，但是很繁琐，麻烦在于：
1. 不确定需要统计哪些词句的频率
2. 需要对输入数据进行整理，使同类原因有一个共性，才方便统计，比如"XXX不熟悉"、"YYY不了解"要统一改为"不了解"或"不熟悉"，才好统计

### 分词统计
以前看过相关文章，让我想到了分词统计可以解决这类问题，搜了下，找到[这篇文章](https://blog.csdn.net/weixin_43886356/article/details/86711012)。
然后就开始照搬，遇到了一系列的问题，记录如下：
1. python2.7 安装pip20报错，查了下只能装pip9 （HUI框架需要装Python，令人费解）
2. python2.7 不支持file.open传encoding，代码报错  
    将文件编码转成GBK，代码去掉encoding参数
3. SyntaxError: Non-ASCII character '\xe6' in file  
   python运行源码报错，在第一行加 # coding=UTF-8解决
4. 无法调用Jieba库--module 'jieba' has no attribute 'Icut'
  文件名是jieba.py，与import jieba冲突。改文件名解决。
  [参考](https://github.com/fxsjy/jieba/issues/715)
5. UnicodeEncodeError: 'ascii' codec can't encode character in position 0: ordinal not in range(128) 
    未查到具体原因，怀疑是python2的问题，决定升级python3。后来判断可能是终端只能输出Ascii码，而分词后要输出中文字符
    [参考](https://stackoverflow.com/questions/20923663/unicodeencodeerror-ascii-codec-cant-encode-character-in-position-0-ordinal)
6. 下载Python官网安装包各种失败，连手机无线也慢如龟爬，找了个现成的33装了，发现装pip要setuptools，装setuptools要pip。还是要下载Python3.5以后版本，自带pip、setuptools，省心。
    一通搜索，终于在Bing上搜到到了淘宝镜像，才很快下载好。（遇到这种官网上下载慢的，要么找国内镜像，要么翻墙，省心省力。用百度搜简直就是翻垃圾堆，找了半天下载下来发现带毒。
7. 装完Python37之后，抄来的程序终于运行成功（所以搞不清问题5到底是啥原因了，以后有空了再去用Python27研究一下。PS:八成不会）。发现统计的数量有问题，“考虑”出现过42次，但是只统计到27次。
    小改了下程序，排除“考虑”，包含‘考’的词还有“考虑不周”，出现15次，是分开统计的，bingo。
    
### 修改后的代码    
````
# coding=UTF-8
import jieba

file = open("ois.txt", "r",encoding='utf-8')#此处需打开txt格式且编码为UTF-8的文本
txt = file.read()
words = jieba.lcut(txt)      # 使用jieba进行分词，将文本分成词语列表
count = {}
for word in words:            #  使用 for 循环遍历每个词语并统计个数
    if len(word) < 2:          # 排除单个字的干扰，使得输出结果为词语
        continue
    else:
        count[word] = count.get(word, 0) + 1    #如果字典里键为 word 的值存在，则返回键的值并加一，如果不存在键word，则返回0再加上1
        
exclude = ["记得", "具体", "原因"]  # 建立无关词语列表
for key in list(count.keys()):     # 遍历字典的所有键，即所有word
	if key in exclude:
		del count[key]                  #  删除字典中键为无关词语的键值对     
list = list(count.items())         # 将字典的所有键值对转化为列表
list.sort(key=lambda x: x[1], reverse=True)     # 对列表按照词频从大到小的顺序排序
for i in range(5):  #   此处统计排名前五的单词，所以range(5)
    word, number = list[i]
    print("关键字：{:-<10}频次：{:+>8}".format(word, number))
````

### 后续处理
 分词统计出来的都是单词，比如：考虑、业务、设计等等，还需要把单词组成短语，然后将相同语义的短语归入一类进行统计，可以用NotePad++计数。语义统计和展示的方式就不再赘述。