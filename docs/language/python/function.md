# 功能点

## try、except 抛出异常 赋予 给变量
可以用这个执行很多报错才能执行的程序。  

``` python
try:
    xxxx1
except Exception as e:
    print(e) #打印所有异常到屏幕
# -----
try:
    xxxx1
except TypeError as e:
    print(e) #打印类型异常到屏幕
```

## pip导出包列表，并下载whl文件，并导入
如果有场景需要离线安装时使用。

``` shell
pip freeze > requirements.txt  
pip install -r requirement.txt
pip download -r requirements.txt -d pip3_cache  # -d 指定路径
pip3 download -r requirements.txt -i https://pypi.douban.com/simple  # -i 指定源
根据requirements安装已下载的pypi包
pip2 install --no-index --find-links=pip2_cache -r py2_requirements.txt  --user
pip3 install --no-index --find-links=pip3_cache -r requirements.txt  --user
``` 

## pip升级导致无法使用
一般apt-get install 新的python之后会出现这个状况，修改对应的pip的脚本即可。  

``` shell
sudo sed -i -e 's/main()/__main__.main()/' /usr/bin/pip
sudo sed -i -e 's/main()/__main__.main()/' /usr/bin/pip2
sudo sed -i -e 's/main()/__main__.main()/' /usr/bin/pip3
```

## pip切换源

``` shell
# 切换到清华源
pip config set global.index-url https://pypi.python.org/simple
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
``` 

## pip升级

``` shell
# 简单升级：
pip install --upgrade pip

# 联网(如果不能联网的话，无法使用pip更新自己，需要python3 -m pip 来更新)
python3 -m pip install --upgrade pip --user

# 离线升级,本地whl
python3 -m pip install --upgrade pip.whl --user
## 如果加--user，相当于将pip重新安装到了自己用户下，需要增加path到~/.local/bin
## 如果不加--user，则需要增加sudo，可能会导致/usr/bin/pip 坏掉，需要修改相关pip的内容
``` 

## 调用函数，后面需要加括号

``` shell
# 函数
def test():
    a=1
    return a

# 此时a输出的是在内存中的位置，type(a)的类型是一个function
a=test
print(a)
print(type(a))

# 此时a输出的是值1，type(a)是一个字符串类型
a=test()
print(a)
print(type(a))
``` 
