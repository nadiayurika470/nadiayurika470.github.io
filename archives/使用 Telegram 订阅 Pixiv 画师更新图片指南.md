对于热爱插画艺术的朋友来说，及时获取 Pixiv 上关注画师的最新作品是一件重要的事情。本文将介绍如何使用 Node.js 开发一个自动化工具，通过 Telegram 机器人实时推送画师新作，让您不再错过任何精彩作品。

**技术方案概述：**

本项目使用 Node.js 开发，主要实现以下功能：
- 通过 RSS 采集 Pixiv 更新数据
- 处理图片链接和预览图
- 使用 Telegram Bot API 推送消息
- 自动化部署和运行

### 第一步：RSS 数据采集

使用 `rss-parser` 组件进行 RSS 内容采集，避免手动解析 XML 文件：

javascript
const Parser = require('rss-parser');
const parser = new Parser();

parser.parseURL(feedUrl, function(err, feed) {
    if (err) throw err;
    // 处理 feed 数据
});


### 第二步：数据处理

处理 RSSHub 生成的内容，提取图片信息：

javascript
// 匹配图片ID的正则表达式
const picIdReg = /https:\/\/pixiv\.cat\/(\d+)-?(\d+)?\.(jpg|png|gif)/gi;

// 生成内容数组
const artworks = [...item.content.matchAll(picIdReg)];


### 第三步：消息发送

使用 Telegram Bot API 发送图片消息：

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
            inline_keyboard: [
                [{
                    text: '🌏',
                    url: picItem.url
                }, {
                    text: '⤵',
                    url: picItem.pic
                }]
            ]
        }
    }
});


### 项目部署

使用 PM2 进行项目部署和管理：

bash
pm2 start bot.js --name phandream


**使用注意事项：**

1. Bot Token 获取：
   - 联系 [@BotFather](https://t.me/BotFather) 申请
   - 妥善保存 Token 信息

2. 图片处理：
   - 使用预览图发送消息
   - 提供原图下载链接
   - 注意 Telegram 的文件大小限制

3. 性能优化：
   - 使用时间戳避免重复推送
   - 添加请求队列管理
   - 合理设置更新间隔

通过以上步骤，您可以搭建一个稳定可靠的 Pixiv 画师更新推送系统。这个自动化解决方案不仅节省了手动检查的时间，还能确保您不会错过任何喜欢的画师的新作品。

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)