# TouchDevelop 服务

B.1 集市 B.2 盒子 B.3 集合 B.4 颜色 B.5 合同 B.6 无效 B.7 语言 B.8 位置 B.9 地图 B.10 数学 B.11 媒体 B.12 电话 B.13 播放器 B.14 传感器 B.15 社交 B.16 标签 B.17 磁贴 B.18 时间 B.19 墙 B.20 网络

本附录转载了 TouchDevelop 网站上 `https://www.touchdevelop.com/docs/api` 的内容。此处提供此内容是为了使本书更加自成体系。附录 B 涵盖了 API 提供的对象（称为资源或服务）。数据类型将在附录 C 中介绍。

### B.1 集市

从集市浏览和审查脚本。

| `ast of(id : String) : Json Object` | 返回指定脚本的抽象语法树 JSON 对象 |
| `leaderboard score : Number` | 获取当前脚本的当前分数 |
| `post leaderboard score(score : Number)` | 将当前游戏分数发布到脚本排行榜 |
| `post leaderboard to wall` | 将当前游戏排行榜发布到墙上 |
| `script id(which : String) : String` | 返回顶级脚本或当前库的标识符 |



## B.2 `box`

访问页面上的当前 `box` 元素。

| `edit(style : String, value : String, changehandler : Text Action)` | 显示可编辑文本，并使用给定的绑定 |
| `on tapped(handler : Action)` | 设置点击盒子时发生的行为 |
| `page height : Number` | 获取页面的总高度 |
| `page width : Number` | 获取页面的总宽度 |
| `pixels per em : Number` | 获取一个 em 对应的像素数 |
| `set background( color : Color)` | 设置背景颜色 |
| `set border( color : Color, width : Number)` | 设置边框的颜色和宽度 |
| `set border widths( top : Number, right : Number, bottom : Number, left : Number)` | 设置各边框的宽度 |
| `set font size(font size : Number)` | 设置此盒子中的字体大小 |
| `set foreground(color : Color)` | 设置元素的前景色 |
| `set height(height : Number)` | 设置此盒子的高度 |
| `set height range(min width : Number, max width : Number)` | 设置此盒子宽度的下限和上限 |
| `set horizontal align(arrange : String)` | 指定如何排列此盒子的内容；`arrange` 为 `"left"`、`"center"`、`"right"` 或 `"justify"` |
| `set horizontal stretch(elasticity : Number)` | 指定如何计算盒子宽度（0 = 收缩以适应内容，1 = 拉伸以适应框架，0.5 = 拉伸至一半宽度） |
| `set margins(top : Number, right : Number, bottom : Number, left : Number)` | 设置此盒子的外边距（在此盒子外部周围留出空间） |
| `set padding(top : Number, right : Number, bottom : Number, left : Number)` | 设置此盒子的内边距（在此盒子内容周围留出空间） |
| `set scrolling(horizontal scrolling : Boolean, vertical scrolling : Boolean)` | 指定当盒子内容溢出时是否使用滚动条 |
| `set text wrapping(wrap : Boolean, minimumwidth : Number)` | 设置是否断开长行，并指定过短而不应断行的长度 |
| `set vertical align(arrange : String)` | 指定如何排列此盒子的内容；`arrange` 为 `"top"`、`"bottom"`、`"center"` 或 `"baseline"` |
| `set vertical stretch(elasticity : Number)` | 指定如何计算盒子高度（0 = 收缩以适应内容，1 = 拉伸以适应框架，0.5 = 拉伸至一半高度） |
| `set width(width : Number)` | 设置此盒子的宽度 |
| `set width range(min width : Number, max width : Number)` | 设置此盒子宽度的下限和上限 |
| `use horizontal layout` | 在此盒子内部从左到右排列子盒子 |
| `use overlay layout` | 在此盒子内部将子盒子分层叠放 |
| `use vertical layout` | 在此盒子内部从上到下排列子盒子；这是默认布局 |

## B.3 `collections`

创建条目集合。

| `create link collection : Link Collection` | 创建一个空的链接集合 |
| `create location collection : Location Collection` | 创建一个空的位置集合 |
| `create message collection : Message Collection` | 创建一个空的消息集合 |
| `create number collection : Number Collection` | 创建一个空的数字集合 |
| `create number map : Number Map` | 创建一个空的数字映射 |
| `create place collection : Place Collection` | 创建一个空的地点集合 |
| `create string collection : String Collection` | 创建一个空的字符串集合 |
| `create string map : String Map` | 创建一个空的字符串映射（区分大小写和区域性） |

