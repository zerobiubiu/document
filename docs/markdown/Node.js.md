# Node.js

## 模块导入

用`require`导入模块

`const	常量=require('模块');`

## 文件系统模块——fs

> fs模块是Node.js官方提供的用来操作文件的模块。它提供了一系列的方法和属性，用来满足用户对文件的操作需求
>
> 使用需要导入fs模块

fs.readFile(path,[options],callback)方法，用来读取指定文件中的内容
path：地址，字符串格式，必填
options：读取编码方式，字符串格式，选填
callback：文件读取完成后通过回调函数拿到读取的结果，函数，必填

fs.writeFile()方法，用来向指定文件中写入内容