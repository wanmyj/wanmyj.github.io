---
layout: post
tags: [config]
categories: [C#]
---
# -----

这个方法参考了博客园的[这篇文章](https://www.cnblogs.com/superhin/p/12600358.html)，它使用了如下的python代码实现

```python
from selenium import webdriver

driver = webdriver.Chrome()
driver.get('https://www.qq.com/')

# 1. 执行 Chome 开发工具命令，得到mhtml内容
res = driver.execute_cdp_cmd('Page.captureSnapshot', {})

# 2. 写入文件
with open('qq.mhtml', 'w', newline='') as f:   # 根据5楼的评论，添加newline=''
    f.write(res['data'])

driver.quit()
```

使用c#代码实现同样的功能
```csharp
using System;
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using OpenQA.Selenium.DevTools;
using OpenQA.Selenium.DevTools.V115.Page;

class Program
{
    static void Main(string[] args)
    {
        using IWebDriver driver = new ChromeDriver();
        driver.Navigate().GoToUrl("https://www.qq.com/");

        IDevTools devTools = driver as IDevTools;
        DevToolsSession devToolsSession = devTools.GetDevToolsSession();
        var res = await devToolsSession.SendCommand<CaptureSnapshotCommandSettings, CaptureSnapshotCommandResponse>(new CaptureSnapshotCommandSettings());

        // 写入文件 res类型是CaptureSnapshotCommandResponse
        string mhtmlContent = res.Data;
        System.IO.File.WriteAllText("qq.mhtml", mhtmlContent);
    }
}
```