## B.4 `colors`

访问预定义颜色或创建新颜色。

| `accent : Color` | 获取当前主题中的强调色 |
| `background : Color` | 获取当前主题中的背景色 |
| `black : Color` | 获取 ARGB 值为 `#FF000000` 的颜色 |
| `blue : Color` | 获取 ARGB 值为 `#FF0000FF` 的颜色 |
| `brown : Color` | 获取 ARGB 值为 `#FFA52A2A` 的颜色 |
| `chrome : Color` | 获取当前主题中的铬色（控件背景） |
| `cyan : Color` | 获取 ARGB 值为 `#FF00FFFF` 的颜色 |
| `dark gray : Color` | 获取 ARGB 值为 `#FFA9A9A9` 的颜色 |
| `foreground : Color` | 获取当前主题中的前景色 |
| `from ahsb(alpha : Number, hue : Number, saturation : Number, brightness : Number) : Color` | 从 Alpha、色相、饱和度、亮度通道创建颜色（范围为 0.0-1.0） |
| `from argb(alpha : Number, red : Number, green : Number, blue : Number) : Color` | 从 Alpha、红色、绿色、蓝色通道创建颜色（范围为 0.0-1.0） |
| `from hsb(hue : Number, saturation : Number, brightness : Number) : Color` | 从色相、饱和度、亮度通道创建颜色（范围为 0.0-1.0） |
| `from rgb(red : Number, green : Number, blue : Number) : Color` | 从红色、绿色、蓝色通道创建颜色（范围为 0.0-1.0） |
| `gray : Color` | 获取 ARGB 值为 `#FF808080` 的颜色 |
| `green : Color` | 获取 ARGB 值为 `#FF008000` 的颜色 |
| `is light theme : Boolean` | 指示用户是否在其手机上使用浅色主题 |
| `light gray : Color` | 获取 ARGB 值为 `#FFD3D3D3` 的颜色 |
| `linear gradient(c1 : Color, c2 : Color, alpha : Number) : Color` | 计算中间色 |
| `magenta : Color` | 获取 ARGB 值为 `#FFFF00FF` 的颜色 |
| `orange : Color` | 获取 ARGB 值为 `#FFFFA500` 的颜色 |
| `purple : Color` | 获取 ARGB 值为 `#FF800080` 的颜色 |
| `random : Color` | 随机选取一种颜色 |
| `red : Color` | 获取 ARGB 值为 `#FFFF0000` 的颜色 |
| `sepia : Color` | 获取 ARGB 值为 `#FF704214` 的颜色 |
| `subtle : Color` | 获取当前主题中的微差色（浅灰色） |
| `transparent : Color` | 获取 ARGB 值为 `#00FFFFFF` 的颜色 |
| `white : Color` | 获取 ARGB 值为 `#FFFFFFFF` 的颜色 |
| `yellow : Color` | 获取 ARGB 值为 `#FFFFFF00` 的颜色 |

## B.5 `contract`

用于测试正确性要求是否满足的语句。

| `assert(condition : Boolean, message : String)` | 检查条件；如果条件为假，则执行失败。对已发布的脚本不执行任何操作。 |
| `requires(condition : Boolean, message : String)` | 指定操作的前置条件契约；如果条件为假，则执行失败。对已发布的脚本不执行任何操作。 |

## B.6 `invalid`

为任意数据类型创建一个无效值。



