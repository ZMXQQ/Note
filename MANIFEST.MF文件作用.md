# MANIFEST.MF文件作用及格式要求

manifest.mf文件格式如下

```
Manifest-Version: 1.0
Class-Path: xxx1.jar xxx2.jar xxx3.jar xxx4.jar
  xxx5.jar
Main-Class: com.example.demo.DemoApplication

```

包括Manifest版本，Class-Path（类加载器通过这个路径找到要用的jar包）和Main-Class（jar文件的入口类）

注意：每项的冒号后有一个空格，Class-Path中的jar文件要用空格分开，每行不超过72字符且开头要有一个空格，最后一行结尾不能有空格。文件最后一行要用回车结束。

**jar包之间用空格分开，行首有空格，所以注意有时行开头要有两个空格**