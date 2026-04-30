# SSH-vscode-codex
SSH反向端口转发实现远程服务器上vscode+codex官方插件超详细教程

<br>

发这个被CSDN制裁了，于是转而来到GitHub，，

### 个人情况：实验室服务器没有科学上网，平时是在vscode里直接用ssh远程连接使用的，查询了很多教程基本上思路都是ssh反向端口转发，也就是让服务器的请求走本地的网，但是跟着实操都莫名其妙不行，最终在本地codex的指导下实现了，整理出来供大家参考

<br>

## 第一步：确定本地代理端口
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e73501e77a4847d696c09be7a3d68315.png)
这里我用的软件是clash verge，端口号就是在setting里面的这一个，不同软件可能界面和端口不一样，自己去设置里看一下就好。

验证这个端口有没有问题，可以在win+r，输入cmd的那个终端页面里输入下面的内容：
（我这里用的我自己的端口7897，记得根据自己情况修改端口号）


```bash
curl.exe -I --proxy http://127.0.0.1:7897 https://api.openai.com
```
返回下面开头是HTTP的内容说明没有问题：

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/eb1868329db949ea963303921c56d0a6.png)



<br>

## 第二步：本机的 SSH 配置里加反向转发
先确保当前vscode打开的是本地的页面，然后ctrl+shift+p，选择Open SSH Configuration File这一条

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9f436002fcb640e282741901cd47fa83.png)

然后会再让你选择一个config文件，如果这里有多个的话一般就是选`C:\Users\你的用户名\.ssh\config`这一个

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d41e194ff28f478bbeee47e4c2ddd05b.png)

在打开的文件中插入下面的内容（前面几行应该本身就有，主要是最后两行）：


```bash
Host myserver
  HostName 你的服务器IP
  User 你的用户名
  RemoteForward 17890 127.0.0.1:7890
  ServerAliveInterval 60
```

解释：这里的myserver是你自己给这台服务器起的一个SSH别名，可以随便取无所谓的，但是需要注意连接服务器时，要用这个别名连，这样RemoteForward才会自动生效。HostName和User是连服务器必须的东西，给你服务器的人肯定会告诉你，这里不再赘述。

重点是后面两句，含义就是服务器上的 127.0.0.1:17890 会通过SSH隧道回到本机的 Clash 127.0.0.1:7890。端口号17890是随便选的一个远端监听端口， 只是为了避免端口冲突挑了个不常用的号，也可以换成其他的，只要不冲突就行。

<br>

## 第三步：服务器上配置协议

在服务器终端中执行下面的指令：

```bash
nano ~/.profile
```
进入该文件的编辑状态。
在该文件的最后加上下面的内容：

```bash
export HTTP_PROXY=http://127.0.0.1:17890
export HTTPS_PROXY=http://127.0.0.1:17890
export ALL_PROXY=http://127.0.0.1:17890
export http_proxy=$HTTP_PROXY
export https_proxy=$HTTPS_PROXY
export all_proxy=$ALL_PROXY
export NO_PROXY=localhost,127.0.0.1
export no_proxy=$NO_PROXY
```
然后保存退出：Ctrl + O 再点回车保存，Ctrl + X 退出。
然后让它立刻生效：

```bash
source ~/.profile
```
验证前面的配置是否生效，在终端中执行下面的指令：

```bash
env | grep -i proxy
```
出现下面这样的输出就说明没有问题。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/31d9eed8cbc2424d9cc93d06d69870a3.png)

接下来还是在已经连上服务器的那个VS Code远端窗口里操作。按 Ctrl + Shift + P，在上方的输入框里粘贴`Preferences: Open Remote Settings (JSON)`

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ca779dddf33a41a685f5e3856c229e14.png)

选择第一条进入文件，在该文件中插入下面的内容：

```bash
"http.proxy": "http://127.0.0.1:17890",
  "http.proxySupport": "override",
  "http.noProxy": [
    "localhost",
    "127.0.0.1"
  ]
```

可能这个文件里面原本会有其他的东西，注意缩进直接加在后面应该就行了，位置无所谓。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3a53ecdeb67d4705ad2a1e96b4afe442.png)
<br>
## 第四步：最后的验证与使用
先在VS Code里关闭远程连接再重新连一下，然后在VS Code远端窗口的终端里输入下面的指令：

```bash
curl -I --proxy http://127.0.0.1:17890 https://api.openai.com
```

返回下面开头是HTTP的内容说明没有问题：

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7262eec105df44ed9edbdc171fb99d18.png)

确定Codex官方插件安装在远程那一端，然后就可以登录自己的账号正常使用了！

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b733c292796d45a9b2dcf50ddfd0154f.png)