| action: `Action` | 创建一个无效的 `Action` 实例 |
| appointment: `Appointment` | 创建一个无效的 `Appointment` 实例 |
| appointment collection: `Appointment Collection` | 创建一个无效的 `Appointment Collection` 实例 |
| board: `Board` | 创建一个无效的 `Board` 实例 |
| boolean: `Boolean` | 创建一个无效的 `Boolean` 实例 |
| camera: `Camera` | 创建一个无效的 `Camera` 实例 |
| color: `Color` | 创建一个无效的 `Color` 实例 |
| contact: `Contact` | 创建一个无效的 `Contact` 实例 |
| contact collection: `Contact Collection` | 创建一个无效的 `Contact Collection` 实例 |
| datetime: `DateTime` | 创建一个无效的 `DateTime` 实例 |
| device: `Device` | 创建一个无效的 `Device` 实例 |
| device collection: `Device Collection` | 创建一个无效的 `Device Collection` 实例 |
| form builder: `Form Builder` | 创建一个无效的 `Form Builder` 实例 |
| json builder: `Json Builder` | 创建一个无效的 `Json Builder` 实例 |
| json object: `Json Object` | 创建一个无效的 `Json Object` 实例 |
| link: `Link` | 创建一个无效的 `Link` 实例 |
| link collection: `Link Collection` | 创建一个无效的 `Link Collection` 实例 |
| location: `Location` | 创建一个无效的 `Location` 实例 |
| location collection: `Location Collection` | 创建一个无效的 `Location Collection` 实例 |
| map: `Map` | 创建一个无效的 `Map` 实例 |
| Matrix: `Matrix` | 创建一个无效的 `Matrix` 实例 |
| media link: `Media Link` | 创建一个无效的 `Media Link` 实例 |
| media link collection: `Media Link Collection` | 创建一个无效的 `Media Link Collection` 实例 |
| media player: `Media Player` | 创建一个无效的 `Media Player` 实例 |
| media player collection: `Media Player Collection` | 创建一个无效的 `Media Player Collection` 实例 |
| media server: `Media Server` | 创建一个无效的 `Media Server` 实例 |
| media server collection: `Media Server Collection` | 创建一个无效的 `Media Server Collection` 实例 |
| message: `Message` | 创建一个无效的 `Message` 实例 |
| message collection: `Message Collection` | 创建一个无效的 `Message Collection` 实例 |
| message collection action: `Action` | 创建一个无效的 `Message Collection Action` 实例 |
| motion: `Motion` | 创建一个无效的 `Motion` 实例 |
| number: `Number` | 创建一个无效的 `Number` 实例 |
| number collection: `Number Collection` | 创建一个无效的 `Number Collection` 实例 |
| number map: `Number Map` | 创建一个无效的 `Number Map` 实例 |
| oauth response: `OAuth Response` | 创建一个无效的 `OAuth Response` 实例 |
| page: `Page` | 创建一个无效的 `Page` 实例 |
| page button: `Page Button` | 创建一个无效的 `Page Button` 实例 |
| page collection: `Page Collection` | 创建一个无效的 `Page Collection` 实例 |
| picture: `Picture` | 创建一个无效的 `Picture` 实例 |
| picture album: `Picture Album` | 创建一个无效的 `Picture Album` 实例 |
| picture albums: `Picture Albums` | 创建一个无效的 `Picture Albums` 实例 |
| pictures: `Pictures` | 创建一个无效的 `Pictures` 实例 |
| place: `Place` | 创建一个无效的 `Place` 实例 |
| place collection: `Place Collection` | 创建一个无效的 `Place Collection` 实例 |
| playlist: `Playlist` | 创建一个无效的 `Playlist` 实例 |
| playlists: `Playlists` | 创建一个无效的 `Playlists` 实例 |
| position action: `Action` | 创建一个无效的 `Position Action` 实例 |
| song: `Song` | 创建一个无效的 `Song` 实例 |
| song album: `Song Album` | 创建一个无效的 `Song Album` 实例 |
| song albums: `Song Albums` | 创建一个无效的 `Song Albums` 实例 |
| songs: `Songs` | 创建一个无效的 `Songs` 实例 |
| sound: `Sound` | 创建一个无效的 `Sound` 实例 |
| sprite: `Sprite` | 创建一个无效的 `Sprite` 实例 |
| sprite action: `Action` | 创建一个无效的 `Sprite Action` 实例 |
| sprite set: `Sprite Set` | 创建一个无效的 `Sprite Set` 实例 |
| sprite set action: `Action` | 创建一个无效的 `Sprite Set Action` 实例 |
| string: `String` | 创建一个无效的 `String` 实例 |
| string collection: `String Collection` | 创建一个无效的 `String Collection` 实例 |
| string map: `String Map` | 创建一个无效的 `String Map` 实例 |
| text action: `Action` | 创建一个无效的 `Text Action` 实例 |
| textbox: `TextBox` | 创建一个无效的 `TextBox` 实例 |
| vector3: `Vector3` | 创建一个无效的 `Vector3` 实例 |
| vector action: `Action` | 创建一个无效的 `Vector Action` 实例 |
| web request: `Web Request` | 创建一个无效的 `Web Request` 实例 |
| web response: `Web Response` | 创建一个无效的 `Web Response` 实例 |
| webresponse action: `Action` | 创建一个无效的 `WebResponse Action` 实例 |
| xml object: `Xml Object` | 创建一个无效的 `Xml Object` 实例 |

## B.7 语言

翻译以及语音转文字服务。

