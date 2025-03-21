## 去除平滑滚动 (聊天窗口)

这部分是聊天窗口的, 消息列表的在**下半部分 - [去除平滑滚动 (消息列表)](#去除平滑滚动-消息列表)**

使用 Cheat Engine 打开 QQ 进程, 打开一个聊天界面, **扫描选项选 `未知初始值`**, 进行扫描

![image](https://github.com/user-attachments/assets/608030f5-9175-4fde-b2d8-e8aaa10f04d1)

然后将聊天页面向下滚动, 扫描选项选 `增大值`, 点下一步

![image](https://github.com/user-attachments/assets/8a7af3a0-0afb-4a7c-8d3c-676e01388597)

重复多次, **向上滚动就筛选 `减小值`, 向下滚就筛选 `增大值`**, 直到出现两个看起来很像页面滚动高度的值为止  
然后添加到地址列表

![image](https://github.com/user-attachments/assets/13e36458-15b1-4066-8226-c9f45f45b557)

修改第一个值会发现, 这个值是滚动条的  
修改第二个值没有生效, 再次滚动会被新值覆盖

![image](https://github.com/user-attachments/assets/3ebd3738-532e-4ad5-847a-2e9345a4b767)

我们研究第二个值, 对其**查找写入操作**  
然后滚动聊天框, 找到一处代码 `AFCtrl.dll+9F643 mov [edi+0000027C],eax`  
将其添加到代码列表

![image](https://github.com/user-attachments/assets/2d52052c-83cb-461f-a2fc-85af54c3e43b)

点击 `显示反汇编` 打个断点玩玩看

可以发现, 打断点的情况下, 滚动一次鼠标滚轮, **第一个 tick 滚动了 `11 (0xB)` 像素, 第二个 tick 滚动到了 `90 (0x5A)` 像素**

![image](https://github.com/user-attachments/assets/ec2ba3af-ccd9-4c33-9d9f-91a72d9ef538)

这附近的汇编不好看懂, **我们直接上 IDA**  
IDA 的地址和 CE 有点不一样, 我喜欢把二进制复制出来搜索

![image](https://github.com/user-attachments/assets/7d5d7e31-40df-4796-b6e5-0d5ebc17ad79)
![image](https://github.com/user-attachments/assets/f54c0c39-ab6c-494a-b91e-6f257ec40887)
![image](https://github.com/user-attachments/assets/e8f6768a-392b-4ff0-8ce8-8cb7843b4612)
![image](https://github.com/user-attachments/assets/2e5c4edb-6ea1-4f58-b54f-e57f2784a501)

反编译出来可以发现是把 `a2` 赋值给了内存, 并且发现 `a2` 是该函数的入参

![image](https://github.com/user-attachments/assets/4d8fc00a-aa2c-4fbd-b912-cc0c6bab78bf)

> 提示: 可以按 `Alt+M` 添加地址书签, 方便后续找回这个位置  
> 按 `Ctrl+Shift+M` 查看地址书签
>
> ![image](https://github.com/user-attachments/assets/03d1526c-d623-4a27-9329-f4940b028ce4)
> ![image](https://github.com/user-attachments/assets/61bede59-3c95-486a-a8db-de971df7a5da)

在函数头选择 `查找引用`, 直接点进去

![image](https://github.com/user-attachments/assets/14c23bac-78cc-4c69-bea9-baa9ad0e5e50)
![image](https://github.com/user-attachments/assets/255edd38-51ea-4971-9871-bec1cb544a10)

发现`v3 = *a2`, `*a2` 是入参, 继续往上找

![image](https://github.com/user-attachments/assets/1c7cdc02-c2fe-423a-901f-c03a484bb261)

但是发现找不到直接调用的方法了

![image](https://github.com/user-attachments/assets/9b4fb3c0-afc3-43ce-baed-6878e0d27290)

我们直接开启调试器**打断点**

选择 `附加到进程`, 选择 QQ 进程

![image](https://github.com/user-attachments/assets/9188ba79-c7e3-4023-b278-aae44b66b41c)
![image](https://github.com/user-attachments/assets/aec01535-3dcd-4153-9ffe-e64625aa0b67)

等待一段时间加载完 symbols, 我们用地址簿找回刚刚的代码, 打断点  
再次滚动聊天框, 触发断点后, 就能看**调用栈**了

![image](https://github.com/user-attachments/assets/8062a90d-baf7-4595-a7c4-87d42deaddab)

往上找, 发现这里是靠指针调用的函数, 所以没法静态找出引用  
这里 `a2` 又是入参, 继续往上找

![image](https://github.com/user-attachments/assets/8bffd82b-3a60-4fa8-b68d-73f3ed75e178)

终于发现 `v4` 似乎是在这个函数通过一系列计算得到的

![image](https://github.com/user-attachments/assets/2ec7d764-3714-4825-8a5b-f63919d2bd8e)

这里看 `v4` 似乎是一堆复杂计算得到的  
我反复打断点研究了好多次, 发现这个似乎是栈里面的 C++ 类, 里面有各种方法通过指针调用, 实在太烧脑了看不下去

打断点也看不出来这附近的变量跟最终滚动位置有关的  
要么就是差几十个像素, 要么就是不知道为什么调用了一个函数就把值改写了

![image](https://github.com/user-attachments/assets/b54864ff-d623-47ee-a5d2-adf84898395c)
![image](https://github.com/user-attachments/assets/e524f91c-0b26-4f01-85fa-a7522af3baaf)

到这里重新思考一下破解思路

我们要做的是去掉平滑滚动, 也就是让它**一下子就滚动到最终位置**  
那么应该有一个变量, **存放了最终滚动位置**

但是这个类是在栈里面的, 每次调用都变地址, 搜索起来极为麻烦

就在我对这一堆满天飞的指针头晕的时候, 我点开了调用栈的上一个函数...

![image](https://github.com/user-attachments/assets/a3fe5443-4f28-4007-b32c-303e4c0e1d5b)

再往上...  
惊人的来了, 这个入参的 **`v14 (0xB)` 不就是当前 tick 滚动的像素数吗!**

![image](https://github.com/user-attachments/assets/361905cf-24fb-4011-89c0-7097a64227e6)

经过多次打断点得出, **`v11 (0x5A)` 是滚动的总像素数**  
下面的判断语句只有在滚动的**第一个 tick 会执行**, 用来计算第一个 tick 滚动的距离  
`v12` 是上一次滚动的值  
`v13` `v14` 分别是正负滚动像素数, 下面的判断语句用来判断是向上还是向下滚动

![image](https://github.com/user-attachments/assets/1578932a-125b-46ff-b98d-ae7fcdfe421b)

我们把那个**第一次才触发的语句强制跳过, 让它第一次就把所有像素滚完**

![image](https://github.com/user-attachments/assets/23b6ea8e-0642-4bac-ae93-ebec93aea1dc)

测试结果是非常成功的, 但是有个问题:  
这个每次滚动的最终位置带有加速, 如果你鼠标滚动滚得很快, 那么聊天记录会飞得越来越快看不清

只要修改一下上面那句**获取总滚动像素的语句, 改成常量就可以了**

右键 `mov eax, [esi+0F4h]`, 点汇编, 把 `[esi+0F4h]` 改成常量 (比如 `100`)  
按回车应用修改

![image](https://github.com/user-attachments/assets/e4831dac-aaff-481c-8cde-d953726edb07)
![image](https://github.com/user-attachments/assets/2866e08a-2f47-475c-b340-eaad60462d91)

现在每次鼠标滚动都是常量了

下面就能点 `编辑` - `修补程序` - `将补丁应用到...`, 输出修改后的文件了  
直接放到 QQ 文件夹替换掉就能修改成功

![image](https://github.com/user-attachments/assets/b5e02b3c-488b-4836-bbf1-a3348201559c)

**大功告成!**

> 另外, 按 `Ctrl+Alt+P` 可以查看已修改的字节
>
> ![image](https://github.com/user-attachments/assets/76d2b202-16f2-4f53-ae38-eaea530fbc46)


## 去除平滑滚动 (消息列表)

用上面一样的方法**找滚动高度值**, 找出来之后**查找写入**, 给 `GF.dll+105DFB - sub [esi+00000200],ecx` **打断点**

![image](https://github.com/user-attachments/assets/cc81039d-f1e7-4353-b8f0-daf3a965a74c)

易发现**每个滚动 tick 都会触发一次断点**, 而且**每个 tick 滚动的像素数在寄存器 `ecx`**  
(这里因为是对内存中的值进行减法, 所以往下滚动是减去负数, `FFFF FFE5` 转换为 uint32 是 `-27`)

![image](https://github.com/user-attachments/assets/6775bd37-6d72-49b9-af81-662c74ca4a5b)

这里不做汇编分析了, 直接上 IDA

发现是把 `v16` 赋给内存, `v16` 从 `a3` 来, `a3` 是入参

![image](https://github.com/user-attachments/assets/91885ece-2df0-4ca5-9a5c-faa904a9297a)
![image](https://github.com/user-attachments/assets/ebb09ee5-0ec2-4b74-abaa-2ce11c65ec06)

但是调用这个函数的地方太多了, 我们还是打断点

![image](https://github.com/user-attachments/assets/3298150c-b2c8-4d6c-92e9-3fb0241eee20)

往上一翻就很明显了, 我们直接研究这个 `v4`

![image](https://github.com/user-attachments/assets/7e53db2c-d538-4472-8047-8ca9605df405)

容易发现 `v4` 前面对 `*(_DWORD *)(this + 596)` 有一个**减法运算**  
多打几次断点可以看出, **`v4` 是当前 tick 滚动的距离, `this + 596` 存放剩余要滚动的像素数**

那么我们的思路就是让滚动一步到位, 直接把**剩余要滚动的总像素数, 作为传参**

![image](https://github.com/user-attachments/assets/2f1e6026-2116-487b-a922-7ff68cb36200)

把 `cvttsd2si eax, qword ptr [esi+258h]` 改为 `mov eax, [esi+254h]`  
立即生效了

> 这里插播一个知识:  
> cvttsd2si 是 `double` 转 `int`, `qword ptr` 是 64 位指针, 跟 `[esi+258h]` 搭配使用  
> 这里不能只修改地址, 否则类型就出错了, 一开始没留意发现修改之后无法滚动

![~)HGF2L8X5U)5ZB2TQ`(TRM](https://github.com/user-attachments/assets/d761f868-983c-4cab-998f-1d387d5e7c36)

如果你不追求细节, 那么修改到这里其实就可以了

但是修改完之后你再打断点会发现, 虽然滚动一步到位了但还是**重复滚动了好几个 tick, 只不过每次的值都一样**  
要做完美, 就要把**多余的滚动 tick 去掉**

如果你从头开始打断点就会发现, 当滚动到最后一 tick 时, 这个 `if ( !*(_DWORD *)(this + 592) )` 会进入  
可以判断这是个**计数器, 指定要滚动多少个 tick 的**

![image](https://github.com/user-attachments/assets/15029e50-c5bc-4e43-8915-9e082a813ee0)

往下翻有一个地方是把 `this + 592` 清零的, 我们试试让它强制执行, 即**第一个 tick 滚动完就将计数器清零**

![image](https://github.com/user-attachments/assets/c180c0c7-cbda-45a4-90fc-2394f9cca664)

调整后打断点实测这个函数**只会触发一次**了

修补完这两个地方之后, 主面板的平滑滚动也没了

**至此完成!**
