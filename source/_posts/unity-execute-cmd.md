---
title: 【Unity】 执行CMD命令行并返回输出数据
date: 2021-11-21 18:41:07
tags: 
    - Unity
    - CMD
categories:
    - 技术
    - Unity
top_img: https://cdn.jsdelivr.net/gh/Wonderland6627/cloudres/blog/07393c4437cc5e0cb7589e9194807fedd2cd0a3c.jpg
---

直接上代码：
```csharp
public delegate void OnDataReceived(string message);
public static OnDataReceived OnDataReceivedHandler;

public static void Execute(string command)
{
		command = "/c chcp 437&&" + command.Trim().TrimEnd('&') + "&exit";

        Process process = new Process();
        process.StartInfo.FileName = "cmd.exe";
        process.StartInfo.Arguments = command;
        process.StartInfo.RedirectStandardOutput = true;
        process.StartInfo.RedirectStandardError = true;
        process.StartInfo.UseShellExecute = false;
        process.StartInfo.CreateNoWindow = true;

        process.OutputDataReceived += (sender, e) =>
        {
            OnDataReceivedHandler?.Invoke(e.Data);
        };

        process.ErrorDataReceived += (sender, e) =>
        {
            OnDataReceivedHandler?.Invoke(e.Data);
        };

        process.Start();

        process.BeginOutputReadLine();
        process.BeginErrorReadLine();

        process.WaitForExit();
        process.Close();
}
```
此方法利用C#的Process，将CMD启动并传入执行参数。

解释一下其中一些处理：
 1. "chcp 437"命令用于将命令行窗口中使用的代码页改为西文字符集，这也是我发现使用Unity调用cmd的一个坑，Unity中调用cmd输出的中文字符会乱码，即使是修改为UTF-8格式也不能解决，但是如果脱离Unity使用却可以正常显示，不知道是什么原因，所以暂使只能是将cmd修改为英文模式，希望有大佬发现解决方法之后指点俺一下~
 2. command中的几个符号：
 ① "/c ":
 "/k command " 表示 执行command命令后保留窗口 可以使用exit关闭窗口
 "/c command " 表示 执行command命令后不保留窗口 即可以省略exit
 ② "&& ":
 cmd指令连接符：
"& "	表示 无条件执行&后的命令	cmd1 & cmd2 即cmd1，cmd2都会执行
"&& " 表示 成功后执行 cmd1 && cmd2 即若cmd1执行成功则执行cmd2
"|| "	表示 失败后执行 cmd1 || cmd2 即若cmd1执行失败则执行cmd2

由于Unity的特性，此方法最好是另开一个线程来执行，并对命令队列进行一些处理，之后继续更新这一部分内容。