| current language: `String` | 获取当前语言代码，用于 `translate` 方法 |
| detect language(`text`: `String`): `String` | 使用 Bing 自动检测给定文本的语言。 |
| picture to text(`lang`: `String`, `pic`: `Picture`): `String` | 使用微软研究院的 Project Hawaii 提取图片中的文本。 |
| record text: `String` | 使用微软研究院的 Project Hawaii 将麦克风口述内容转换为文本。 |
| speak(`lang`: `String`, `text`: `String`): `Sound` | 使用 Bing 以指定语言朗读文本。 |
| speech to text(`lang`: `String`, `speech`: `Sound`): `String` | 使用微软研究院的 Project Hawaii 将声音转换为文本。 |
| translate(`source lang`: `String`, `target lang`: `String`, `text`: `String`): `String` | 使用 Bing 在两种语言之间翻译一些文本。源语言为空时则自动检测。 |

## B.8 位置

地理坐标服务。

| create location(`latitude`: `Number`, `longitude`: `Number`): `Location` | 创建一个新的地理坐标位置 |
| create location list: `Location Collection` | 创建一个空的位置列表 |
| describe location(`location`: `Location`): `String` | 使用 Bing 查找位置附近的地址 |
| search location(`address`: `String`, `postal code`: `String`, `city`: `String`, `country`: `String`): `Location` | 使用 Bing 查找地址的坐标 |

## B.9 地图

地图、位置转地址和地址转位置服务。

| create full map: `Map` | 创建一个全屏的 Bing 地图。使用 `post to wall` 显示它。 |
| create map: `Map` | 创建一个 Bing 地图。使用 `post to wall` 显示它。 |
| directions(`from`: `Location`, `to`: `Location`, `walking`: `Boolean`): `Location Collection` | 使用 Bing 计算两个坐标之间的方向。 |
| open directions(`start search`: `String`, `start loc`: `Location`, `end search`: `String`, `end loc`: `Location`) | 在 Bing 地图应用中显示方向。如果提供了搜索词，则忽略位置信息。为起点和终点提供搜索词或位置。 |
| open map(`center`: `Location`, `search`: `String`, `zoom`: `Number`) | 打开 Bing 地图应用。缩放介于 0（近）和 1（远）之间。 |



## B.10 `math`

数学常量、运算符和函数，例如 `cos`、`sin` 等。

| `∞- : Number` | 返回负无穷大 |
| `∞+ : Number` | 返回正无穷大 |
| `abs(x : Number) : Number` | 返回一个数的绝对值 |
| `acos(x : Number) : Number` | 返回余弦值为指定数的角度 |
| `asin(x : Number) : Number` | 返回正弦值为指定数的角度 |
| `atan(x : Number) : Number` | 返回正切值为指定数的角度 |
| `atan2(y : Number, x : Number) : Number` | 返回正切值为两个指定数之商的角度 |
| `ceiling(x : Number) : Number` | 返回大于或等于指定数的最小整数值 |
| `cos(angle : Number) : Number` | 返回指定角度的余弦值 |
| `cosh(angle : Number) : Number` | 返回指定角度的双曲余弦值 |
| `create matrix(rows : Number, columns : Number) : Matrix` | 创建一个指定大小的零矩阵 |
| `create vector3(x : Number, y : Number, z : Number) : Vector3` | 创建一个三维向量 |
| `deg to rad(degrees : Number) : Number` | 将角度转换为弧度 |
| `e : Number` | 返回自然对数的底数，由常量 e 指定 |
| `ε : Number` | 返回机器 epsilon，即大于零的最小正数 |
| `exp(x : Number) : Number` | 返回 e 的指定次幂 |
| `floor(x : Number) : Number` | 返回小于或等于指定数的最大整数 |
| `gravity : Number` | 返回标准重力值 (9.80665)，单位为米/秒² |
| `ieee remainder(x : Number, y : Number) : Number` | 返回一个指定数除以另一个指定数所得的余数 |
| `is ∞(x : Number) : Boolean` | 指示一个数是否为负无穷大或正无穷大 |
| `is ∞-(x : Number) : Boolean` | 指示一个数是否为负无穷大 |
| `is ∞+(x : Number) : Boolean` | 指示一个数是否为正无穷大 |
| `is nan(x : Number) : Boolean` | 指示值无法表示为数字，即非数字。通常发生在数是除以零所得的结果时。 |
| `log(x : Number, base : Number) : Number` | 返回指定数在指定底数下的对数 |
| `log10(x : Number) : Number` | 返回指定数的以 10 为底的对数 |
| `loge(x : Number) : Number` | 返回指定数的自然对数（以 e 为底） |
| `max(x : Number, y : Number) : Number` | 返回两个数中较大的一个 |
| `min(x : Number, y : Number) : Number` | 返回两个数中较小的一个 |
| `mod(x : Number, y : Number) : Number` | 返回一个数除以另一个数所得的模数 |
| `π : Number` | 返回圆周率常量 pi |
| `pow(x : Number, y : Number) : Number` | 返回指定数的指定次幂 |
| `rad to deg(radians : Number) : Number` | 将弧度转换为角度 |
| `random(max : Number) : Number` | 返回一个随机整数 x：0 ≤ x < max |
| `random normalized : Number` | 返回一个随机浮点数 x：0 ≤ x < 1 |
| `round(x : Number) : Number` | 将一个数四舍五入到最接近的整数值 |
| `round with precision(x : Number, digits : Number) : Number` | 将一个数四舍五入到指定的小数位数 |
| `sign(x : Number) : Number` | 返回一个指示数字符号的值 |
| `sin(angle : Number) : Number` | 返回指定角度的正弦值 |
| `sinh(angle : Number) : Number` | 返回指定角度的双曲正弦值 |
| `sqrt(x : Number) : Number` | 返回指定数的平方根 |
| `tan(angle : Number) : Number` | 返回指定角度的正切值 |
| `tanh(angle : Number) : Number` | 返回指定角度的双曲正切值 |

