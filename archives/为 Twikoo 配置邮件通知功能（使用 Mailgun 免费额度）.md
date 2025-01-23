虽然早早配置了 Twikoo 评论系统，但一直没有配置邮件通知功能。今天检查文章时偶然发现了用户一个月前的留言，虽然回复了但恐怕帮不到他了。为了避免类似问题，再次发生，决定加个评论通知功能。

## 操作步骤

### 一、注册 Mailgun 账号并绑定域名

Mailgun 提供了免费的每天 100 封邮件的发信额度，非常适合个人博客使用。

👉 [WildCard | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)

注册 Mailgun 后，绑定你的域名。例如绑定博客域名 `example.com`，按照 Mailgun 提供的 DNS 信息完成验证即可。

### 二、获取 Mailgun 的 SMTP 信息

在 Mailgun 控制台中，进入 `Send -> Sending -> Domains`，选择已验证的域名，点击 `SMTP credentials` 查看 SMTP 信息。默认会创建一个 `postmaster` 用户，重置密码后即可获取完整的 SMTP 信息。

以下是一个简单的 Python 示例，用于测试 SMTP 是否正常：

python
import smtplib
from email.mime.text import MIMEText
from email.header import Header

# 邮件内容
msg = MIMEText('Hello, this is a test email.', 'plain', 'utf-8')
msg['Subject'] = Header('这是一封测试邮件', 'utf-8')
msg['From'] = '[email protected]'
msg['To'] = '[email protected]'

# SMTP 信息
smtp_server = 'smtp.mailgun.org'
smtp_port = 587
smtp_user = '[email protected]'
smtp_password = 'your_password'

# 收信方
to_addrs = ['[email protected]', '[email protected]']

# 发信
try:
    server = smtplib.SMTP(smtp_server, smtp_port)
    server.starttls()
    server.login(smtp_user, smtp_password)
    server.sendmail(smtp_user, to_addrs, msg.as_string())
    print('邮件发送成功')
except Exception as e:
    print('邮件发送失败:', e)
finally:
    server.quit()


运行脚本后，确认邮件是否发送成功。

### 三、配置 Twikoo 的邮件通知功能

1. 进入 Twikoo 评论模块，点击右上角的设置图标。
2. 选择配置管理，找到邮件通知设置。
3. 填写 SMTP 信息，包括服务器地址、端口、用户名和密码。
4. 保存配置后，发送测试邮件，确认功能正常。

### 四、测试邮件通知功能

#### 1. 配置管理员邮箱

当用户评论时，管理员邮箱会收到通知邮件。

#### 2. 测试用户评论

用户发表评论后，管理员邮箱会收到通知。

#### 3. 测试回复评论

管理员回复评论后，用户邮箱会收到通知。

### 五、设置 Mailgun 的发信上限

在 Mailgun 控制台中，进入 `Manage Account`，设置 `Custom Message Limit`。根据需求设置单月发信上限，最小值为 1000。

---

👉 [WildCard | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)