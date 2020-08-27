# KokkoroBot
KokkoroBot is a forked version of **HoshinoBot** and migrates from QQ to other platforms.

本项目是 HoshinoBot 的分支版本，充斥着大量个人魔改产物。若希望体验原汁原味的 HoshinoBot 请自行魔改回去。

web界面敬请期待。

目前已适配以下平台
- Tomon(国内平台)
    - 体验服务器：https://beta.tomon.co/invite/kCA4Qg?t=htMviz
- Discord
    - 体验服务器：https://discord.gg/U9XFCVj
- Telegram
- 企业微信


## 1. Motivation
HoshinoBot 无论是基础设施还是应用层，大量使用了来自 Nonebot 中的概念如 `CQEvent`, `MessageSegment`，这直接导致上层服务与 Nonebot 以及 Nonebot 底层的 CQHttp 极大程度上耦合在一起，难以将 HoshinoBot 移植到其他平台或框架，比如从 QQ 迁移至 Discord。

因此 KokkoroBot 针对这一问题对底层代码进行重构，期望应用服务与 IM 平台解耦合，能够较迅速的迁移至其他平台。~~其实是因为酷Q凉了，mirai随时可能会挂~~

## 2. 使用方法
### 2.1 配置文件
首先将 `kokkoro` 文件夹下的 `config_example` 在当前文件夹复制一份并重命名为 `config`。然后修改其中的各个配置文件

```sh
cd kokkoro
cp -r config_example config
```

#### 2.1.1 通用配置
配置文件为 `kokkoro/config/__bot__.py`
- `BOT_TYPE`
    - 将 `BOT_TYPE` 修改为你需要使用的 IM 平台，可填写的值有以下三种
    - `discord`
    - `telegram`
    - `wechat_enterprise` (企业微信)
- `BOT_ID`, `SUPER_USER`, `ENABLED_GROUP`
    - 这三个参数都是平台相关的。需要在对应平台的配置文件中进行修改
    - `BOT_ID`：bot 自己的 ID
        - 部分平台 bot 会收到自己的消息，因此使用该参数过滤 bot 自己的消息
    - `SUPER_USER`：在 bot 中拥有最高权限的用户，通常将自己的 ID 填在这里
    - `ENABLED_GROUP`：bot 仅会对 `ENABLED_GROUP` 中的群组作出回应
- `LOG_LEVE`
    - 开发人员 DEBUG 用，一般用户设置为 "INFO" 即可
- `ENABLE_IMAGE`
    - 是否允许 bot 发送图片
    - 第一次部署建议先设置为 `False`，成功后再改为 `True`
- `NICK_NAME`
    - bot 别称，通过在消息最前面加别称达到与 @bot 相同的效果
- `RES_PROTOCOL`
    - 加载内置资源文件的方式，可填写的值有以下两种：
    - `file` （通过文件访问）
    - `http` （通过 http 协议访问，需另外部署静态资源服务器）
- `RES_DIR`
    - 资源文件的根目录路径
    - 示例：`~/.kokkoro/res/`
- `RES_URL`
    - 资源文件的根 URL
    - 示例：`http://123.123.123.123/`
- `FONT_PATH`
    - 字体文件定义
    - 老用户需要直接从 `kokkoro/config_example/__bot__.py` 复制最新的内容过来覆盖以前的内容，新用户无需任何特殊操作。
- `MODULES_ON`
    - 开启的模块
    - 将 `kokkoro/modules/` 目录下想要开启的模块名称填写在这个列表中

#### 2.1.2 Discord 配置
配置文件为 `kokkoro/config/bot/discord.py`。

Discord 需要将客户端设置为开发者模式才能查看群组、用户 ID。开发者模式的开启请参考该链接 https://support.discord.com/hc/en-us/articles/206346498-Where-can-I-find-my-User-Server-Message-ID-

