作为一名插画爱好者，保持与自己喜欢的 pixiv 画师的更新同步是一种重要的生活习惯。然而，每天手动检查新图既费时又费力，是否有更便捷的方法来推送这些喜爱的图片，让每一张图都有机会展现光彩呢？

联想到一种常用的资讯订阅方式：RSS，我在 RSSHub 的文档中找到了一种针对 pixiv 关注画师新图的订阅接口。这种方法虽然方便，但光靠 RSS 订阅还是不够，还需要一个独立的阅读器。为了简化流程，我曾尝试过基于 IFTTT 的解决方案，将 RSSHub 生成的内容推送至 Telegram 频道。然而，随着 IFTTT 的使用政策改变，非付费用户只能创建三个小程序，我决定放弃这个方案。

作为一个开发者，我不想再受制于第三方平台，因此我决定自己编写一个功能类似的工具，这不仅可以锻炼我的代码能力，还能提供更高的自定义程度。

我开始查询各种 API 接口，整合资料。我的思路很简单：通过 RSS 采集数据，再通过 Bot 的 API 接口将其发送至 Telegram 频道。由于我习惯使用 Node.js 进行开发，所以这次也一样选择了 Node.js 作为开发与部署平台。

## 第一步：RSS 采集

首先，进行 RSS 的采集工作。为了避免费时费力地手动分析 XML 文件，我直接使用了一个现成的组件：`rss-parser`。这个组件可以获取 RSS 资讯，并将其转化为方便后续处理的对象。

bash
npm i rss-parser --save


使用后，参数调用的方式也非常简洁。尽管可以使用官方样例中的 async 与 await 处理方式，但出于方便，我选择了直接传递参数来处理。

测试时，可以将获取的 feed 信息输出。以 RSSHub 生成的数据为例，我们可以发现其内部是一个对象数组，因此可以直接使用相关字段进行内容处理。

## 第二步：数据处理

由于 RSSHub 是为了方便阅读而设计的，因此生成的内容格式主要以用户阅读为主。而获取的内容中包含了我们所需的所有信息，因此可以使用正则表达式进行提取。

对于单张图片的内容，RSSHub 将其处理成形如 `https://pixiv.cat/*ArtworkID*.jpg` 的链接；目前，pixiv 的 ArtworkID 仅为数字，因此可以使用 `\d` 来匹配。而对于多图内容，ArtworkID 后的 `-PicID` 也是我们关注的重点。我们可以使用如下的正则表达式进行处理：

javascript
const picIdReg = /https:\/\/pixiv\.cat\/(\d+)-?(\d+)?\.(jpg|png|gif)/gi;


结合这条正则表达式，可以直接生成内容数组：

javascript
const artworks = [...item.content.matchAll(picIdReg)];


对于单图的输入，我们能得到形如这样的输出：


> Array ["https://pixiv.cat/*ArtworkID*-1.jpg", "*ArtworkID*", "1", "jpg"]
> Array ["https://pixiv.cat/*ArtworkID*-2.jpg", "*ArtworkID*", "2", "jpg"]


考虑到 Telegram 对于图片大小的限制，我们选择使用预览图发送，并通过完整图下载的方案。

通过查看 pixiv 前端页面的源码，发现预览图的地址由前缀、发布时间（UTC+9）、作品ID和一些固定组合搭配而成。发布时间可以由 RSS item 中的 isoDate 手动计算获取，我们可以写一个整合表达式来完成预览图地址的拼接工作。

## 第三步：消息发送

之前使用 IFTTT 的时候直接推送订阅内容，让 Telegram 自动获取，但导致大图片（≥5MB）无法有效获取、图片缓存请求限制等问题。因此这次我们加入了请求缓冲队列和预览图片，尝试解决这些问题。

本来考虑使用 Telegraf 作为推送框架，但发现请求信息也就一条，因此直接使用 got 进行 POST 请求的发送。根据 Telegram 的 Bot API 文档，可以整合出如下的请求样式：

javascript
const apiBaseUrl = `https://api.telegram.org/bot${confData.bot.token}`;

got('sendPhoto', {
    method: 'POST',
    prefixUrl: apiBaseUrl,
    json: {
        chat_id: confData.bot.chat,
        photo: picItem.preview,
        caption: picItem.text,
        reply_markup: {
            inline_keyboard: [[
                {
                    text: '🌏',
                    url: picItem.url
                },
                {
                    text: '⤵',
                    url: picItem.pic
                }
            ]]
        }
    }
});


直接调用 Telegram 的 `sendPhoto` 接口来发送请求数据，其中加入了消息下的内联小键盘 `inline_keyboard`，提供图片的原始链接和直接下载按钮，方便用户操作。同时， Bot API Token 可以向 [BotFather](https://t.me/BotFather) 申请，Chat ID 可以使用 `@频道名`，避免纠结于一长串的频道ID。

为了避免每次启动都将所有图片加入队列等待发送，并考虑到数据按时间排序，因此维护一个 timestamp，初始化为当前时间，每次发送后更新该时间戳，从而只发送新内容。

为进一步便捷用户，我加入了 js-yaml 组件来读取 yml 配置文件。最终整合的版本已发布至 GitHub 的 [PhanDream](https://github.com/Candinya/PhanDream) 仓库中；您可以使用如 pm2 等工具进行运行：

bash
pm2 start bot.js --name phandream


当一切准备完成后，稍等片刻，您就能在频道中看到新的图片了！

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)