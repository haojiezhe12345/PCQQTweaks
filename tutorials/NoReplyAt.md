## 回复不自动添加@

使用 Cheat Engine 打开QQ进程, 搜索聊天框输入的字符串, 注意勾选 UTF-16  
(为了避免网络IO造成打断点时卡死, 建议先把QQ调成离线模式)

![image](https://github.com/user-attachments/assets/4b807e7d-0fa4-4438-bf8d-03384c241794)

改变字符串, 继续搜索, 直到找到两个地址为止

![image](https://github.com/user-attachments/assets/2381a4c0-1647-41a6-a0c0-1ac73fb66b1e)

尝试在CE修改字符串, 其中一个可以成功回显

![image](https://github.com/user-attachments/assets/91fbb063-02e7-4303-ad22-be5b87717df0)

删除聊天框内所有文字, 然后右键那个地址, **查找写入操作**

![image](https://github.com/user-attachments/assets/db901d00-7b34-4730-9f09-af3b2247c685)

这个时候右键回复一条消息, 就会出现访问那个地址的操作

![image](https://github.com/user-attachments/assets/a21b2780-9799-4895-baa4-b3f42f928856)

选择 `39702610 - 88 07  - mov [edi],al` 添加到代码列表, 以便后续定位

![image](https://github.com/user-attachments/assets/1c974187-8061-43dd-b2ec-ffd742baefa9)

为了保证不出现多余操作, 先清空QQ聊天框  
然后在反汇编中**给 `mov [edi],al` 打断点**

![image](https://github.com/user-attachments/assets/f3ce1bf2-45f5-4a8b-b0c7-c3f80e5f6615)

再次回复消息之后, 触发断点

![image](https://github.com/user-attachments/assets/e2d9c97e-73d7-429a-a302-09f31c909101)
![image](https://github.com/user-attachments/assets/9ffaf2af-7a50-44a7-a1f2-140cbadf8988)


接下来一直点Run, **直到调用栈里面看到 `AppFramework.dll+2A98B` 为止**  
然后双击跳转到那个函数地址, 往上翻一行, 把 `call edi` 加入代码列表

(经验之谈, 我也是试了很多次才发现这个函数打断点方便)

![image](https://github.com/user-attachments/assets/99d16242-dafa-4e57-986a-2b420fcc8d6f)

回到 `riched20.dll+2610`, **取消断点**

![image](https://github.com/user-attachments/assets/0ab13d1d-8f83-448b-8064-1d0e417334d2)

**给 `AppFramework.dll+2A98B` 打断点**, 然后**回复自己**  
(这里利用的原理是**回复自己的时候是不带@的**)

![image](https://github.com/user-attachments/assets/4b83e35e-ff3e-4a48-b09c-ec502b022f9f)

**回复自己**的时候会触发**两次** (也有可能是三次) 断点

可以看到这里已经出现一个很可疑的函数了, `AppUtil.MsgReplyHelper::InsertReplyOleCtrl+20E`  
但我们并不对它研究 (这个估计是UI的handler, 我研究过了没什么用)

![image](https://github.com/user-attachments/assets/79ae5456-70b0-4bfe-bf69-df08cf36cd29)

我们记录下这两次断点的调用栈

![image](https://github.com/user-attachments/assets/77a82a2b-302a-4a58-a138-d6a2f9282562)
![image](https://github.com/user-attachments/assets/2fbab05c-c48b-4f64-bc2d-b9f30f0bc93c)

然后删除聊天框文本  
(注意, 此时你打过断点, 那么删除文本也会触发断点, 也需要点一次Run)

然后**回复别人**的消息

![image](https://github.com/user-attachments/assets/4752f04e-4364-4613-a997-9bf95b40b305)

**回复别人**会触发**五次**断点, 我们也是把调用栈都记下来  
(第二次和第三次调用栈完全一样, 这里忽略)

![image](https://github.com/user-attachments/assets/5436c46a-d837-4394-8659-9754cda7e60a)
![image](https://github.com/user-attachments/assets/ab9e13f6-2617-41a4-ac95-4222db481e98)
![image](https://github.com/user-attachments/assets/dc44e111-ebb4-4ba1-bc42-31bec5977841)
![image](https://github.com/user-attachments/assets/841aa19b-867b-4912-bfa3-3f149adae376)

对比发现, 回复自己和别人, **前两次调用栈完全一致**

但是回复别人触发的**第4次断点**, 调用栈有点不一样  
(左边是第1次, 右边是第4次)

![image](https://github.com/user-attachments/assets/678944e3-0759-4d16-9824-2419bc928c68)

这里已经可以判断**第4次触发断点的操作就是添加回复@**

**而第4次断点中, 没有走 `MsgMgr.dll+52C47`, 而是走了 `MsgMgr.dll+52D0E`**

那么我们的思路就是**让 `MsgMgr.dll+52D0E` 不触发就行了**

我们跳转到 `MsgMgr.dll+52D0E`, 发现前面有一个判断语句, 符合条件则触发 `call dword ptr [edx+10]` 函数调用

![image](https://github.com/user-attachments/assets/479b7d69-0446-4a52-8f37-049767b18886)

想要 `call dword ptr [edx+10]` 不触发, 只需要改一下前面的判断语句

把 `je MsgMgr.dll+52D0E` 改为 `jmp MsgMgr.dll+52D0E`, 无论什么条件都跳过执行下面那块

![image](https://github.com/user-attachments/assets/916fcf88-a4a2-4176-a648-5ba1620b1251)

至此, 破解成功

![image](https://github.com/user-attachments/assets/1219379a-2707-403d-b0be-862a88b2e00d)

接下来**手动修补DLL**

请注意先备份原DLL

把修补前和修补后的字节码记下来

修补前, `85 C0 74 16 FF 75 F4`  
修补后, `85 C0 EB 16 FF 75 F4`

![image](https://github.com/user-attachments/assets/ec11181c-1a8a-4a10-81c9-412a8a3d0bdd)

然后用十六进制编辑器打开 `MsgMgr.dll` (这里使用WinHex)

查找字节 (注意去掉空格), 搜索方向为全部

![image](https://github.com/user-attachments/assets/b2c5f093-2eb4-4837-85d2-5428eb4d5a2c)

找到一条匹配结果, 把 `74` 改为 `EB`, 保存即可

然后关闭QQ, 替换掉 `QQ.exe` 同目录下的 `MsgMgr.dll` 即可
