尽管我早已配置了 Twikoo 评论系统，但一直没有启用邮件通知功能。今天在检查之前文章时，意外发现了用户一个月前的留言 🤦 虽然我已经进行了回复，但可能对他没有帮助。因此，我决定添加评论通知功能，以防将来再发生类似的问题。

## 操作步骤

### 一、注册 Mailgun 账号并绑定域名

Mailgun 的官方网址是 [Mailgun](https://www.mailgun.com/)。在查看价格时，你会发现 Mailgun 提供每日 100 封邮件的免费发信额度，这对于个人博客来说非常充足。

需要注意的是，使用免费额度需要绑定信用卡。如果你没有信用卡，可以在这里创建 [野卡](https://bit.ly/bewildcard)，我推荐使用虚拟信用卡，因为它能有效控制限额，避免潜在的损失。

注册完账号后，我们需要绑定域名。在这里，我绑定的是我的博客域名 `senjianlu.com`。具体操作步骤如下：

1. 进入 Mailgun 控制台，选择自定义域名。
2. 按照给定的 DNS 信息进行验证。
3. 点击右上角的 `Verify` 按钮，等待验证通过。

### 二、获取 Mailgun 的 SMTP 信息

你的域名 SMTP 信息位于 `Send` -> `Sending` -> `Domains` 中。进入相关页面后，点击 `SMTP credentials`，即可查看 SMTP 相关信息。默认情况下，Mailgun 会创建一个 `postmaster` 用户。通过点击 `Reset Password` 重置并保存密码，即可完成 SMTP 信息的配置。

接下来，我们使用 Python 编写一个测试邮件功能的示例：

python
import smtplib
from email.mime.text import MIMEText
from email.header import Header

# 邮件内容
msg = MIMEText('Hello, this is a test email.', 'plain', 'utf-8')
msg['Subject'] = Header('这是一封内部测试邮件', 'utf-8')

msg['From'] = 'postmaster@senjianlu.com'
msg['To'] = '测试邮件接收者'

# 发信方信息
smtp_server = 'smtp.mailgun.org'
smtp_port = 587
smtp_user = 'postmaster@senjianlu.com'
smtp_password = '你的SMTP密码'

# 收信方信息
to_addrs = ['接收者邮箱1', '接收者邮箱2']

# 发信
try:
    server = smtplib.SMTP(smtp_server, smtp_port)
    server.starttls()
    server.login(smtp_user, smtp_password)
    server.sendmail(smtp_user, to_addrs, msg.as_string())
    print('邮件发送成功。')
except Exception as e:
    print('发送邮件失败:', e)
finally:
    server.quit()


> 我们在这一步主要用于确认 Mailgun 的 SMTP 信息是否正确。

确保信息无误并成功接收到邮件后，便可进入下一步。

### 三、配置 Twikoo 评论系统的邮件通知功能

进入博客评论模块，点击右上角的设置图标，选择配置管理并设置邮件通知。完成后，保存设置，然后可以进行邮件发送测试。

### 四、测试邮件通知功能

#### 1. 配置管理员邮箱

当有用户评论时，管理员邮箱将收到邮件通知。

#### 2. 测试用户发布评论

确保用户能够成功发表评论，并且管理员邮箱能够接收到该评论的通知邮件。

#### 3. 测试回复评论

回复评论后，确保用户邮箱也能顺利收到邮件。

### 五、设置 Mailgun 的发信上限

在 Mailgun 控制台中，选择 `Manage Account`，进入账户详情界面。设置 `Custom Message Limit`，此值代表单月的发信上限，可以根据需求进行调整。最小值为 1000，我设置为 1000。

---

通过以上步骤，你就可以成功配置 Twikoo 评论系统的邮件通知功能，使用 Mailgun 的免费发信额度，帮助你及时回复用户评论。

👉 [野卡 | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)