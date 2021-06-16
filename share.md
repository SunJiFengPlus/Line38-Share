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
接下来的任务就添加参数, 让这些参数尽可能覆盖所有代码<br>
如果无论如何调整参数都不能100%覆盖, 那恭喜你, 你发现了Dead Code, 放心的删除它吧<br>

使用命令来认定当前的KV文件是正确的
``` shell
mv /Users/sftc/project/my-github/line83-4/4th_round_8f0d5e29ea48612ea1be0ab4537dc51f66d6610d/./src/test/java/store/StoreTest.test.received.txt /Users/sftc/project/my-github/line83-4/4th_round_8f0d5e29ea48612ea1be0ab4537dc51f66d6610d/./src/test/java/store/StoreTest.test.approved.txt
```
如果后面代码的改动导致形成的KV文件与刚刚确认文件有所不同, 那当前代码的改动**也许**就是错误的

### 小步重构
看到 IDEA 提示代码中有一些警告
> 展示当前上下文操作(Show Context Action): Command + 1

可以看到 IDEA 给出几个方案解决这个警告, 无脑的选择吧
然后运行一下刚刚执行过的测试用例
> 运行当前上下文(Run Context): Control + R

看到这次修改并没有影响代码运行的效果, OK, 安全的重构<br><br>

对于当前的题目, 我的思路是以```Item```作为父类, 父类有```value```与```sellIn```两个属性, 
有一个```updateValue```方法, ```Sulfuras```、```Aged Wine```、```Show Ticket``` 作为子类,
重写```updateValue```方法.<br>
我首先要做的就是把各个类型的商品逻辑拆开, 让代码变成下面这样
``` java
if (name.equalsTo("Sulfuras")) {
    // do something
} else if (name.equalsTo("Aged Wine")) {
    // do something
} else if (name.equalsTo("Show Ticket")) {
    // do something
} else {
    // do something
}
```

按照上面的思路来, 要把代码搞成 if、else if、else 的结构
> 提取方法(Extract Method): Option + Command + M

把核心逻辑提取到一个方法中, 随便命名一下, 再把代码格式调整到预期的格式<br>
运行测试用例, 通过, 不错, 没有影响代码的运行效果, 虽然代码变的更烂了😂<br>
然后我们再把代码内联回来
> 内联(Inline Method): Option + Command + N

运行测试用例, 测试通过<br>
看到代码中多了好多警告, 利用 IDEA 把它解决掉吧<br>
运行测试用例, 测试通过<br>
- 创建```AgedWine```子类, 并重写相关方法, 运行测试用例, 测试通过<br>
- 替换```Store```中```AgedWine```的逻辑, 运行测试用例, 测试通过<br>
- 创建```ShowTicket```子类, 并重写相关方法, 运行测试用例, 测试通过<br>
- 替换```Store```中```ShowTicket```的逻辑, 运行测试用例, 测试通过<br>
- 创建```Sulfuras```子类, 并重写相关方法, 运行测试用例, 测试通过<br>
- 替换```Store```中```Sulfuras```的逻辑, 运行测试用例, 测试通过<br>
- 补充普通商品的逻辑, 运行测试用例, 测试通过<br>
- 替换普通商品的逻辑, 运行测试用例, 测试通过<br>

每个步骤结束之后都运行一下测试用例, 每当看到测试用例通过时能使你信心满满的去进行下一步重构<br>
小步重构的对立面就是刀砍斧劈的重构, 大改一通, 运行测试用例失败了, 距离上一次测试用例通过已经过去2个小时了, 这期间改的东西太多了, 我也不知道哪一步出错了, 那就只能慢慢排查了<br>

### 找出并修复Bug
现在各个商品的逻辑已经全部下沉到子类了, 按照文档和每个子类比对即可<br>

### 加入特效药

### 总结
如果今天的内容你只能记住一件事, 那请记住: **做有安全保障的重构**. 安全保障可以是自动的也可以是手动的