# .NET源码解析

## 第一章：简介

1. [开源版.NET（SSCLI）简介](https://github.com/shiyimin/shiyimin.github.io/blob/master/sscli/chapter1/%E5%87%86%E5%A4%87CLR%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E7%8E%AF%E5%A2%83.md)
1. SSCLI架构
1. 准备SSCLI源码阅读环境
1. SSCLI编译过程简介
1. 调试SSCLI程序
  1. [在SSCLI中调试托管代码](https://github.com/shiyimin/shiyimin.github.io/blob/master/sscli/chapter1/%E5%9C%A8SSCLI%E4%B8%AD%E8%B0%83%E8%AF%95%E6%89%98%E7%AE%A1%E4%BB%A3%E7%A0%81.md)
  2. 调试SSCLI非托管代码
1. 启动CLR

## 第二章：.NET类型系统

1. .NET对象布局
  1. Object对象布局
  1. String对象布局
  1. Array对象布局
  1. 已释放的对象
  1. Object header
  1. Sync块
  1. 使用sos查看对象信息
2. .NET类型
  1. 类型在运行时的布局
  1. 函数表
  1. EEClass和MethodDesc
  1. 泛型实现
  1. 使用sos查看类型信息
3. 值类型
4. 装箱和拆箱的实现
5. Type Loader

## 第三章：.NET装配件

1. PE格式简介
1. 元数据格式
1. 元数据表类型
1. Blob流
1. Assembly Loader
1. Fusion

## 第三章：应用程序域

## 第四章：JIT

1. IL简介
1. IL基础
1. 用IL写程序
1. JIT设计
1. IL验证
1. NGEN


## SSCLI中的编译器源码
1. C#编译器
1. MS JScript编译器
1. C编译器
1. MSIL编译器
1. MSIL反编译器