## B.11 `media`

图片和音乐。

| `choose picture : Picture` | 从媒体库中选择一张图片 |
| `choose song : Song` | 从媒体库中选择一首歌曲（计划添加到 API 中） |
| `create board(height : Number) : Board` | 创建一个新的游戏面板 |
| `create landscape board(width: Number, height : Number) : Board` | 创建一个横向模式的新游戏面板。在可旋转设备上，发布时将占据整个屏幕。 |
| `create picture(width : Number, height : Number) : Picture` | 创建一个给定尺寸的新图片 |
| `create portrait board(width: Number, height : Number) : Board` | 创建一个纵向模式的新游戏面板。在可旋转设备上，发布时将占据整个屏幕。 |
| `picture albums : Picture Albums` | 获取图片相册 |
| `pictures : Pictures` | 获取手机上的图片 |
| `playlists : Playlists` | 获取手机上的播放列表 |
| `saved pictures : Pictures` | 获取手机上已保存的图片 |
| `search marketplace(terms : String, type : String)` | 搜索 Windows Phone 商店（类型为应用或音乐） |
| `song albums : Song Albums` | 获取手机上的歌曲专辑 |
| `songs : Songs` | 获取手机上的歌曲 |

## B.12 `phone`

电话号码、震动等。

| `choose address : Link` | 从联系人中选择一个地址 |
| `choose phone number : Link` | 从联系人列表中选择一个电话号码 |
| `dial phone number(number : String)` | 开始拨打电话 |
| `power source : String` | 指示手机电源是 'battery'（电池）还是 'external'（外部电源）。 |
| `save phone number(phone number : String)` | 允许用户保存电话号码 |
| `vibrate(seconds : Number)` | 使手机震动指定的秒数（最小值为 0.02） |

## B.13 `player`

播放、停止或恢复歌曲

| `active song : Song` | 获取当前活动的歌曲（如果有） |
| `is muted : Boolean` | 指示播放器是否静音 |
| `is paused : Boolean` | 指示播放器是否暂停 |
| `is playing : Boolean` | 指示播放器是否正在播放歌曲 |
| `is repeating : Boolean` | 指示播放器是否重复播放 |
| `is shuffled : Boolean` | 指示播放器是否随机播放 |
| `is stopped : Boolean` | 指示播放器是否停止 |
| `next` | 跳到正在播放的歌曲队列中的下一首 |
| `pause` | 暂停当前正在播放的歌曲 |
| `play(song : Song)` | 播放一首歌曲 |
| `play home media(media : Media Link)` | 从家庭网络播放音频/视频文件 |
| `play many(songs : Songs)` | 播放一组歌曲 |
| `play position : Number` | 获取当前活动歌曲中的播放位置（秒） |
| `previous` | 跳到正在播放的歌曲队列中的上一首 |
| `resume` | 恢复播放已暂停的歌曲 |
| `set repeating(repeating : Boolean)` | 设置重复播放的开启和关闭 |
| `set shuffled(shuffled : Boolean)` | 设置随机播放的开启和关闭 |
| `set sound volume(x : Number)` | 设置音量级别，范围为 0（静音）到 1（当前音量） |
| `sound volume : Number` | 获取声音的音量，范围为 0（静音）到 1（当前音量） |
| `stop` | 停止播放歌曲 |