修改配置文件之前，先按照以下链接的说明在 Discord 网站上创建好 App，然后在 App 内创建好 Bot。

https://discord.com/developers/docs/intro

此外如果你还没有一个 Discord Server（群组），需要在 Discord 客户端中自行创建。

- `DISCORD_TOKEN`
    - 填写之前创建的 Bot 的 TOKEN
- `ENABLED_GROUP`
    - 允许使用 Bot 的 Discord Server ID
- `SUPER_USER`
    - 填写自己的用户 ID
- `BOT_ID`
    - 填写之前创建的 App 的 CLIENT ID（在 General Information 页面）
- `BROADCAST_CHANNEL`
    - 广播频道的名称
    - KokkoroBot 只会把推送消息发送到广播频道中。比如药水提醒、新闻推送。

#### 2.1.3 Telegram 配置
配置文件为 `kokkoro/config/bot/telegram.py`

Telegram 获取用户与群组 ID 很麻烦，需要从 API 中自己获取，请参考该链接 https://stackoverflow.com/questions/32423837/telegram-bot-how-to-get-a-group-chat-id

> 在修改配置文件之前，先在在 Telegram 客户端中找 BotFather 申请好 BOT，并使用 `/setprivacy` 命令将 bot 设置为允许接受来自群组的消息
>
> https://core.telegram.org/bots

- `TELEGRAM_TOKEN`
    - 填写从 BotFather 得到的 TOKEN
- `ENABLED_GROUP`
    - 允许使用 BOT 的 telegram 群组 ID
- `SUPER_USER`
    - 填写自己的 ID
- `BOT_ID`
    - 填写 BOT 的 ID
    - 目前 telegram 暂时没有一个比较好的手段获得 `BOT_ID`，不过 telegram bot 也不会收到自己发的消息，因此设置为 `None` 即可

#### 2.1.4 Tomon (国内平台)
配置文件为 `kokkoro/config/bot/tomon.py`

Tomon 获取用户与群组 ID 目前需要通过 API 获取，请参考链接 https://developer.tomon.co/docs/user#%E6%9F%A5%E7%9C%8B%E4%B8%AA%E4%BA%BA%E8%B5%84%E6%96%99

> 目前只能使用自己的账号作为 bot 部署，参考以下链接获取 Token。之后将允许为 bot 创建额外账号。
>
> https://developer.tomon.co/docs/auth

- `TOMON_TOKEN`
    - 填写从 Auth api 得到的 TOKEN
- `ENABLED_GROUP`
    - 允许使用 BOT 的 Tomon 群组 ID
- `SUPER_USER`
    - 填写自己的 ID
- `BOT_ID`
    - 填写 BOT 的 ID

#### 2.1.5 企业微信
TODO

### 2.2 部署流程
#### 2.2.1 Linux/macOS (推荐)
使用 Linux 部署可以通过 Docker 省去自行搭建环境的麻烦与各种平台问题。
- 安装 Docker 环境。

    https://docs.docker.com/engine/install/centos/

- 安装 Docker Compose。

    https://docs.docker.com/compose/install/

- 构建 Docker 镜像。

    ```sh
    cd deploy
    docker-compose build
    ```

- 使用 Docker Compose 部署。

    进行 debug 时请删除 `-d` 参数。

    ```sh
    # 确保处于 deploy 目录中
    docker-compose up -d
    ```

    也可以只启动 Bot，不运行 Web 界面。

    ```sh
    # 确保处于 deploy 目录中
    docker-compose up bot -d
    ```

- 在群里发一句 `help`，Bot 如果反馈帮助信息则说明初步搭建完成！

- 如果对配置文件或者代码有任何修改，需要重新执行以下命令才能生效。

    ```sh
    # 确保处于 deploy 目录中
    docker-compose build
    docker-compose up -d
    ```

#### 2.2.2 Windows
TODO

