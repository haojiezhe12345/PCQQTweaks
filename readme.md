## 所有修补仅适用于PC旧版QQ, 不适用于QQNT!
没有列出参考资料的, 都是本人自行破解, 有空会更新教程


### 显示99+消息具体数量
`ChatFrameApp.dll`

查　找: `83 F8 63 76 66`   
替换为: `83 F8 63 EB 66`

局限性: 仅聊天窗口左侧可以显示, 主面板和任务栏预览暂未实现

### 防撤回
`IM.dll`

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
`MsgMgr.dll`

查　找: `85 C0 74 16 FF 75 F4`  
替换为: `85 C0 EB 16 FF 75 F4`