## B.14 传感器

摄像头、定位、麦克风及其他传感器。

| `acceleration quick`：`Vector3` | 获取经过滤波的加速度计数据，采用每轴低通与阈值触发高通组合滤波，消除传感器大部分低振幅噪声，同时能快速跟踪大幅偏移（此时信号并非完全平滑），延迟极低。适合需要快速响应的 UI 更新。 |
| `acceleration smooth`：`Vector3` | 获取经过滤波的加速度计数据，采用每轴 1Hz 一阶低通滤波，消除主要传感器噪声，延迟适中。可用于需要平滑信号的中等响应 UI 更新。 |
| `acceleration stable`：`Vector3` | 获取经过滤波和时间平均的加速度计数据，采用最近 25 个“最优滤波”样本的算术平均（每轴 50Hz 下超过 500ms），几乎消除大多数传感器噪声。该数值非常稳定，但延迟很高，不适用于快速响应的 UI。 |
| `battery level`：`Number` | 获取电池电量，范围为 0（耗尽）到 1（充满）。若信息不可用则返回无效值。 |
| `camera`：`Camera` | 获取主摄像头 |
| `current location`：`Location` | 获取当前手机位置。手机会根据功耗、性能和其他成本因素优化定位精度。 |
| `current location accurate`：`Location` | 以最高精度获取当前手机位置。这可能涉及使用收费服务，或消耗更高的电量或网络带宽。 |
| `front camera`：`Camera` | 获取前置摄像头 |
| `has gyroscope`：`Boolean` | 指示设备是否支持陀螺仪 |
| `heading`：`Number` | 获取指南针航向，单位为度，从地球地理北极顺时针测量。 |
| `is device stable`：`Boolean` | 指示设备是否处于"稳定"状态（约 0.5 秒无移动） |
| `motion`：`Motion` | 获取当前手机运动数据，综合了加速度计、指南针和陀螺仪的数据。 |
| `orientation`：`Vector3` | 获取当前方向（若可用），单位为度。(x,y,z) 也称为 (pitch, roll, yaw) 或 (alpha, beta, gamma)。 |
| `record microphone`：`Sound` | 使用麦克风录制音频 |
| `rotation speed`：`Vector3` | 获取陀螺仪围绕设备各轴的旋转速度，单位为度/秒。 |
| `take camera picture`：`Picture` | 拍照并返回图片。此图片不包含 GPS 位置信息。 |

## B.15 社交

电子邮件、短信、联系人和日历服务。

| `choose contact`：`Contact` | 从联系人列表中选择一个联系人 |
| `choose email`：`Link` | 从联系人列表中选择一个电子邮件 |
| `create contact(nickname : String)`：`Contact` | 创建一个新联系人 |
| `create message(message : String)`：`Message` | 创建一条要分享的消息 |
| `create place(name : String, location : Location)`：`Place` | 创建一个地点 |
| `link email(email address : String)`：`Link` | 从电子邮件创建一个链接 |
| `link phone number(phone number : String)`：`Link` | 从电话号码创建一个链接 |
| `save contact(contact : Contact)` | 保存一个新联系人 |
| `save email(email address : String)` | 允许用户保存电子邮件地址 |
| `search(network : String, terms : String)`：`Message Collection` | 在社交网络（推特、脸书）中搜索最近的帖子 |
| `search appointments(start : DateTime, end : DateTime)`：`Appointment Collection` | 在指定时间范围内搜索约会事项 |
| `search contacts(prefix : String)`：`Contact Collection` | 按名称搜索联系人 |
| `search places nearby(network : String, terms : String, location : Location, distance : Number)`：`Place Collection` | 搜索附近的地点。距离单位为米。 |
| `send email(to : String, subject : String, body : String)` | 打开邮件客户端 |
| `send sms(to : String, body : String)` | 打开短信客户端（收件人、正文） |

## B.16 标签

二维条码、二维码和 NFC 标签。

| `send nfc(value : String, type : String)` | 使用 NFC 发送网址或文本 |
| `tag text(text : String, size : Number, bw : Boolean)`：`Picture` | 使用微软标签生成指向文本的二维条码。文本长度必须小于 1000 字符，尺寸必须在 0.75 到 5 英寸之间。 |
| `tag url(url : String, size : Number, bw : Boolean)`：`Picture` | 使用微软标签生成指向网址的二维条码。网址长度必须小于 1000 字符，尺寸必须在 0.75 到 5 英寸之间。 |

