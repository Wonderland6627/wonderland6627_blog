---
title: SVN 利用Post-commit hook 同步提交信息至钉钉群
date: 2022-06-06 14:48:40
tags:
    - SVN
categories:
    - 技术
    - 开发流程
top_img: https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/07393c4437cc5e0cb7589e9194807fedd2cd0a3c.jpg
---

***
### 参考文章：
SVN提交后自动推送消息到钉钉群：[https://www.likecs.com/show-305859871.html](https://www.likecs.com/show-305859871.html)
钉钉开放平台第三方自定义机器人：[https://open.dingtalk.com/document/group/custom-robot-access](https://open.dingtalk.com/document/group/custom-robot-access)
***

### 钉钉群添加机器人：

- 1.在你要同步通知的钉钉群创建一个机器人

![机器人管理](https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/19142f921df748b19cbe567078025bab.png)
- 2.三种安全设置至少需要选择一种，这里选择“自定义关键词”，这样这个机器人只会推送带有指定关键词的消息
![安全设置](https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/3115da50a09b4bf4a9e289b91ac79ebb.png)
- 3.获取Webhook，主要是需要这个，揣兜里，不要给别人看
![Webhook](https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/776b4aae80eb4d629a65c46af6eecf9d.png)
### 写一个WebRequest发送Post请求：
看[钉钉官方文档](https://open.dingtalk.com/document/group/custom-robot-access)，只要给指定链接配置正确参数发出请求，就能够在钉钉群里发消息了，这也是为啥要让把链接保密，不然别人可以随意的通过链接让你的机器人在群里发消息了。
网络请求的方式有很多种，这里用C#作为示例：
```csharp
	class MainClass
    {
        public static void Main(string[] args)
        {
            if (args == null || args.Length == 0)
            {
                return;
            }

            string content = args[0];
            content = "SVN Message Sync:" + content; //注意content中要有机器人设置所需要的keyword

            string json = "{'msgtype': 'text','text': {'content':'" + content + "'}}";

            string url = "https://oapi.dingtalk.com/robot/send?access_token=";
            string token = "这里填写机器人给你的token，就是之前揣兜里的";
            url += token;

            HttpPost(url, json);
        }

        public static string HttpPost(string url, string param)
        {
            HttpWebRequest request = WebRequest.Create(url) as HttpWebRequest;
            request.Method = "POST";
            request.ContentType = "application/json;charset=UTF-8";
            byte[] payload = Encoding.UTF8.GetBytes(param);
            request.ContentLength = payload.Length;

            Stream writer = request.GetRequestStream();
            writer.Write(payload, 0, payload.Length);
            writer.Close();

            HttpWebResponse response = request.GetResponse() as HttpWebResponse;
            Stream stream = response.GetResponseStream();
            StreamReader reader = new StreamReader(stream, Encoding.UTF8);
            string value = reader.ReadToEnd();

            reader.Close();
            stream.Close();
            response.Close();

            return value;
        }
    }
```
将这个方法生成一个.exe，待会云服要用这个.exe
其实这个时候你已经可以通过传入正确的参数使你的机器人推送消息了，不过我们的目的是让SVN的Post-commit hook来帮我们操作，所以继续

把这个.exe放到云服上VisualSVN的安装目录下
以我们的为例，改成你自己的：

	C:\Program Files\VisualSVN Server\bin

为什么要放到这里呢？因为如果放到Desktop等位置会有访问权限的问题，当然如果你的机器没有这个问题就请忽略～

### 修改SVN Post-commit hook
![SVN Hook](https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/5632f7c3e1474a04b200e6475af95b46.png)
打开你的VisualSVN Server，打开仓库属性，双击"Post-commit hook"，把这段填进去，别问我这是啥，我也不知道，网上找的：[钉钉通知系列]SVN提交后自动推送消息到钉钉群：[https://www.likecs.com/show-305859871.html](https://www.likecs.com/show-305859871.html)

```bash
set REPOS=%1
set REV=%2
set tttt=%date:~0,10% %time:~0,8%

for /f "tokens=1,2 delims=:" %%a in ('svnlook author -r %REV% %REPOS%') do (
    if not defined AUTHOR set AUTHOR=%%a
)

for /f "tokens=1,2 delims=:" %%a in ('svnlook log -r %REV% %REPOS%') do (
    if not defined MESSAGE set MESSAGE=%%a
)

set CONTENT="\n提交时间：%tttt%\n提交版本：%REV%\n提交作者：%AUTHOR%\n提交备注：%MESSAGE%"

"C:\Program Files\VisualSVN Server\bin\CSWebRequest.exe" %CONTENT%
```

***
至此，再进行SVN提交的时候，就会在钉钉群发出消息提示了

### 可能遇到的问题
如果没有像预期那样，钉钉机器人不帮你发消息，你可以试试把Response打印出来，看看钉钉给你返回了什么错误信息，列出集中遇到过的问题，可以对号入座进行修改，但不一定全面：
1. content中没有钉钉机器人设定的关键词，这一点在注释中也提到了
2. message格式不对，或者符号不匹配，需要传入的是json
3. 访问权限问题，改.exe放置的路径
4. 传的参数带空格，被当成了多个参数
5. 。。。未完待续，欢迎留言讨论
***