# TouchDevelop 数据类型

C.1 日历约会 C.2 日历约会集合 C.3 游戏面板 C.4 布尔值 C.5 摄像头 C.6 颜色 C.7 联系人 C.8 联系人集合 C.9 日期时间 C.10 表单生成器 C.11 JSON 生成器 C.12 JSON 对象 C.13 链接 C.14 链接集合 C.15 位置 C.16 位置集合 C.17 地图 C.18 矩阵 C.19 消息 C.20 消息集合 C.21 运动 C.22 数字 C.23 数字集合 C.24 数字映射 C.25 OAuth 响应 C.26 页面 C.27 页面按钮 C.28 页面集合 C.29 图片 C.30 图片专辑 C.31 图片专辑集合 C.32 图片集 C.33 地点 C.34 地点集合 C.35 播放列表 C.36 播放列表集合 C.37 歌曲 C.38 歌曲集合 C.39 歌曲专辑 C.40 歌曲专辑集合 C.41 歌曲集 C.42 声音 C.43 精灵 C.44 精灵集合 C.45 字符串 C.46 字符串集合 C.47 字符串映射 C.48 文本框 C.49 三维向量 C.50 Web 请求 C.51 Web 响应 C.52 XML 对象

本附录复现了 TouchDevelop 网站上 [`https://www.touchdevelop.com/docs/api`](https://www.touchdevelop.com/docs/api) 中的资料，提供了对 TouchDevelop 中实现的数据类型的描述。附录 B 涵盖了服务（也称为资源）。

## C.1 日历约会

此类型的值用于描述一个日历约会。

| `attendees` : `Contact Collection` | 获取与会者列表。每个联系人包含姓名和电子邮件地址。 |
| `details` : `String` | 获取详细说明 |
| `end time` : `DateTime` | 获取结束时间 |
| `is all day event` : `Boolean` | 指示是否为全天事件 |
| `is invalid` : `Boolean` | 如果当前实例无效，则返回 true |
| `is private` : `Boolean` | 指示此约会是否为私人事件 |
| `location` : `String` | 获取地点 |
| `organizer` : `Contact` | 获取组织者 |
| `post to wall` | 将约会发布到墙上 |
| `source` : `String` | 获取此约会的来源（Facebook 等） |
| `start time` : `DateTime` | 获取开始时间（注：原文此处可能笔误，应为 start time） |
| `status` : `String` | 获取您的状态（空闲、待定、忙碌、外出） |
| `subject` : `String` | 获取主题 |

## C.2 日历约会集合

此类型的值表示一个约会集合。

| `at(index` : `Number)` : `Appointment` | 获取指定索引处的约会 |
| `count` : `Number` | 获取约会的数量 |
| `is invalid` : `Boolean` | 如果当前实例无效，则返回 true |
| `post to wall` | 将约会集合发布到墙上 |

## C.3 游戏面板

`Board` 的实例是一个包含精灵和其他图形对象的 2D 图像，由游戏程序显示。

| `at(i` : `Number)` : `Sprite` | 获取索引为 i 的精灵 |
| `clear background camera` | 清除背景摄像头 |
| `clear background picture` | 清除背景图片 |
| `clear events` | 清除所有与此面板相关的排队事件 |
| `count` : `Number` | 获取精灵数量 |
| `create anchor(width` : `Number, height` : `Number)` : `Sprite` | 创建一个锚点精灵 |
| `create boundary(distance` : `Number)` | 在给定距离处围绕面板创建墙壁 |
| `create ellipse(width` : `Number, height` : `Number)` : `Sprite` | 创建一个新的椭圆精灵 |
| `create obstacle(x` : `Number, y` : `Number, x segment` : `Number, y segment` : `Number, elasticity` : `Number)` | 使用给定的起点和给定的范围创建一条线性障碍物。弹性系数 0 表示粘性，1 表示完全反弹。 |
| `create picture(picture` : `Picture)` : `Sprite` | 创建一个新的图片精灵。 |
| `create rectangle(width` : `Number, height` : `Number)` : `Sprite` | 创建一个新的矩形精灵 |
| `create spring(sprite1` : `Sprite, sprite2` : `Sprite, stiffness` : `Number)` | 在两个精灵之间创建一个弹簧 |
| `create sprite set` : `Sprite Set` | 为精灵创建一个新的集合 |
| `create text(width` : `Number, height` : `Number, fontSize` : `Number, text` : `String)` : `Sprite` | 创建一个新的文本精灵。 |
| `evolve` | 更新面板上精灵的位置。 |
| `height` : `Number` | 获取以像素为单位的高度 |
| `is invalid` : `Boolean` | 如果当前实例无效，则返回 true |
| `is landscape` : `Boolean` | 获取一个值，指示面板是否为横屏模式设计 |
| `post to wall` | 在墙上显示面板。 |
| `set background(color` : `Color)` | 设置背景颜色 |
| `set background camera(camera` : `Camera)` | 设置背景摄像头 |
| `set background picture(picture` : `Picture)` | 设置背景图片 |
| `set debug mode(debug` : `Boolean)` | 在调试模式下，面板会显示精灵的速度和其他信息 |
| `set friction(friction` : `Number)` | 将精灵的默认摩擦力设置为 0 到 1 之间的速度损失比例 |
| `set gravity(x` : `Number, y` : `Number)` | 将面板上物体的均匀加速度向量设置为像素/秒² |
| `touch current` : `Vector3` | 当前触摸点 |
| `touch end` : `Vector3` | 上次触摸结束点 |
| `touch start` : `Vector3` | 上次触摸起始点 |
| `touch velocity` : `Vector3` | 触摸结束后的最终触摸速度 |
| `touched` : `Boolean` | 如果面板被触摸过则为 true |
| `update on wall` | 使更新可见。 |
| `width` : `Number` | 获取以像素为单位的宽度 |

## C.4 布尔值

该数据类型只有 `true` 和 `false` 两个值。

| `and(right` : `Boolean)` : `Boolean` | 构造逻辑与 |
| `equals(right` : `Boolean)` : `Boolean` | 指示两个值是否相等 |
| `is invalid` : `Boolean` | 如果当前实例无效，则返回 true |
| `not` : `Boolean` | 对布尔表达式取反 |
| `or(right` : `Boolean)` : `Boolean` | 构造逻辑或 |
| `post to wall` | 在墙上显示该值 |
| `to json` : `Json Object` | 将该值转换为 json 数据结构 |
| `to number` : `Number` | 将 true 转换为 1，false 转换为 0 |
| `to string` : `String` | 将布尔值转换为字符串 |

## C.5 摄像头

前置或后置摄像头。

| `height` : `Number` | 获取摄像头图像的高度（像素） |
| `is front` : `Boolean` | 如果这是主摄像头则返回 false，否则返回 true |
| `is invalid` : `Boolean` | 如果当前实例无效，则返回 true |
| `post to wall` | 全屏显示摄像头视频流 |
| `preview` : `Picture` | 从摄像头获取一张低质量图片 |
| `width` : `Number` | 获取摄像头图像的宽度（像素） |



## C.6 颜色

一种 ARGB 颜色（alpha 透明度, 红色, 绿色, 蓝色）

| `A` : `数值` | 获取 Alpha 透明度值（0.0-1.0） |
| `B` : `数值` | 获取蓝色值（0.0-1.0） |
| `blend(other : Color) : Color` | 使用 Alpha 混合合成一种新颜色 |
| `brightness` : `数值` | 获取颜色的亮度分量 |
| `darken(delta : 数值) : 颜色` | 根据介于 0 和 1 之间的 delta 值，生成一种更暗的颜色 |
| `equals(other : 颜色) : 布尔值` | 检查颜色是否与另一种颜色相等 |
| `G` : `数值` | 获取绿色值（0.0-1.0） |
| `hue` : `数值` | 获取颜色的色相分量 |
| `is invalid` : `布尔值` | 如果当前实例无效则返回 true |
| `lighten(delta : 数值) : 颜色` | 根据介于 0 和 1 之间的 delta 值，生成一种更亮的颜色 |
| `make transparent(alpha : 数值) : 颜色` | 通过更改 Alpha 通道创建一个新颜色，范围从 0（透明）到 1（不透明） |
| `post to wall` | 将值打印到墙面 |
| `R` : `数值` | 获取红色值（0.0-1.0） |
| `saturation` : `数值` | 获取颜色的饱和度分量 |

## C.7 联系人

此类型的实例代表一个个人联系人。方法列表分为三部分：获取方法（检索联系人的单个属性）、设置方法（设置或更新单个属性）以及其他方法。

### 联系人类型的获取方法

| `birthday` : `日期时间` | 获取出生日期（如果有）。 |
| `company` : `字符串` | 获取公司名称（如果有）。 |
| `email` : `链接` | 获取工作或个人电子邮件（如果有） |
| `first name` : `字符串` | 获取名字（如果有）。 |
| `home address` : `字符串` | 获取工作地址（如果有） |
| `home phone` : `链接` | 获取家庭电话号码（如果有） |
| `job title` : `字符串` | 获取在公司中的职位（如果有）。 |
| `last name` : `字符串` | 获取姓氏（如果有） |
| `middle name` : `字符串` | 获取中间名（如果有） |
| `mobile phone` : `链接` | 获取手机号码（如果有） |
| `name` : `字符串` | 获取显示名称（保存联系人时不使用） |
| `nick name` : `字符串` | 获取昵称（如果有） |
| `office` : `字符串` | 获取在公司中的办公室位置（如果有） |
| `personal email` : `链接` | 获取个人电子邮件（如果有） |
| `phone number` : `链接` | 获取手机、工作或家庭电话号码（如果有） |
| `picture` : `图片` | 获取联系人的图片（如果有） |
| `source` : `字符串` | 获取此联系人的来源（电话等…） |
| `suffix` : `字符串` | 获取姓名后缀（如果有） |
| `title` : `字符串` | 获取姓名称谓（如果有） |
| `web site` : `链接` | 获取网站（如果有） |
| `work address` : `字符串` | 获取家庭地址（如果有） |
| `work email` : `链接` | 获取工作电子邮件（如果有） |
| `work phone` : `链接` | 获取工作电话号码（如果有） |

### 联系人类型的设置方法

| `set company(value : 字符串)` | 设置公司 |
| `set first name(value : 字符串)` | 设置名字 |
| `set home phone(home phone : 字符串)` | 设置家庭电话 |
| `set job title(value : 字符串)` | 设置职位 |
| `set last name(value : 字符串)` | 设置姓氏 |
| `set middle name(middle name : 字符串)` | 设置中间名 |
| `set mobile phone(value : 字符串)` | 设置手机号码 |
| `set personal email(value : 字符串)` | 设置个人电子邮件 |
| `set source(value : 字符串)` | 设置来源 |
| `set suffix(value : 字符串)` | 设置后缀 |
| `set title(value : 字符串)` | 设置称谓 |
| `set web site(value : 字符串)` | 设置网站 |
| `set work email(value : 字符串)` | 设置工作电子邮件 |
| `set work phone(work phone : 字符串)` | 设置工作电话 |

### 联系人类型的其他方法

| `is invalid` : `布尔值` | 如果当前实例无效则返回 true |
| `post to wall` | 将联系人发布到墙面 |

## C.8 联系人集合

一个联系人的集合

| `at(index : 数值) : 联系人` | 获取指定索引处的联系人 |
| `count` : `数值` | 获取联系人数量 |
| `is invalid` : `布尔值` | 如果当前实例无效则返回 true |
| `post to wall` | 将联系人发布到墙面 |

## C.9 日期时间

一个 `DateTime` 值是日期和时间的组合。方法列表分为一个获取方法表（返回单个属性）和其他方法。

### 日期时间类型的获取方法

| `date` : `日期时间` | 获取日期 |
| `day` : `数值` | 获取月中的第几天 |
| `hour` : `数值` | 获取小时 |
| `millisecond` : `数值` | 获取毫秒 |
| `minute` : `数值` | 获取分钟 |
| `month` : `数值` | 获取月份 |
| `second` : `数值` | 获取秒数 |
| `week day` : `数值` | 获取星期几（星期日 = 0, 星期一 = 1, … 星期六 = 6） |
| `year` : `数值` | 获取年份 |
| `year day` : `数值` | 获取一年中的第几天，介于 1 到 366 之间 |

### 日期时间类型的其他方法

| `add days(days : 数值) : 日期时间` | 返回一个新的日期，该日期在当前实例的值上增加指定的天数 |
| `add hours(hours : 数值) : 日期时间` | 返回一个新的日期，该日期在当前实例的值上增加指定的小时数 |
| `add milliseconds(milliseconds : 数值) : 日期时间` | 返回一个新的日期，该日期在当前实例的值上增加指定的毫秒数 |
| `add minutes(minutes : 数值) : 日期时间` | 返回一个新的日期，该日期在当前实例的值上增加指定的分钟数 |
| `add months(months : 数值) : 日期时间` | 返回一个新的日期，该日期在当前实例的值上增加指定的月数 |
| `add seconds(seconds : 数值) : 日期时间` | 返回一个新的日期，该日期在当前实例的值上增加指定的秒数 |
| `add years(years : 数值) : 日期时间` | 返回一个新的日期，该日期在当前实例的值上增加指定的年数 |
| `equals(other: DateTime): Boolean` | 比较日期是否相等 |
| `greater(other: DateTime): Boolean` | 比较一个日期是否大于另一个 |
| `greater or equal(other : DateTime) : Boolean` | 比较一个日期是否大于或等于另一个 |
| `is invalid` : `布尔值` | 如果当前实例无效则返回 true |
| `less(other : DateTime) : Boolean` | 比较一个日期是否小于另一个 |
| `less or equals(other : DateTime) : Boolean` | 比较一个日期是否小于或等于另一个 |
| `not equals(other : DateTime) : Boolean` | 比较日期是否不相等 |
| `post to wall` | 将日期打印到墙面 |
| `subtract(value: DateTime): Number` | 计算两个日期时间之间的差值，以秒为单位 |
| `to local time : DateTime` | 转换为本地时间 |
| `to json : Json 对象` | 将该值转换为一个 json 数据结构 |
| `to local time : DateTime` | 转换为本地时间 |
| `to string : 字符串` | 将日期转换为字符串 |
| `to universal time : DateTime` | 转换为协调世界时 |

## C.10 表单生成器

用于创建 HTML 表单数据的构建器。

| `add boolean(name : 字符串, value : 布尔值)` | 添加一个布尔值 |
| `add number(name : 字符串, value : 数值)` | 添加一个数值 |
| `add picture(name : 字符串, value : 图片, picture Name : 字符串)` | 添加一张图片 |
| `add string(name : 字符串, value : 字符串)` | 添加一个字符串值 |
| `is invalid` : `布尔值` | 如果当前实例无效则返回 true |
| `post to wall` | 将表单发布到墙面 |

## C.11 Json 生成器

一个 json 数据结构构建器。

| `add(value : Json 对象)` | 向数组添加一个值 |
| `add null` | 向数组添加一个 null 值 |
| `is invalid` : `布尔值` | 如果当前实例无效则返回 true |
| `set boolean(name : 字符串, value : 布尔值)` | 设置布尔值 |
| `set field(name : 字符串, value : Json 对象)` | 设置字段值 |
| `set field null(name : 字符串)` | 将字段值设置为 null |
| `set number(name : 字符串, value : 数值)` | 设置数值 |
| `set string(name : 字符串, value : 字符串)` | 设置字符串值 |
| `to json : Json 对象` | 将构建器转换为一个 json 数据结构并清空构建器 |



## C.12 JSON 对象

一种 JSON 数据结构

| `at(index : Number) : Json Object` | 获取第 i 个 JSON 值 |
| `boolean(key : String) : Boolean` | 将字段值作为布尔值获取 |
| `contains key(key: String) : Boolean` | 指示键是否存在 |
| `count : Number` | 获取值的数量 |
| `field(key : String) : Json Object` | 按名称获取值 |
| `is invalid : Boolean` | 若当前实例无效则返回真值 |
| `keys : String Collection` | 获取键列表 |
| `kind : String` | 获取 JSON 类型（字符串、数字、对象、数组、布尔值） |
| `number(key : String) : Number` | 将字段值作为数字获取 |
| `post to wall` | 将值打印到墙面上 |
| `string(key : String) : String` | 将字段值作为字符串获取 |
| `time(key : String) : DateTime` | 将字段值作为时间获取 |
| `to boolean : Boolean` | 转换为布尔值（类型必须为布尔） |
| `to number : Number` | 转换为数字（类型必须为数字） |
| `to string : String` | 转换为字符串（类型必须为字符串） |
| `to time : DateTime` | 转换并解析为日期时间（类型必须为字符串） |

## C.13 链接

指向视频、图像、电子邮件或电话号码的链接。

| `address : String` | 获取 URL |
| `is invalid : Boolean` | 若当前实例无效则返回真值 |
| `kind : String` | 获取资源类型 - 媒体、图像、电子邮件、电话号码、超链接、深度缩放链接、广播 |
| `location : Location` | 获取位置（若有） |
| `name : String` | 获取名称（若有） |
| `post to wall` | 在墙面上显示链接 |
| `set location(location : Location)` | 设置位置 |
| `set name(name : String)` | 设置名称 |
| `share(network : String)` | 分享链接（电子邮件、短信、Facebook、社交网络或留空以从列表中选择） |

## C.14 链接集合

链接列表。

| `add(value : Link)` | 添加一个链接 |
| `add many(value : Link Collection)` | 一次添加多个链接 |
| `at(index : Number) : Link` | 获取第 i 个链接 |
| `clear` | 清空集合 |
| `count : Number` | 获取元素数量 |
| `index of(item : Link, start : Number) : Number` | 获取项目首次出现的索引。若未找到或起始索引超出范围则返回-1。 |
| `insert at(index : Number, item : Link)` | 在指定索引位置插入一个链接。若索引超出范围则不执行任何操作。 |
| `is invalid : Boolean` | 若当前实例无效则返回真值 |
| `post to wall` | 在墙面上显示链接 |
| `random : Link` | 从集合中获取随机元素。若集合为空则返回无效值。 |
| `remove(item : Link) : Boolean` | 移除首次出现的链接。若移除成功则返回真值。 |
| `remove at(index : Number)` | 移除指定索引位置的链接。 |
| `reverse` | 反转元素的顺序。 |
| `set at(index : Number, value : Link)` | 设置第 i 个链接 |

## C.15 位置

地理坐标（纬度、经度等）

| `altitude : Number` | 获取坐标的海拔高度 |
| `course : Number` | 获取坐标的航向 |
| `distance(other : Location) : Number` | 计算距离（以米为单位） |
| `equals(other : Location) : Boolean` | 指示此实例是否与另一实例相等 |
| `hor accuracy : Number` | 获取坐标的水平精度 |
| `is invalid : Boolean` | 若当前实例无效则返回真值 |
| `latitude : Number` | 获取坐标的纬度 |
| `longitude : Number` | 获取坐标的经度 |
| `post to wall` | 使用 Bing 在地图上显示位置 |
| `share(network : String, message : String)` | 分享位置（电子邮件、短信、Facebook、社交网络或留空以从列表中选择） |
| `speed : Number` | 获取坐标的速度 |
| `to string : String` | 转换为纬度,经度字符串 |
| `vert accuracy : Number` | 获取坐标的垂直精度 |

## C.16 位置集合

位置列表。

| `add(value : Location)` | 添加一个位置 |
| `add many(value : Location Collection)` | 一次添加多个位置 |
| `at(index : Number) : Location` | 获取第 i 个地理坐标 |
| `clear` | 清空集合 |
| `count : Number` | 获取元素数量 |
| `index of(item : Location, start : Number) : Number` | 获取项目首次出现的索引。若未找到或起始索引超出范围则返回-1。 |
| `insert at(index : Number, item : Location) : Nothing` | 在指定索引位置插入一个位置。若索引超出范围则不执行任何操作。 |
| `is invalid : Boolean` | 若当前实例无效则返回真值 |
| `post to wall` | 使用 Bing 在地图上显示位置 |
| `random : Location` | 从集合中获取随机元素。若集合为空则返回无效值。 |
| `remove(item : Location) : Boolean` | 移除首次出现的位置。若移除成功则返回真值。 |
| `remove at(index: Number)` | 移除指定索引位置的位置 |
| `reverse` | 反转元素的顺序 |
| `set at(index : Number, value : Location)` | 设置第 i 个地理坐标 |
| `sort by distance(loc : Location)` | 按到该位置的距离进行排序 |

## C.17 地图

一个必应地图。

| `add line(locations : Location Collection, color : Color, thickness : Number)` | 添加一条经过多个地理坐标的折线 |
| `add link(link : Link, background : Color, foreground : Color)` | 在地图上添加一个链接图钉（若未设置位置则忽略） |
| `add message(msg : Message, background : Color, foreground : Color)` | 在地图上添加一个消息图钉（若未设置位置则忽略） |
| `add picture(location : Location, picture : Picture, background : Color)` | 在地图上添加一个图片图钉 |
| `add place(place : Place, background : Color, foreground : Color)` | 在地图上添加一个地点图钉（若未设置位置则忽略） |
| `add text(location : Location, text : String, background : Color, foreground : Color)` | 在地图上添加一个文本图钉 |
| `center : Location` | 获取地图中心位置 |
| `clear` | 清除线条、区域和图钉 |
| `fill region(locations : Location Collection, fill : Color, stroke : Color, thickness : Number)` | 用颜色填充一个区域 |
| `is invalid : Boolean` | 若当前实例无效则返回真值 |
| `post to wall` | 使用 Bing 在墙面上显示地图 |
| `set center(center : Location)` | 设置地图中心位置 |
| `set zoom(level : Number)` | 设置缩放级别，范围从 1（地球）到 21（街道） |
| `view pushpins` | 更改当前缩放和中心，使所有图钉可见。若地图尚未发布到墙面，此方法无效。 |
| `zoom : Number` | 获取缩放级别 |



## C.18 `Matrix`

一个二维数字矩阵。

| `add(b : Matrix) : Matrix` | 返回将此矩阵与 `b` 相加后得到的矩阵。两个矩阵的大小必须匹配。 |
| `at(index : Number) : Number` | 获取指定索引处的值。元素按行顺序排列，从左上角开始。 |
| `clear(value : Number)` | 将矩阵的所有元素设置为该值。 |
| `clone : Matrix` | 创建矩阵的深拷贝。 |
| `column count : Number` | 获取列数 |
| `count : Number` | 获取元素总数 |
| `is invalid : Boolean` | 如果当前实例无效则返回 `true` |
| `item(row : Number, column : Number) : Number` | 获取指定位置的值。如果超出数组维度则返回无效值。 |
| `max : Number` | 计算所有值中的最大值 |
| `min : Number` | 计算所有值中的最小值 |
| `multiply(b : Matrix) : Matrix` | 返回将矩阵中每个元素相乘后得到的矩阵。两个矩阵的大小必须匹配。 |
| `negate : Matrix` | 返回取反后的矩阵。 |
| `post to wall` | 使用必应（Bing）在墙上显示地图 |
| `random : Number` | 获取一个随机元素。如果矩阵为空则返回无效值。 |
| `row count : Number` | 获取行数 |
| `scale(factor : Number) : Matrix` | 返回按 `factor` 缩放后的矩阵副本 |
| `set at(index : Number, value : Number)` | 设置指定索引处的值。元素按行顺序排列，从左上角开始。 |
| `set item(row : Number, column : Number, value : Number)` | 设置特定位置的值。如果该位置超出边界，矩阵将用零值扩展。 |
| `subtract(b : Matrix) : Matrix` | 返回从此矩阵中减去 `b` 后得到的矩阵。两个矩阵的大小必须匹配。 |
| `to string : String` | 获取矩阵的字符串表示形式 |
| `transpose : Matrix` | 返回转置矩阵 |

## C.19 `Message`

一个 `Message` 值包含留言板上某条帖子的详细信息。方法列表分为 get 方法（获取消息的单个属性）、set 方法（设置或更新属性值）以及其他方法。

### `Message` 类型的 Get 方法

| `from : String` | 获取作者 |
| `id : String` | 获取消息标识符 |
| `link : String` | 获取与该消息关联的链接 |
| `location : Location` | 获取地理坐标 |
| `media link : String` | 获取媒体的 URL |
| `message : String` | 获取消息文本 |
| `picture link : String` | 获取图片的 URL |
| `source : String` | 获取此消息的来源（Facebook、Twitter 等…​） |
| `time : DateTime` | 获取时间 |
| `title : String` | 获取标题文本 |
| `to : String` | 获取收件人 |
| `values : String Map` | 获取消息中存储的附加值 |

### `Message` 类型的 Set 方法

| `set from(author : String)` | 设置作者 |
| `set id(value : String)` | 设置消息标识符 |
| `set link(url : String)` | 设置与该消息关联的链接 |
| `set location(location : Location)` | 设置地理坐标 |
| `set media link(url : String)` | 设置媒体的 URL |
| `set message(message : String)` | 设置消息文本 |
| `set picture link(url : String)` | 设置图片的 URL |
| `set source(source : String)` | 设置此消息的来源 |
| `set time(time : DateTime)` | 设置时间 |
| `set title(title : String)` | 设置标题文本 |
| `set to(author : String) : Nothing` | 设置收件人 |

### `Message` 类型的其他方法

| `is invalid : Boolean` | 如果当前实例无效则返回 `true` |
| `post to wall` | 将消息发布到墙上 |
| `share(where : String) : Nothing` | 共享此消息（可选参数包括 email、sms、Facebook、social 或 '' 以从列表中选择） |

## C.20 `Message Collection`

消息列表。

| `add(value : Message)` | 添加一条 `Message` |
| `add many(value : Message Collection)` | 添加一个 `Message` 项集合 |
| `at(index : Number) : Message` | 获取第 i 条 `Message` |
| `clear` | 清空集合 |
| `continuation : String` | 获取下一组消息的标识符 |
| `count : Number` | 获取元素数量 |
| `index of(item : Message, start : Number) : Number` | 获取 `item` 第一次出现时的索引。如果未找到或 `start` 超出范围，则返回 -1。 |
| `insert at(index : Number, item : Message)` | 在位置 `index` 处插入一个链接。如果 `index` 超出范围，则不执行任何操作。 |
| `is invalid : Boolean` | 如果当前实例无效则返回 `true` |
| `post to wall` | 在墙上显示消息 |
| `random : Message` | 从集合中获取一个随机元素。如果集合为空则返回无效值。 |
| `remove(item : Message) : Boolean` | 移除该消息的第一次出现。如果已移除则返回 `true`。 |
| `remove at(index : Number)` | 移除位置 `index` 处的消息 |
| `reverse` | 反转元素的顺序 |
| `set at(index : Number, value : Message)` | 设置第 i 条 `Message` |
| `set continuation(value : String)` | 设置下一组消息的标识符 |
| `sort by date` | 按从最新到最旧排序 |

## C.21 `Motion`

描述设备的运动状态。

| `acceleration : Vector3` | 获取设备的线性加速度，以重力单位计 |
| `gravity : Vector3` | 获取与此读数相关的重力向量 |
| `is invalid : Boolean` | 如果当前实例无效则返回 `true` |
| `pitch : Number` | 获取姿态的俯仰角，单位为度 |
| `post to wall` | 将运动读数显示到墙上 |
| `roll : Number` | 获取姿态的横滚角，单位为度 |
| `rotation speed : Vector3` | 获取设备的旋转速度，单位为度每秒 |
| `time : DateTime` | 获取一个时间戳，指示该读数被计算的时间 |
| `yaw : Number` | 获取姿态的偏航角，单位为度 |

## C.22 `Number`

一个数字（可能为负和/或小数）

| `-(right : Number) : Number` | 数字相减 |
| `*(right : Number) : Number` | 数字相乘 |
| `/(right : Number) : Number` | 数字相除 |
| `+(right : Number) : Number` | 数字相加 |
| `<(right : Number) : Boolean` | 比较数字是否小于 |
| `=(right : Number) : Boolean` | 比较数字是否相等 |
| `≠(right : Number) : Boolean` | 比较数字是否不等 |
| `>(right : Number) : Boolean` | 比较数字是否大于 |
| `≤(right : Number) : Boolean` | 比较数字是否小于等于 |
| `≥(right : Number) : Boolean` | 比较数字是否大于等于 |
| `is invalid : Boolean` | 如果当前实例无效则返回 `true` |
| `post to wall` | 将数字打印到墙上 |
| `to character : String` | 将数字解释为 Unicode 值并将其转换为单个字符串 |
| `to color : Color` | 将数字解释为 ARGB（alpha、红、绿、蓝）颜色 |
| `to json : Json Object` | 将值转换为 json 数据结构 |
| `to string : String` | 将数字转换为字符串 |



## C.23 `数字集合`

数字的集合。

| `add(item : Number)` | 在集合末尾添加一个数字。 |
| :--- | :--- |
| `add many(items : Number Collection)` | 一次性添加多个数字。 |
| `at(index : Number) : Number` | 获取位置`index`处的数字。如果`index`超出范围，则返回`invalid`。 |
| `avg : Number` | 计算所有数值的平均值。 |
| `clear` | 清除所有数字。 |
| `contains(item : Number) : Boolean` | 指示集合是否包含该`item`。 |
| `count : Number` | 获取元素的数量。 |
| `index of(item : Number, start : Number) : Number` | 获取某个数字首次出现的索引。如果未找到或`start`超出范围，则返回`-1`。 |
| `insert at(index : Number, item : Number)` | 在位置`index`处插入一个浮点数。如果`index`超出范围，则不执行任何操作。 |
| `is invalid : Boolean` | 如果当前实例无效，则返回`true`。 |
| `max : Number` | 计算所有数值的最大值。 |
| `min : Number` | 计算所有数值的最小值。 |
| `post to wall` | 在墙上显示这些数字。 |
| `random : Number` | 从集合中获取一个随机元素。如果集合为空，则返回`invalid`。 |
| `remove(item : Number) : Boolean` | 移除某个数字的首次出现。如果已移除，则返回`true`。 |
| `remove at(index : Number)` | 移除位置`index`处的数字。 |
| `reverse` | 反转元素顺序。 |
| `set at(index : Number, item : Number)` | 在位置`index`处设置数字。如果`index`超出范围，则不执行任何操作。 |
| `sort` | 对该集合中的数字进行排序。 |
| `sum : Number` | 计算所有数值的总和。 |

## C.24 `数字映射`

数字到数字的映射。

| `at(index : Number) : Number` | 获取`index`处的元素。`Index`可以是任意浮点值。 |
| :--- | :--- |
| `avg : Number` | 计算所有数值的平均值。 |
| `clear` | 清除数字映射。 |
| `count : Number` | 获取元素的数量。 |
| `is invalid : Boolean` | 如果当前实例无效，则返回`true`。 |
| `max : Number` | 计算所有数值的最大值。 |
| `min : Number` | 计算所有数值的最小值。 |
| `post to wall` | 以折线图显示该映射；如果你希望更改能反映出来，后续需调用`'update on wall'`。 |
| `remove(index : Number)` | 移除给定`index`处的值。 |
| `set at(index : Number, value : Number)` | 在`index`处设置元素。`Index`可以是任意浮点值。 |
| `set many(numbers : Number Map)` | 一次性设置多个元素。 |
| `slice(start : Number, end : Number) : Number Map` | 提取索引在`start`（包含）和`end`（不包含）之间的元素。 |
| `sum : Number` | 计算所有数值的总和。 |
| `update on wall` | 更新该映射的任何显示。 |

## C.25 `OAuth 响应`

OAuth 2.0 访问令牌或错误。

| `access token : String` | 由授权服务器颁发的访问令牌。 |
| :--- | :--- |
| `error : String` | 一个 ASCII [USASCII] 错误代码。 |
| `error description : String` | （可选）一个人类可读的错误描述。 |
| `error uri : String` | （可选）一个 URI，指向包含错误信息的人类可读网页，用于向客户端开发者提供有关错误的更多信息。 |
| `expires in : Number` | （可选）访问令牌的生命周期（以秒为单位）。 |
| `is error : Boolean` | 指示此响应是否为错误。 |
| `is expiring(lookup : Number) : Boolean` | （可选）指示令牌是否可能在接下来的`lookup`秒内过期。 |
| `is invalid : Boolean` | 如果当前实例无效，则返回`true`。 |
| `others : String Map` | （可选）OAuth 2.0 规范未涵盖的其他键值对。 |
| `post to wall` | 显示该响应。 |
| `set at(index : Number, item : Number)` | 在位置`index`处设置数字。如果`index`超出范围，则不执行任何操作。 |
| `sort` | 对该集合中的数字进行排序。 |
| `scope : String` | （可选）如果与客户端请求的范围相同，则为可选；否则，表示第 3.3 节所述访问令牌的作用域。 |

## C.26 `页面`

墙上的一个页面。

| `equals(other : Page) : Boolean` | 获取一个值，指示该页面是否与`other`相等。 |
| :--- | :--- |
| `is invalid : Boolean` | 如果当前实例无效，则返回`true`。 |

## C.27 `页面按钮`

墙上的一个页面按钮。

| `equals(page button : Page) : Boolean` | 获取一个值，指示两个实例是否相等。 |
| :--- | :--- |
| `icon : String` | 获取图标名称。 |
| `is invalid : Boolean` | 如果当前实例无效，则返回`true`。 |
| `page : Page` | 获取承载此按钮的页面。 |
| `post to wall` | 将此按钮推送到墙上。 |
| `text : String` | 获取文本内容。 |

## C.28 `页面集合`

页面的集合。

| `at(index : Number) : Page` | 获取`index`处的页面。 |
| :--- | :--- |
| `count : Number` | 获取页面的数量。 |
| `is invalid : Boolean` | 如果当前实例无效，则返回`true`。 |
| `post to wall` | 将页面发布到墙上。 |



### C.29 图片（Picture）

一个 `Picture` 值代表一张可显示的图像。方法列表已分为获取方法（返回图片的单个属性）和其他方法。

| 图片类型的获取方法 |
| --- |
| `at(index : Number) : Color` | 获取给定线性索引处的像素颜色 |
| `count : Number` | 获取像素数量 |
| `date : DateTime` | 获取拍摄照片的日期和时间；如果有的话 |
| `height : Number` | 获取以像素为单位的高度 |
| `location : Location` | 获取拍摄照片的位置；如果有的话 |
| `pixel(left : Number, top : Number) : Color` | 获取一个像素的颜色 |
| `width : Number` | 获取以像素为单位的宽度 |

| 图片类型的其他方法 |
| --- |
| `blend(other : Picture, left : Number, top : Number, angle : Number, opacity : Number)` | 在指定位置写入另一张图片。不透明度范围从 0（透明）到 1（不透明）。 |
| `blend svg(markup : String, left : Number, top : Number, width : Number, height : Number, angle : Number)` | 在指定位置写入一份可缩放矢量图形（SVG）文档。默认情况下，当宽度或高度为负数时，此操作使用 SVG 文档中提供的视口大小。 |
| `brightness(factor : Number)` | 改变图片的亮度。系数在 [-1, 1] 范围内。 |
| `clear(color : Color)` | 将图片清除为指定颜色 |
| `clone : Picture` | 返回图像的副本 |
| `colorize(background : Color, foreground : Color, threshold : Number)` | 基于介于 0.0 和 1.0 之间的颜色阈值，使用背景色和前景色重新着色图片 |
| `contrast(factor : Number)` | 改变图片的对比度。系数在 [-1, 1] 范围内。 |
| `crop(left : Number, top : Number, width : Number, height : Number)` | 裁剪出一个子图像 |
| `desaturate` | 使图片变为灰色 |
| `draw ellipse(left : Number, top : Number, width : Number, height : Number, angle : Number, c : Color, thickness : Number)` | 使用给定颜色绘制椭圆边框 |
| `draw line(x1 : Number, y1 : Number, x2 : Number, y2 : Number, color : Color, thickness : Number)` | 在两点之间绘制一条线 |
| `draw rect(left : Number, top : Number, width : Number, height : Number, angle : Number, c : Color, thickness : Number)` | 使用给定颜色绘制矩形边框 |
| `draw text(left : Number, top : Number, text : String, font size : Number, angle : Number, color : Color)` | 使用给定颜色和字体大小绘制一些文本边框 |
| `fill ellipse(left : Number, top : Number, width : Number, height : Number, angle : Number, color : Color)` | 使用给定颜色填充椭圆 |
| `fill rect(left : Number, top : Number, width : Number, height : Number, angle : Number, color : Color)` | 使用给定颜色填充矩形 |
| `flip horizontal` | 水平翻转图片 |
| `flip vertical` | 垂直翻转图片 |
| `invert` | 反转红色、蓝色和绿色通道 |
| `is invalid : Boolean` | 如果当前实例无效则返回 true |
| `is panorama : Boolean` | 指示图片宽度是否大于其高度 |
| `post to wall` | 将图像显示到墙上；如果希望更改生效，之后需要调用 `update on wall` |
| `resize(width : Number, height : Number)` | 将图片调整为指定像素尺寸 |
| `save to library : String` | 将图片保存到“已保存图片”相册。返回文件名。 |
| `set pixel(left : Number, top : Number, color : Color)` | 设置给定像素处的像素颜色 |
| `share(where : String, message : String)` | 分享此消息（传入空字符串 '' 将从列表中选择） |
| `tint(color : Color)` | 将每个像素转换为灰色，并用给定颜色为其着色。 |
| `update on wall` | 刷新墙上的图片 |

### C.30 图片相册（Picture Album）

一个 `Picture Album` 值代表一个图片相册。

| | |
| --- | --- |
| `albums : Picture Albums` | 获取子相册 |
| `is invalid : Boolean` | 如果当前实例无效则返回 true |
| `name : String` | 获取相册名称 |
| `pictures : Pictures` | 获取图片 |
| `post to wall` | 将相册显示到墙上 |

### C.31 图片相册集合（Picture Albums）

图片相册的集合。

| | |
| --- | --- |
| `at(index : Number) : Picture Album` | 获取位置 `index` 处的项；如果索引越界则无效 |
| `count : Number` | 获取集合中的元素数量 |
| `is invalid : Boolean` | 如果当前实例无效则返回 true |
| `post to wall` | 将该值显示到墙上 |
| `random : Picture Album` | 获取随机项；如果集合为空则无效 |

### C.32 图片集合（Pictures）

图片的集合。

| | |
| --- | --- |
| `at(index : Number) : Picture` | 获取位置 `index` 处的项；如果索引越界则无效 |
| `count : Number` | 获取集合中的元素数量 |
| `find(name : String) : Number` | 按名称查找图片并返回索引。如果未找到则返回 -1。 |
| `full(index : Number) : Picture` | 获取第 i 张图片的全分辨率版本 |
| `is invalid : Boolean` | 如果当前实例无效则返回 true |
| `post to wall` | 将图片缩略图显示到墙上 |
| `random : Picture` | 获取随机项；如果集合为空则无效 |
| `thumbnail(index : Number) : Picture` | 获取第 i 张图片的缩略图 |

### C.33 地点（Place）

附加或使用某个命名地点的信息。方法列表已分为三个表格：访问 `Place` 属性的获取方法、更新或替换属性的设置方法，以及其他方法。

| 地点类型的获取方法 |
| --- |
| `category : String` | 获取地点的类别 |
| `id : String` | 获取此位置的标识符 |
| `link : String` | 获取与消息关联的链接 |
| `location : Location` | 获取地点的位置 |
| `name : String` | 获取地点的名称 |
| `picture link : String` | 获取图片的 URL |
| `source : String` | 获取此位置的来源（Facebook, TouchDevelop） |

| 地点类型的设置方法 |
| --- |
| `set category(category : String)` | 设置地点的类别 |
| `set id(id : String)` | 设置此位置的标识符 |
| `set link(url : String)` | 设置与消息关联的链接 |
| `set location(location : Location)` | 设置地点的位置 |
| `set name(name : String)` | 设置地点的名称 |
| `set picture link(url : String)` | 设置图片的 URL |
| `set source(source : String)` | 设置此位置的来源 |

| 地点类型的其他方法 |
| --- |
| `is invalid : Boolean` | 如果当前实例无效则返回 true |
| `post to wall : Nothing` | 将地点发布到墙上 |
| `to string : String` | 转换为字符串名称，加上纬度和经度 |

### C.34 地点集合（Place Collection）

地点的集合。

| | |
| --- | --- |
| `add(value : Place)` | 添加一个地点 |
| `add many(value : Place Collection)` | 一次性添加多个地点 |
| `at(index : Number) : Place` | 获取第 i 个地点 |
| `clear` | 清空集合 |
| `continuation : String` | 获取下一组消息的标识符 |
| `count : Number` | 获取元素数量 |
| `index of(item : Place, start : Number) : Number` | 获取 `item` 首次出现的索引。如果未找到或 `start` 超出范围则返回 -1。 |
| `insert at(index : Number, item : Place)` | 在位置 `index` 处插入一个地点。如果 `index` 超出范围则不执行任何操作。 |
| `is invalid : Boolean` | 如果当前实例无效则返回 `true` |
| `post to wall` | 将地点发布到墙上 |
| `random : Place` | 从集合中获取一个随机元素。如果集合为空则返回无效值。 |
| `remove(item : Place) : Boolean` | 移除首次出现的地点的。如果已移除则返回 `true`。 |
| `remove at(index : Number)` | 移除位置 `index` 处的地点 |
| `reverse` | 反转元素的顺序 |
| `set at(index : Number, value : Place)` | 设置第 i 个地点 |
| `set continuation(value : String)` | 设置下一组消息的标识符 |
| `sort by distance(loc : Location)` | 按与地点的距离排序 |



### C.35 播放列表

一个歌曲播放列表

| `duration : Number` | 获取持续时间（秒） |
| `is invalid : Boolean` | 如果当前实例无效则返回 `true` |
| `name : String` | 获取歌曲名称 |
| `play` | 播放播放列表中的歌曲 |
| `post to wall` | 在墙上显示播放列表 |
| `songs : Songs` | 获取歌曲 |

### C.36 播放列表集合

一个播放列表的集合

| `at(index : Number) : Playlist` | 获取第 i 个播放列表 |
| `count : Number` | 获取播放列表的数量 |
| `is invalid : Boolean` | 如果当前实例无效则返回 `true` |
| `post to wall` | 在墙上显示该值 |

### C.37 歌曲

一个歌曲

| `album : Song Album` | 获取包含该歌曲的专辑 |
| `artist : String` | 获取艺术家名称 |
| `duration : Number` | 获取持续时间（秒） |
| `genre : String` | 获取歌曲流派 |
| `is invalid : Boolean` | 如果当前实例无效则返回 `true` |
| `name : String` | 获取歌曲名称 |
| `play` | 播放歌曲 |
| `play count : Number` | 获取播放次数 |
| `post to wall` | 在墙上显示歌曲 |
| `protected : Boolean` | 获取一个指示歌曲是否受 DRM 保护的值 |
| `rating : Number` | 获取用户评分；如果未评分则为-1 |
| `track : Number` | 获取专辑中的曲目编号 |

### C.38 歌曲集合

一个歌曲的集合

| `at(index : Number) : Song` | 获取位置 `'index'` 处的项目；如果索引越界则无效 |
| `count : Number` | 获取集合中的元素数量 |
| `is invalid : Boolean` | 如果当前实例无效则返回 `true` |
| `play` | 播放歌曲 |
| `post to wall` | 在墙上显示歌曲 |
| `random : Song` | 获取随机项目；如果集合为空则无效 |

### C.39 歌曲专辑

一个歌曲专辑

| `art : Picture` | 获取专辑封面图片 |
| `artist : String` | 获取艺术家名称 |
| `duration : Number` | 获取持续时间（秒） |
| `genre : String` | 获取歌曲流派 |
| `has art : Boolean` | 表示专辑是否有封面 |
| `is invalid : Boolean` | 如果当前实例无效则返回 `true` |
| `name : String` | 获取专辑名称 |
| `play` | 播放专辑中的歌曲 |
| `post to wall` | 在墙上显示歌曲专辑 |
| `songs : Songs` | 获取歌曲 |
| `thumbnail : Picture` | 获取缩略图图片 |

### C.40 歌曲专辑集合

一个专辑的集合

| `at(index : Number) : Song Album` | 获取位置 `'index'` 处的项目；如果索引越界则无效 |
| `count : Number` | 获取集合中的元素数量 |
| `is invalid : Boolean` | 如果当前实例无效则返回 `true` |
| `post to wall` | 在墙上显示该值 |
| `random : Song Album` | 获取随机项目；如果集合为空则无效 |

### C.41 歌曲集合

一个歌曲的集合。

| `at(index : Number) : Song` | 获取位置 `'index'` 处的项目；如果索引越界则无效 |
| `count : Number` | 获取集合中的元素数量 |
| `is invalid : Boolean` | 如果当前实例无效则返回 `true` |
| `play` | 播放歌曲 |
| `post to wall` | 在墙上显示歌曲 |
| `random : Song` | 获取随机项目；如果集合为空则无效 |

### C.42 音效

一个音效

| `duration : Number` | 获取持续时间（秒） |
| `is invalid : Boolean` | 如果当前实例无效则返回 `true` |
| `pan : Number` | 获取声像，范围从 -1.0（最左）到 1.0（最右） |
| `pitch : Number` | 获取音高调整，范围从 -1（降低一个八度）到 1（升高一个八度） |
| `play` | 播放音效 |
| `play special(volume : Number, pitch : Number, pan : Number)` | 使用不同的音量（0 到 1）、音高（-1 到 1）和声像（-1 到 1）播放歌曲 |
| `post to wall` | 在墙上显示播放器 |
| `set pan(pan : Number)` | 设置声像，范围从 -1.0（最左）到 1.0（最右） |
| `set pitch(pitch : Number)` | 设置音高调整，范围从 -1（降低一个八度）到 1（升高一个八度） |
| `set volume(v : Number)` | 设置音量，从 0（静音）到 1（最大音量） |
| `volume : Number` | 获取音量，从 0（静音）到 1（最大音量） |

### C.43 精灵

精灵是一个图形对象，可以显示在 `Board` 实例上。 `Sprite` 类型的方法列表已分为获取方法（返回单个属性）、设置方法（分配或更新属性）和其他方法。

| `Sprite` 类型的获取方法 |
| --- |
| `acceleration x : Number` | 获取沿 x 轴的加速度，单位为像素/秒² |
| `acceleration y : Number` | 获取沿 y 轴的加速度，单位为像素/秒² |
| `angle : Number` | 获取精灵的角度，单位为度 |
| `angular speed : Number` | 获取旋转速度，单位为度/秒 |
| `color : Color` | 获取精灵颜色 |
| `elasticity : Number` | 获取精灵弹性，表示为每次弹跳速度保持比例（0-1） |
| `friction : Number` | 获取速度损失比例，介于 0 和 1 之间 |
| `height : Number` | 获取高度，单位为像素 |
| `location : Location` | 获取分配给精灵的地理位置 |
| `mass : Number` | 获取质量 |
| `opacity : Number` | 获取不透明度（0 为透明，1 为不透明） |
| `picture : Picture` | 获取图片精灵上的图片（如果是图片精灵） |
| `speed x : Number` | 获取沿 x 轴的速度，单位为像素/秒 |
| `speed y : Number` | 获取沿 y 轴的速度，单位为像素/秒 |
| `text : String` | 文本精灵上的文本（如果是文本精灵） |
| `width : Number` | 获取宽度，单位为像素 |
| `x : Number` | 获取 x 位置，单位为像素 |
| `y : Number` | 获取 y 位置，单位为像素 |
| `z index : Number` | 获取精灵的 z 索引 |

| `Sprite` 类型的设置方法 |
| --- |
| `set acceleration(vx : Number, vy : Number)` | 设置加速度，单位为像素/秒² |
| `set acceleration x(vx : Number)` | 设置 x 轴加速度，单位为像素/秒² |
| `set acceleration y(vy : Number)` | 设置 y 轴加速度，单位为像素/秒² |
| `set angle(angle : Number)` | 设置精灵的角度，单位为度 |
| `set angular speed(speed : Number)` | 设置旋转速度，单位为度/秒 |
| `set clip(left : Number, top : Number, width : Number, height : Number)` | 为图像精灵设置裁剪区域（如果是图像精灵） |
| `set elasticity(elasticity : Number)` | 设置精灵弹性，表示为每次弹跳速度保持比例（0-1） |
| `set friction(friction : Number)` | 设置摩擦力，表示为速度损失比例，介于 0 和 1 之间 |
| `set height(height : Number)` | 设置高度，单位为像素 |
| `set location(location : Location)` | 设置精灵的地理位置 |
| `set mass(mass : Number)` | 设置精灵质量 |
| `set opacity(opacity : Number)` | 设置精灵不透明度（0 为透明，1 为不透明） |
| `set picture(pic : Picture) : Nothing` | 更新图片精灵上的图片（如果是图片精灵） |
| `set pos(x : Number, y : Number)` | 设置位置，单位为像素 |
| `set speed(vx : Number, vy : Number)` | 设置速度，单位为像素/秒 |
| `set speed x(vx : Number)` | 设置 x 轴速度，单位为像素/秒 |
| `set speed y(vy : Number)` | 设置 y 轴速度，单位为像素/秒 |
| `set text(text : String)` | 更新文本精灵上的文本（如果是文本精灵） |
| `set width(width : Number)` | 设置宽度，单位为像素 |
| `set x(x : Number)` | 设置 x 位置，单位为像素 |
| `set y(y : Number) : Nothing` | 设置 y 位置，单位为像素 |
| `set z index(zindex : Number)` | 设置精灵的 z 索引 |
| `speed towards(other : Sprite, magnitude : Number)` | 将精灵速度方向设置为朝向另一个精灵，并给定大小 |



### 精灵类型的其他方法

| 方法 | 描述 |
| --- | --- |
| `delete` | 删除精灵 |
| `equals(other: Sprite): Boolean` | 判断是否为同一个精灵？ |
| `hide` | 隐藏精灵 |
| `isInvalid: Boolean` | 如果当前实例无效则返回 `true` |
| `isVisible: Boolean` | 如果精灵未隐藏则返回 `true` |
| `move(deltaX: Number, deltaY: Number)` | 移动精灵 |
| `moveClip(x: Number, y: Number)` | 移动裁剪区域，如有需要则环绕图像（如果是图像精灵） |
| `moveTowards(other: Sprite, fraction: Number)` | 向另一个精灵移动 |
| `overlapWith(sprites: SpriteSet): SpriteSet` | 返回给定集合中与当前精灵重叠的子集 |
| `overlapsWith(other: Sprite): Boolean` | 精灵之间是否重叠？ |
| `show` | 显示精灵 |

## C.44 精灵集合

精灵的集合。与其他集合类型不同，`SpriteSet` 不允许重复项，且集合中的项是有序的。

| 方法 | 描述 |
| --- | --- |
| `add(sprite: Sprite): Boolean` | 将精灵添加到集合。如果精灵不在集合中则返回 `true`。 |
| `addFrom(oldSet: SpriteSet, sprite: Sprite): Boolean` | 将精灵添加到新集合并从旧集合中移除。如果精灵在旧集合中且不在新集合中则返回 `true`。 |
| `at(index: Number): Sprite` | 返回给定索引处的精灵 |
| `clear` | 移除集合中的所有精灵 |
| `contains(sprite: Sprite): Boolean` | 如果精灵在集合中则返回 `true` |
| `count: Number` | 返回集合中精灵的数量 |
| `indexOf(sprite: Sprite): Number` | 返回精灵在此集合中的索引，如果不在集合中则返回 `-1` |
| `isInvalid: Boolean` | 如果当前实例无效则返回 `true` |
| `remove(sprite: Sprite): Boolean` | 从集合中移除精灵。如果精灵在集合中则返回 `true` |
| `removeFirst: Sprite` | 移除最先添加到集合中的精灵 |

## C.45 字符串

一段文本。

| 方法 | 描述 |
| --- | --- |
| `∥(right: String): String` | 拼接两段文本 |
| `at(index: Number): String` | 获取指定索引处的字符 |
| `compare(other: String): Number` | 比较两段文本 |
| `concat(other: String): String` | 拼接两段文本 |
| `contains(value: String): Boolean` | 返回一个值，指示是否包含第二个字符串 |
| `copyToClipboard` | 将文本存储到剪贴板 |
| `count: Number` | 返回字符数 |
| `endsWith(value: String): Boolean` | 确定结尾是否匹配指定字符串 |
| `equals(other: String): Boolean` | 检查两个字符串是否相同 |
| `indexOf(value: String, start: Number): Number` | 从给定位置开始查找，返回第一个匹配项的索引 |
| `insert(start: Number, value: String): String` | 在给定位置插入字符串 |
| `isEmpty: Boolean` | 指示字符串是否为空 |
| `isInvalid: Boolean` | 如果当前实例无效则返回 `true` |
| `isMatchRegex(pattern: String): Boolean` | 指示字符串是否匹配正则表达式 |
| `lastIndexOf(value: String, start: Number): Number` | 从给定位置开始查找，返回最后一个匹配项的索引 |
| `matches(pattern: String): StringCollection` | 获取与正则表达式（pattern）匹配的字符串 |
| `postToWall` | 在墙上显示字符串 |
| `remove(start: Number): String` | 从字符串中返回从给定索引开始的所有字符 |
| `replace(old: String, new: String): String` | 返回替换后的字符串 |
| `replaceRegex(pattern: String, replace: String): String` | 根据替换字符串替换正则表达式的每个匹配项 |
| `share(network: String)` | 分享字符串（电子邮件、短信、Facebook、社交网络或留空以从列表中选择） |
| `split(separator: String): StringCollection` | 返回一个字符串集合，其中包含当前字符串中由指定字符串分隔的子字符串 |
| `startsWith(value: String): Boolean` | 确定开头是否匹配指定字符串 |
| `substring(start: Number, length: Number): String` | 给定起始索引和长度返回子字符串 |
| `toBoolean: Boolean` | 将字符串解析为布尔值 |
| `toColor: Color` | 将字符串解析为颜色 |
| `toDatetime: DateTime` | 将字符串解析为日期和时间 |
| `toJson: JsonObject` | 将值转换为 JSON 数据结构 |
| `toLocation: Location` | 将字符串解析为地理坐标 |
| `toLowerCase: String` | 返回此字符串转换为小写的副本，使用当前区域设置的规则 |
| `toNumber: Number` | 将字符串解析为数字 |
| `toTime: Number` | 将字符串解析为时间（12:30:12）并返回秒数 |
| `toUnicode: Number` | 将单个字符字符串转换为其 Unicode 编码 |
| `toUpperCase: String` | 返回此字符串转换为大写的副本，使用当前区域设置的规则 |
| `trim(chars: String): String` | 从当前字符串中移除所有前导和尾随的指定字符集中的字符 |
| `trimEnd(chars: String): String` | 从当前字符串中移除所有尾随的指定字符集中的字符 |
| `trimStart(chars: String): String` | 从当前字符串中移除所有前导的指定字符集中的字符 |

## C.46 字符串集合

字符串的集合

| 方法 | 描述 |
| --- | --- |
| `add(item: String)` | 添加一个字符串 |
| `addMany(items: StringCollection)` | 一次添加多个字符串 |
| `at(index: Number): String` | 获取位置 index 处的字符串。如果索引超出范围则返回无效值。 |
| `clear` | 清空字符串 |
| `contains(item: String): Boolean` | 指示集合是否包含该项 |
| `count: Number` | 获取字符串数量 |
| `indexOf(item: String, start: Number): Number` | 获取第一个匹配项的索引。如果未找到或起始索引超出范围则返回 `-1`。 |
| `insertAt(index: Number, item: String): Nothing` | 在位置 index 处插入字符串。如果索引超出范围则不执行任何操作。 |
| `isInvalid: Boolean` | 如果当前实例无效则返回 `true` |
| `join(separator: String): String` | 将分隔符和项目拼接成一个字符串 |
| `postToWall` | 在墙上显示字符串 |
| `random: String` | 从集合中获取一个随机元素。如果集合为空则返回无效值。 |
| `remove(item: String): Boolean` | 移除第一个匹配的字符串。如果已移除则返回 `true`。 |
| `removeAt(index: Number): Nothing` | 移除位置 index 处的字符串 |
| `reverse` | 反转项目顺序 |
| `setAt(index: Number, item: String)` | 设置位置 index 处的字符串。如果索引超出范围则不执行任何操作。 |
| `sort` | 对此集合中的字符串进行排序 |

## C.47 字符串映射

从字符串到字符串的映射

| 方法 | 描述 |
| --- | --- |
| `at(key: String): String` | 获取指定键对应的值；如果未找到则返回无效值 |
| `clear` | 清空映射中的值 |
| `count: Number` | 获取映射中的元素数量 |
| `isInvalid: Boolean` | 如果当前实例无效则返回 `true` |
| `keys: StringCollection` | 获取映射中的键 |
| `postToWall` | 在表格中显示键值对列表 |
| `remove(key: String)` | 移除指定键对应的值 |
| `setAt(key: String, value: String): Nothing` | 设置指定键对应的值；如果未找到则返回无效值 |
| `setMany(other: StringMap): Nothing` | 一次设置多个元素 |



## C.48 `TextBox`

文本框

| `background` : `Color` | 获取背景颜色 |
| `border` : `Color` | 获取边框颜色 |
| `font size` : `Number` | 获取字体大小 |
| `foreground` : `Color` | 获取前景颜色 |
| `icon` : `Picture` | 获取图标图片（最大 173x173） |
| `is invalid` : `Boolean` | 如果当前实例无效则返回 true |
| `post to wall` | 将文本框发布到墙上 |
| `set background(color : Color)` | 设置背景颜色 |
| `set border(color : Color)` | 设置边框颜色 |
| `set font size(size: Number)` | 设置字体大小（small = 14, normal = 15, medium = 17, medium large = 19, large = 24, extra large = 32, extra extra large = 54, huge = 140） |
| `set foreground(color : Color)` | 设置前景颜色 |
| `set icon(pic : Picture)` | 设置图标图片（最大 96 x 96） |
| `set text(text : String)` | 设置文本 |
| `text` : `String` | 获取文本 |

## C.49 `Vector3`

三维向量

| `add(other : Vector3) : Vector3` | 添加一个向量 |
| `clamp(min : Vector3, max : Vector3) : Vector3` | 将向量限制在指定范围内 |
| `cross(other : Vector3) : Vector3` | 计算与另一个向量的叉积 |
| `distance(other : Vector3) : Number` | 获取两个向量之间的距离 |
| `is invalid` : `Boolean` | 如果当前实例无效则返回 true |
| `length` : `Number` | 获取向量的长度 |
| `linear interpolation(other : Vector3, amount : Number) : Vector3` | 两个向量之间的线性插值 |
| `multiply(other : Vector3) : Vector3` | 与一个向量按分量相乘 |
| `negate` : `Vector3` | 返回一个指向相反方向的向量 |
| `normalize` : `Vector3` | 返回一个单位向量，方向与原向量相同 |
| `post to wall` | 在墙上显示向量 |
| `scale(scalar : Number) : Vector3` | 乘以一个缩放因子 |
| `subtract(other : Vector3) : Vector3` | 减去另一个向量 |
| `to string` : `String` | 将向量转换为字符串 |
| `x` : `Number` | 获取 x 分量 |
| `y` : `Number` | 获取 y 分量 |
| `z` : `Number` | 获取 z 分量 |

## C.50 `Web Request`

HTTP 网络请求

| `equals(other : Web Request) : Boolean` | 指示两个请求是否为同一实例 |
| `header(name : String) : String` | 获取指定标头的值 |
| `header names` : `String Collection` | 获取标头名称 |
| `is invalid` : `Boolean` | 如果当前实例无效则返回 true |
| `method` : `String` | 确定请求是 'get' 还是 'post' |
| `on response received(handler : Web Response Action)` | 设置从 'send async' 收到响应时要执行的操作 |
| `post to wall` | 在墙上显示请求 |
| `send` : `Web Response` | 同步执行请求 |
| `set accept(type : String)` | 设置 Accept 标头类型（'text/xml' 用于 XML，'application/json' 用于 JSON） |
| `set compress(value : Boolean)` | 使用 gzip 压缩请求内容并设置 Content-Encoding 标头 |
| `set content(content : String)` | 设置 'post' 请求的内容 |
| `set content as form(form : Form Builder)` | 将内容设置为 multipart/form-data |
| `set content as json(json : Json Object)` | 将 'post' 请求的内容设置为 JSON 树 |
| `set content as picture(picture : Picture, quality : Number)` | 将 'post' 请求的内容设置为 JPEG 编码图像。质量从 0（最差）到 1（最好）。 |
| `set content as xml(xml : Xml Object)` | 将 'post' 请求的内容设置为 XML 树 |
| `set credentials(name : String, password : String)` | 为基本身份验证设置名称和密码。需要 HTTPS URL，空字符串则清除。 |
| `set header(name : String, value : String)` | 设置 HTML 标头值。空字符串则清除该值。 |
| `set method(method : String)` | 将方法设置为 'get' 或 'post'。默认值为 'get'。 |
| `set url(url : String)` | 设置请求的 URL。必须是有效的互联网地址。 |
| `url` : `String` | 获取请求的 URL |

## C.51 `Web Response`

HTTP 网络响应

| `content` : `String` | 将响应正文读取为字符串 |
| `content as json` : `Json Object` | 将响应正文读取为 JSON 树 |
| `content as picture` : `Picture` | 将响应正文读取为图片 |
| `content as sound` : `Sound` | 将响应正文读取为音频 |
| `content as xml` : `Xml Object` | 将响应正文读取为 XML 树 |
| `header(name : String) : String` | 获取指定标头的值 |
| `header names` : `String Collection` | 获取标头名称 |
| `is invalid` : `Boolean` | 如果当前实例无效则返回 true |
| `post to wall` | 在墙上显示响应 |
| `request` : `Web Request` | 获取与此响应关联的请求 |
| `status code` : `Number` | 获取请求的 HTTP 状态码（如果有） |

## C.52 `Xml Object`

一个 XML 元素或元素集合。

| `at(index : Number) : Xml Object` | 获取集合中第 i 个子元素 |
| `attr(name : String) : String` | 获取属性的值 |
| `attr names` : `String Collection` | 获取属性名称列表 |
| `child(name : String) : Xml Object` | 获取第一个与完全限定名称匹配的子元素 |
| `children(name : String) : Xml Object` | 获取与完全限定名称匹配的子元素集合 |
| `count` : `Number` | 获取子元素的数量 |
| `create name(local name : String, namespace uri : String) : String` | 从命名空间和本地名称创建限定的完整名称 |
| `is element` : `Boolean` | 指示此实例是一个元素还是一个过滤后的集合 |
| `is invalid` : `Boolean` | 如果当前实例无效则返回 true |
| `local name` : `String` | 获取此元素的本地名称 |
| `name` : `String` | 获取此元素的全名 |
| `namespace` : `String` | 获取此元素的命名空间 |
| `post to wall` | 在墙上显示 XML 内容 |
| `to string` : `String` | 获取 XML 字符串 |
| `value` : `String` | 获取此元素的连接文本内容 |

![知识共享](https://creativecommons.org/licenses/by-nc-nd/4.0) 开放获取 本章采用知识共享署名-非商业性使用-禁止演绎 4.0 国际许可协议 ([`​creativecommons.​org/​licenses/​by-nc-nd/​4.​0/​`](http://creativecommons.org/licenses/by-nc-nd/4.0/) ) 进行许可，该协议允许在任何介质或格式中进行任何非商业性使用、共享、分发和复制，前提是您适当注明原始作者和来源，提供指向知识共享许可协议的链接，并指明您是否修改了许可材料。根据本许可协议，您无权分享基于本章或其部分内容改编的材料。本章中的图片或其他第三方材料已包含在本章的知识共享许可协议中，除非在材料的致谢行中另有说明。如果材料未包含在本章的知识共享许可协议中，并且您的预期使用不受法定法规允许或超出允许范围，您需要直接向版权所有者获取许可。

