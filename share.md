### 获得大家的认同感
日常开发中肯定遇到过这种情况, 这次要在原有的需求上改点东西, 打开相关的代码后看到了一个屎山, 往往有两种结果
1. 薛定谔的重构: 我就看以前的代码不爽, 重构一波, 不知道重构的对不对, 只要代码没上线, 就是没bug
2. 不敢重构: 逻辑太多, 不敢改, 只好找个角落, 加个 if else, 往屎山上贡献一份自己的力量

### 过一遍 README.md
### 过一遍 Store.java
代码烂地不行, 应该是屎山(青春版), 有可能业务代码比这个还烂

### 任务分解
1. 加测试
2. 小步重构, 频繁测试
3. 找出&修复bug
4. 添加新的商品

### 加测试
测试用例一定要基于代码的逻辑去写, 看这个屎山这么多 if else, 我要写多少测试才能覆盖全部逻辑?<br>
这里介绍一个测试工具 [ApprovalsTests](https://approvaltests.com/), 这个工具能将给定的参数做笛卡尔积然后传到方法里, 最后形成一个输出输出的KV文件, 这个文件就相当于当前方法的快照<br>

> 跳转到测试类(Go To Test): command + shift + t

使用ApprovalTests只需要两步:
1. 识别输入与输出
2. 让给定参加尽可能覆盖100%的代码

可以看到这个方法既没有参数也没有输出, 那怎么搞?<br>
这个输入和输出不一定会显示的写到方法签名里的, 也有可能来自 DB、MQ、全局变量<br>
如果真有一个方法没有返回值, 也没写 DB、MQ、全局变量, 那这个方法就是没有意义的<br><br>
当前方法的输入和输出都来自全局变量, 那我可以创建一个包含这个全局变量的对象, 执行被测试方法, 再拿到全局变量去验证

> 使用覆盖度运行(Run With Coverage): command + R

可以看到绿色部分是被100%覆盖的区域, 黄色部分是部分覆盖区域, 红色部分是未被覆盖区域<br>