## B.17 磁贴

Windows 8 和 Windows Phone 在开始屏幕上显示磁贴，点击后可启动应用程序。通过此服务可将磁贴与 TouchDevelop 脚本关联。注意：本资源替代了不再支持的`Tile`数据类型。

| `pin default` | 提示用户是否应将磁贴固定到开始屏幕 |
| `set default counter( n : Number )` | 设置显示在磁贴上的计数器（数字）；仅显示 1 到 99 之间的值；其余值隐藏 |
| `set default text( title: String, text : String)` | 在磁贴上显示短标题和较长文本 |

## B.18 时间

时间和日期操作。

| `create(year : Number, month : Number, day : Number, hour : Number, minute : Number, second : Number)`：`DateTime` | 创建一个新的日期实例 |
| `fail if not(condition : Boolean)` | 如果条件为假，则中止执行。 |
| `log(message : String)` | 将此消息追加到调试日志中。脚本发布后不执行任何操作。 |
| `now`：`DateTime` | 获取当前时间 |
| `sleep(seconds : Number)` | 等待指定的秒数 |
| `stop` | 停止执行并停留在墙（指 TouchDevelop 的 Wall 界面）上。 |
| `today`：`DateTime` | 获取今天的日期（不含时间） |
| `tomorrow`：`DateTime` | 获取明天的日期（不含时间） |



### B.19 墙面

请求或显示墙面上的值。

| `add button(icon : String, text : String) : Page Button` | 添加一个新按钮。`icon` 必须是内置图标的名称，`text` 不能为空。 |
| `ask boolean(text : String, caption : String) : Boolean` | 使用`确定`和`取消`按钮提示用户 |
| `ask number(text : String) : Number` | 提示用户输入一个数字 |
| `ask string(text : String) : String` | 提示用户输入一个字符串 |
| `button icon names : String Collection` | 获取可用页面按钮名称的列表。 |
| `clear` | 清除背景、按钮和条目 |
| `clear background` | 清除背景颜色、图片和摄像头 |
| `clear buttons` | 清除应用栏按钮并隐藏该栏 |
| `create text box(text : String, font size : Number) : TextBox` | 创建一个可更新的文本框 |
| `current page : Page` | 获取当前在墙面上显示的页面 |
| `display search(on : Boolean)` | 指示是否显示或隐藏搜索图标 |
| `pages : Page Collection` | 返回当前页面的返回栈，从当前页面到底部页面。 |
| `pick date(text : String, caption : String) : DateTime` | 提示用户选择日期。返回一个设置了日期的日期时间对象，时间为 12:00:00。 |
| `pick string(text : String, caption : String, values : String Collection) : Number` | 提示用户从列表中选择一个字符串。返回所选索引。 |
| `pick time(text : String, caption : String) : DateTime` | 提示用户选择时间。返回一个设置了时间的日期时间对象，日期未定义。 |
| `pop page : Boolean` | 弹出当前页面并恢复上一个墙面页面。如果已在默认页面，则返回`false`。 |
| `prompt(text : String)` | 使用`确定`按钮提示用户 |
| `push new page : Page` | 在墙面上推送一个空白页面 |
| `screenshot : Picture` | 截取墙面的屏幕截图 |
| `set background(color : Color)` | 设置墙面背景颜色 |
| `wall→set background camera( camera : Camera)` | 设置墙面背景摄像头 |
| `set background picture(picture : Picture)` | 设置墙面背景图片。图片将根据需要调整大小并裁剪到屏幕背景 |
| `set foreground(color : Color)` | 设置墙面元素的前景色 |
| `set reversed(bottom : Boolean)` | 反转墙面上的元素，并在底部插入新元素。 |
| `set subtitle(title : String)` | 设置墙面的副标题 |
| `set title(title : String)` | 设置墙面的标题 |
| `set transform matrix(m11 : Number, m12 : Number, m21 : Number, m22 : Number, offsetx : Number, offsety : Number)` | 设置应用于墙面的 3x3 仿射矩阵变换 |

### B.20 网络

搜索和浏览网页。

