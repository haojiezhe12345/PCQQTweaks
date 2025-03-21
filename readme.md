## 所有修补仅适用于PC旧版QQ, 不适用于QQNT!
没有列出参考资料的, 都是本人自行破解, 有空会更新教程


### 显示99+消息具体数量
`ChatFrameApp.dll` (9.7.23.29394)

查　找: `83 F8 63 76 66`   
替换为: `83 F8 63 EB 66`

局限性: 仅聊天窗口左侧可以显示, 主面板和任务栏预览暂未实现


### 防撤回
`IM.dll` (9.7.23.29394)

查　找: `FF 83 C4 28 85 C0 75 34 8D 45 0C C7 45 0C D8 B9`  
替换为: `FF 83 C4 28 85 C0 90 90 8D 45 0C C7 45 0C D8 B9`

查　找: `56 56 FF 50 78 85 C0 79 39 8D 45 0C C7 45 0C E8`   
替换为: `56 56 FF 50 78 85 C0 90 90 8D 45 0C C7 45 0C E8`

局限性: 别人撤回消息时无法感知

参考资料:  
https://github.com/huiyadanli/RevokeMsgPatcher/wiki/QQ%E6%88%96TIM%E9%98%B2%E6%92%A4%E5%9B%9E%E6%95%99%E7%A8%8B  
https://www.52pojie.cn/thread-908763-1-1.html  
https://www.52pojie.cn/thread-1008764-1-1.html  


### 回复不自动添加@
`MsgMgr.dll` (9.7.23.29394)

查　找: `85 C0 74 16 FF 75 F4`  
替换为: `85 C0 EB 16 FF 75 F4`

教程: [NoReplyAt.md](tutorials/NoReplyAt.md)


### 去除消息列表平滑滚动
`GF.dll` (9.7.21.903)

查　找: `F2 0F 2C 86 58 02 00 00`  
替换为: `8B 86 54 02 00 00 90 90`

查　找: `66 0F 2F D1 72 16`  
替换为: `66 0F 2F D1 90 90`


### 去除聊天窗口消息平滑滚动
注: 原版滚动存在加速, 这里修改为每次滚动100像素

`AFCtrl.dll` (9.7.23.29394)

```
查　找: 8B 86 F4 00 00 00 3B F9 73 43
替换为: B8 64 00 00 00 90 3B F9 EB 43
           ↑
           修改此 64 (十六进制) 为你想要的每次滚动的像素值
```

教程: [NoSmoothScroll.md](tutorials/NoSmoothScroll.md)
