# 平台能力

D.1 支持的浏览器 D.2 通用特性 D.3 支持的传感器和设备 D.4 对服务/资源的支持 D.5 对所创建应用的支持 关键词 操作系统 通用特性 编程技术 运动传感器 方向传感器

`TouchDevelop` 提供的资源（服务）依赖于特定平台的能力。某些数据类型的某些方法可能仅在特定设备上可用。除非在脚本属性中将目标平台设置得当，否则它们不会在 `TouchDevelop` 编辑器中列出。¹



### D.1 支持的浏览器

`TouchDevelop` 的 WebApp 版本要求使用下表中列出的浏览器版本。

| 平台 | 浏览器 |
| --- | --- |
| PC、Mac、Linux | Internet Explorer 10、Safari 6+、Chrome 22+、Firefox 16+ |
| iPad、iPhone、iPod Touch | iOS 6+ 上的 Mobile Safari |
| Android | Chrome 18+ |

### D.2 通用功能

所有受支持的平台/浏览器组合下的 WebApp 以及 Windows Phone 上的 `TouchDevelop` 应用 v2.11 均提供对 `TouchDevelop` 系统下列通用特性的支持：

-   完整的 `TouchDevelop` 脚本语言
-   执行 `TouchDevelop` 脚本
-   编辑 `TouchDevelop` 脚本
-   离线编辑与脚本执行
-   使用 Microsoft、Facebook 或 Google 账号登录的能力

`TouchDevelop` 脚本的以下部分可用于要在浏览器中执行的脚本，但尚不能用于将在 Windows Phone 上运行的脚本：

-   页面和盒子（请参见第 10 章）
-   使用记录的库（请参见第 2 章）

### D.3 支持的传感器与设备

即便平板电脑或计算机具备 `TouchDevelop` 脚本通常可利用的传感器和设备，操作系统也可能会禁止访问或使浏览器中运行的程序难以访问。下表显示了当前哪些平台可以访问哪些传感器和设备的状态。（注：Windows Phone 支持所有这些传感器和设备。）

| 传感器/设备 | 支持状态 |
| --- | --- |
| 加速度计 | 适用于 iPad、iPhone、iPod Touch。在 PC、Mac、Linux 上通过跟踪鼠标（或手指触摸）移动进行模拟。在 Android 上通过跟踪设备方向模拟；加速度向量具有固定大小。 |
| 摄像头 | 在 PC、Mac、Linux 和 Android 上可通过 Chrome 浏览器使用。 |
| 指南针 | 适用于 iPad、iPhone、iPod Touch 和 Android。 |
| 陀螺仪 | 适用于 iPad、iPhone、iPod。 |
| 麦克风 | 计划适用于 iPad、iPhone、iPod 和 Android。 |
| 运动 | 计划适用于 iPad、iPhone、iPod 和 Android。 |
| 电话 | 如果设备具有通话功能或安装了 Lync 或 Skype，则可使用。 |
| 方向 | 计划适用于 iPad、iPhone、iPod 和 Android。 |

### D.4 服务/资源支持

除一项例外，附录 B 中列出的由 `TouchDevelop` API 提供的所有资源（或服务）均可用于 Windows Phone。这一例外是 `Project Hawaii` 提供的语言翻译服务，该服务暂时不可用。

下表中列出的资源在 PC、Mac、iPad、iPhone、iPod Touch 和 Android 平台上获得的支持有限或完全不支持。

| 资源 | 支持状态 |
| --- | --- |
| social/calendar | 不支持通过 social 资源访问用户的日历 |
| social/contacts | 不支持通过 social 资源访问联系人 |
| media/songs | 不支持通过 media 资源访问歌曲或歌曲专辑 |
| media/pictures | 不支持通过 media 资源访问图片或图片专辑 |

### D.5 对已创建应用的支持

已创建的应用是指已导出到 Windows Phone 商店或 Windows 商店（其中包含适用于运行 Windows 8 操作系统的 Surface 平板电脑和 PC 的应用）的应用。

#### D.5.1 适用于 Windows Phone 商店的应用

脚本不能使用盒子、页面或库。

脚本不能使用 `Project Hawaii` 提供的语言翻译服务。

#### D.5.2 适用于 Windows 商店的应用

脚本不能使用陀螺仪、麦克风、运动传感器或方向传感器，但计划在未来支持这些设备。

![知识共享许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0) 开放获取 本章采用知识共享署名-非商业性使用-禁止演绎 4.0 国际许可协议（[`​creativecommons.​org/​licenses/​by-nc-nd/​4.​0/​`](http://creativecommons.org/licenses/by-nc-nd/4.0/)）授权，该协议允许任何非商业性使用、共享、分发以及在任何媒介或格式中复制，前提是您给予原作者和出处适当的信誉，提供知识共享许可协议的链接，并注明是否修改了许可材料。根据本许可协议，您无权分享改编自本章或其部分内容的材料。除非在材料的署名行中另有说明，本章中的图片或其他第三方材料均包含在本章的知识共享许可协议中。如果材料未包含在本章的知识共享许可协议中，且您的预期使用未得到法定规定许可或超出允许范围，您需要直接获得版权所有者的许可。脚注 1

本附录转载自 TouchDevelop 网站上的内容，网址为 [`https://www.touchdevelop.com/platforms`](https://www.touchdevelop.com/platforms)

