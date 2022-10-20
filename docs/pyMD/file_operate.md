# 文件的理解
- 文件是数据的抽象和集合
- 文件是存储在辅助存储器上的数据序列
- 文件是数据存储的一种形式
- 文件展现形态：文本文件和二进制文件

```py
                           文本or二进制，读or写
  <变量名> = open( <文件名>,<打开模式> )
 文件句柄        文件路径和名称
```

## 文件的打开
| 文件打开模式 | 描述 | 
| :---: | :---: |
| 'r' | 只读模式，默认值，如果文件不存在，返回FileNotFountError | 
| 'w' | 覆盖写模式，文件不存在则创建，存在则完全覆盖 | 
| 'x' | 创建写模式，文件不存在则创建，存在则返回FileExistsError | 
| 'a' | 追加写模式，文件不存在则创建，存在则在文件最后追加内容 | 
| 'b' | 二进制文件模式 | 
| 't' | 文本文件模式，默认值 | 
| '+' | 与r/w/x/a一同使用，在原功能基础上增加同时读写功能 | 

- 文件关闭：```f.close()```

## 文件内容的读取
| 文件内容的读取 | 描述 | 
| :---: | :---: |
| f.read(size = -1) | 读入全部内容，如果给出参数，读入前size长度 | 
| f.readline(size = -1) | 读入一行内容，如果给出参数，读入改行前size长度 | 
| f.readlines(hint = -1) | 读入文件所有行，以每行为元素形成列表，如果给出参数，读入前hint行 | 

## 数据的文件写入
| 文件内容的读取 | 描述 | 
| :---: | :---: |
| f.write(s) | 向文件写入一个字符串或字节流 | 
| f.writelines(lines) | 将一个元素全为字符串的列表写入文件(直接拼接字符串) | 
| f.tell(self, *args, **kwargs) | 返回当前文件操作的光标位置 | 
| f.flush(self, *args, **kwargs) | 把文件从内存buffer里强制刷新到硬盘 | 
| f.seek(offset) | 改变当前文件操作指针的位置，offset含义如下 | 

- offset的定义：0 -> 文件开头；1 -> 当前位置；2 -> 文件结尾

## 数据存储格式
- CSV：Comma-Separated Values
- 国际通用的一二维数据存储格式，一般.csv扩展名
- 每行一个一维数据，采用逗号分隔，无空行
- Excel和一般编辑软件都可以读入或另存为csv文件
- 如果某个元素缺失，逗号仍要保留
- 二位数据的表头可以作为数据存储，也可以另行存储
- 逗号为英文半角，逗号与数据之间无需空格

#### 以命令行实现对文件的全局替换
- 执行 ```python  replace.py  oldStr newStr filename```

```py
import sys

# sys.argv 返回命令行参数
oldStr = sys.argv[1]
newStr = sys.argv[2]
file_name = sys.argv[3]

f = open(file_name, "r+")   # open file
data = f.read() # read all content
str_count = data.count(oldStr)  # count all oldStr

new_data = data.replace(oldStr,newStr)

f.seek(0)   # 将文件指针指向行首
f.truncate()    # 清空全部旧内容
f.write(new_data)   # 写入新内容
f.close()   # 关闭文件

print(f"找到{oldStr}共{str_count}处，以全部替换为{newStr}...")
```