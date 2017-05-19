# 第一章 目录结构

说到分析源码，当然第一件事就是看目录结构了。然后找到哪里是入口文件。

首先看看react项目文件都有些什么:

```
├─eslint-rules
├─fixtures
├─flow
├─mocks
├─packages
├─scripts
├─src
```

从上面的结构上看，其实有用的只有`src`，其他的都一些编译、构建的脚本，可以暂时忽略。

然后再仔细看看`src`中有些什么

```
├─addons
├─fb
├─isomorphic  
├─renders
├─shared
```

在我们的依次寻找中，发现isomorphic中有一个React.js，打开一看，发现了新世界，原来这个就是react奇妙世界的入口。

因此，我们来看看一下isomorphic目录下都包含了什么

```
├─children
├─classic
├─hooks
├─modern
React.js
```