| `base64 decode(text : String) : String` | 解码一个经过 base64 编码的字符串 |
| `base64 encode(text : String) : String` | 将一个字符串转换为 base64 编码的字符串 |
| `browse(url : String)` | 打开一个网页浏览器访问某个网址 |
| `connection name : String` | 获取当前连接网络中为互联网请求提供服务的名称 |
| `connection type : String` | 获取为互联网请求提供服务的网络类型（unknown, none, ethernet, wifi, mobile） |
| `create form builder : Form Builder` | 创建一个表单构建器 |
| `create json builder : Json Builder` | 创建一个 JSON 构建器 |
| `create request(url : String) : Web Request` | 创建一个网页请求 |
| `csv(text : String, delimiter : String) : Json Object` | 将逗号分隔值文档解析为 `JsonObject`，其中 headers 是列名的字符串数组；records 是行数组，每行本身是一个字符串数组。如果未指定分隔符，则会推断分隔符。 |
| `download(url : String) : String` | 下载一个互联网页面的内容（http get） |
| `download json(url : String) : Json Object` | 将网络服务响应下载为 JSON 数据结构（http get） |
| `download picture(url : String) : Picture` | 从互联网下载一张图片 |
| `download song(url : String, name : String) : Song` | 从互联网创建一个流式歌曲文件（播放时下载） |
| `download sound(url : String) : Sound` | 从互联网下载一个 WAV 声音文件 |
| `feed(value : String) : Message Collection` | 将新闻提要字符串（RSS 2.0 或 Atom 1.0）解析为消息集合 |
| `html decode(html : String) : String` | 解码一个经过 HTML 编码的字符串 |
| `html encode(text : String) : String` | 将一个文本字符串转换为 HTML 编码的字符串 |
| `is connected : Boolean` | 指示是否有任何网络连接可用 |
| `json(value : String) : Json Object` | 将字符串解析为 JSON 对象 |
| `json array : Json Object` | 返回一个空的 JSON 数组 |
| `json object : Json Object` | 返回一个空的 JSON 对象 |
| `link image(url : String) : Link` | 创建一个指向互联网图片的链接 |
| `link media(url : String) : Link` | 创建一个指向互联网音频/视频的链接 |
| `link url(name : String, url : String) : Link` | 创建一个指向互联网页面的链接 |
| `oauth v2(oauth url : String) : OAuth Response` | 使用 OAuth 2.0 进行身份验证并接收访问令牌或错误。（有关选择重定向 URI 的更多信息，请参见 `oauthv2`。） |
| `open connection settings(page : String)` | 打开连接设置页面（airplanemode, bluetooth, wiki, cellular） |
| `play media(url : String)` | 以全屏模式播放互联网音频/视频 |
| `search(terms : String) : Link Collection` | 使用 Bing 搜索网络 |
| `search images(terms : String) : Link Collection` | 使用 Bing 搜索图片 |
| `search images nearby(terms : String, location : Location, distance : Number) : Link Collection` | 使用 Bing 搜索某个位置附近的图片。距离以米为单位，负值忽略。 |
| `search nearby(terms : String, location : Location, distance : Number) : Link Collection` | 使用 Bing 搜索某个位置附近的网络信息。距离以米为单位，负值忽略。 |
| `search news(terms : String) : Link Collection` | 使用 Bing 搜索新闻 |
| `search news nearby(terms : String, location : Location, distance : Number) : Link Collection` | 使用 Bing 搜索某个位置附近的新闻。距离以米为单位，负值忽略。 |
| `upload(url : String, body : String) : String` | 将文本上传到互联网页面（http post） |
| `upload picture(url : String, pic : Picture) : String` | 将图片上传到互联网页面（http post） |
| `url decode(url : String) : String` | 解码一个经过 URL 编码的字符串 |
| `url encode(text : String) : String` | 将一个文本字符串转换为 URL 编码的字符串 |
| `xml(value : String) : Xml Object` | 将字符串解析为 XML 元素 |

![知识共享许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0) 开放获取 本章根据知识共享署名-非商业性使用-禁止演绎 4.0 国际许可协议 ([`creativecommons.org/licenses/by-nc-nd/4.0/`](http://creativecommons.org/licenses/by-nc-nd/4.0/)) 的条款提供许可，该协议允许任何非商业性使用、共享、分发和复制，只要您给予原作者和来源适当的署名，提供指向知识共享许可协议的链接，并指出您是否修改了许可材料。根据本许可协议，您无权分享从本章或其部分内容衍生的改编材料。本章中的图片或其他第三方材料已包含在本章的知识共享许可协议中，除非在材料的出处说明中另有注明。如果材料未包含在本章的知识共享许可协议中，并且您的预期使用不受法定法规允许或超出允许的使用范围，您将需要直接从版权所有者处获得许可。