### 2.3 资源文件
目前 KokkoroBot 中的图片等资源文件基于 HoshinoBot 群内提供的资源包，额外添加了字体文件，因此只能从 KokkoroBot 群下载。

KokkoroBot 群：887897168。

下载好的资源文件放在服务器的 `~/.kokkoro` 目录下，保证目录看上去是如下结构 `~/.kokkoro/res/img`。

### 2.4 第三方应用的 API Key
TODO

## 4. 二次开发与新平台适配
详情见 `DEVELOPER-README.md` .这里只做简要介绍。
### 4.1 接口设计
KokkoroBot 在基础设施与应用层中加一层统一接口 `common_interface` 以达到解耦合的目的。

任何平台的机器人都需要实现 `kokkoro.common_interface.KokkoroBot` 接口。

任何平台的事件(`Event`)都需要转化为 `kokkoro.common_interface.EventInterface` 供上层服务使用。同理，任何平台的用户(`User`)都需要转化为 `kokkoro.common_interface.UserInterface`。

发送图片时不再依赖任何平台相关逻辑，直接将 `common_interface.SupportedMessageType` 格式的消息传给 `kokkoro.common_interface.KokkoroBot.kkr_send` 函数，在具体实现类中实现具体平台相关的处理逻辑。

上层服务与平台彻底解耦合。平台依赖于 `kokkoro.common_interface`，上层服务依赖于 `kokkoro.commmon_interface`。因此能够兼容多种平台。

- 在这样的设计下，将 KokkoroBot 移植到其他平台便不再困难。只需要以下三步：
    - 完成平台相关的机器人
        - 机器人需要实现 `kokkoro.common_interface.KokkoroBot` 接口
        - 需要支持 `common_interface.SupportedMessageType` 中所有消息类型的发送
        - 示例: `kokkoro.bot.discord.KokkroDiscordBot`
    - 实现 `Event` 的适配
        - 将平台相关 `Event` 适配为 `common_interface.EventInterface`
            - 比如 `CQEvent`, `discord.Message`
        - 示例：`kokkoro.bot.discord.discord_adaptor.DiscordEvent`
    - 实现 `User` 的适配
        - 将平台相关 `User` 适配为 `common_interface.UserInterface`
            - 比如 `CQEvent` 中的 at 信息，`discord.User`
        - 示例：`kokkoro.bot.discord.discord_adaptor.DiscordUser`

### 4.2 新 IM 平台适配
以 telegram 为例，从零对 telegram 平台进行开发主要包含以下几步。
- `kokkoro.config.__bot__` 中配置 `BOT_TYPE = 'telegram'`
- `kokkoro.config.bot` 中添加 `telegram.py` 配置文件
- 在 `kokkoro.bot.telegram` 包中实现相关适配
- 在 `kokkoro.bot.__init__` 中添加 telegram bot

# TODO
- [ ] Web
    - Doing by @SonodaHanami
- [x] Isolate platform specific logic from application services
- [x] Scheduler
- [ ] Modules migration
    - [ ] Arknights
    - [x] Pcr Clanbattle
    - [ ] Dice
    - [ ] Flac
    - [ ] Group Master
    - [ ] Priconne
        - [x] Arena
        - [x] Calender
        - [ ] Comic Spider
        - [x] Horse
        - [x] News
        - [x] Query
        - [x] Reminder
    - [ ] Setu
    - [ ] Weibo Spider
- [x] Multi-platform
    - [x] Discord
        - [ ] Audio
        - [x] Image
        - [x] Text
        - [x] Permission control with admin
    - [x] Telegram
        - [ ] Audio
        - [ ] Image
        - [x] Text
        - [ ] Permission control with admin
    - [x] Tomon
        - [ ] Audio
        - [ ] Image
        - [x] Text
        - [ ] Permission control with admin
    - [x] Wechat Enterprise
        - [ ] Audio
        - [ ] Image
        - [x] Text
        - [ ] Permission control with admin
