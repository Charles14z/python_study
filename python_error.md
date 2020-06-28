# python error
## 无法定位程序输入点 OPENSSL_sk_new_reserve 于动态链接库Anaconda3\Library\bin\libssl-1_1-x64.dll上”的解决办法
如果Anaconda3\DLLs下的libssl-1_1-x64.dll文件和Anaconda3\Library\bin下的libssl-1_1-x64.dll的日期和大小都不一样，应该把DLLs里的libssl-1_1-x64.dll文件复制粘贴到bin里。
移动之前先备份


## 'utf-8' codec can't decode byte 0xc8 in position 2: invalid continuation byte
出现异常报错是由于设置了decode()方法的第二个参数errors为严格（strict）形式造成的，因为默认就是这个参数，将其更改为ignore等即可。例如:
```python
line.decode("utf8","ignore")
```

