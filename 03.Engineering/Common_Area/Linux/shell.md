**Shell**

    命令行语言解释器，接收用户输入的命令行，交给操作系统执行，并将执行结果反馈给用户，提供用户和操作系统交互的接口。  

**Shell标准化  
**

在Shell伴随Unix发展过程中，各种变种、分支发展可谓百花齐放（如下图），带来丰富功能的同时，也带来了跨平台、兼容性等问题。为解决这些问题IEEE在POSIX规范中对Shell进行了标准化。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/GFYVpmKsImDuGJ6tYzYvlxsiaByf6J2SKkBfiaz0jhVnZceekD0thn1Cez0okBphE2xykgFO88ibRTVhGtehjjNqg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

# **什么是 sh ?**

```
#!/bin/sh
```

- 在POSIX对Shell进行标准化前，sh主要是指Unix系统的(Bourne **Sh**ell)。在标准化后，我们可以认为sh是Shell的规范标准，而其具体实现由dash、csh、ksh来完成，bash也可以被视为sh的一种实现（对比python与 CPython、JPython）。
    

  

- `由于sh是一个规范而不是实现，大多数POSIX系统上的/bin/sh是一个符号链接（或硬链接），指向实际的实现版本（如下图所示）。`
    

`   `

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/GFYVpmKsImDKtSlLXWCfsv0QhHm5Hc5zibDE2SBgcD1wOWicEYibZHiaMIP054QnibbqJAxYk2Aj8QVYpONZqaBmsDQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

# **什么是 bash ?**

```
#!/bin/bash
```

- Bash（GNU的**B**ourne-**A**gain **SH**ell）是一个完整实现IEEE POSIX和Open Group Shell规范的命令行解释器，具有交互式命令行编辑、支持作业控制（在支持的架构上）、类似csh的功能，如历史替换和花括号展开，以及其他一系列功能。
    
      
    
- bash是sh的超集，但并不是一个符合POSIX标准的Shell，它是POSIX Shell语言的一个方言。