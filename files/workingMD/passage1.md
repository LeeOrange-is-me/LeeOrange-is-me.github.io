# A Large-Scale Measurement of Website Login Policies（32nd USENIX）

 <center>Suood Al Roomi, *Georgia Institute of Technology, Kuwait University;* Frank Li, *Georgia Institute of Technology*</center>

## 目的与结果

寻求一个更好的能够对真实的网站登陆策略更加了解的口令策略自动调查工具，提供比以前研究大到两到三个数量级的群体特征。

## 思路

### 网站登陆政策

研究将登录分为六个阶段

- `Login Page visit` 页面是否会通过https来提供资源服务。
  - 方法：同时HTTP和HTTPS进行抓取，记录HTTP的错误或重定向。
- `Account Credential Entry`口令框内是否支持复制粘贴。有一些论文建议允许（复杂密码的可用性+口令管理器）
  - 方法：seleniumAPI复制，然后检查字段值里头有没有我们复制的内容
- `Password Transmission`提交登录表单之后，ID+password是否通过TLS安全信道传输的。如果不是，安全凭证可能会受损
  - 方法：基于action的属性查看是否通过HTTP来传输
- `Password Storage/Retrival`口令在服务器是否以明文（或者是其他的可逆的形式）存储？
  - 方法：或许可以检测重置密码后通过邮箱传递的明文？
- `Password Validation`主要是去看，用户提供的口令Password@a和存储的口令Password@a匹配的情况。Typo-tolerant的口令策略是说是否允许常见的错误，如a不小心碰到s；不小心reversed或者是不小心遗漏某些字母等情况。打字容忍性需要在口令可用性和安全性之间trade off
  - 方法：测试typo-tolerant的口令。综合考虑大写锁定和shift的情况（win和mac不同：大写锁定时候win下shift会开启小写）
  - 呜呜呜，可能会要间隔一个小时才能进行下一次的测试
  - ![image-20240124145919735](D:\Desktop\github.io\files\workingMD\passage1.assets\image-20240124145919735.png)
- `Login Failures`①有些登陆失败的反馈信息“密码错误”可以让攻击者进行用户名枚举，因此最好不要这样子②拉入黑名单“一个小时内不允许登录”，防止暴力猜测攻击。
  - 方法
    - 两次测试：一个wrong ID 一次wrong pw
    - 从HTML当中提取错误信息，1.移除html标签2.翻译非英文的消息，使用Python-langdetect来检测英文消息3.替换id、email、password
    - 登录错误的速率：15次连续错误登录

### 整体工作

## 工作流程

### 检测表单

SVM二进制分类器（html的{title、ID、class，action}、密码的数量、表单输入框的数量）Hyper-parameters网格搜索

### 发现页面

- landing page 搜索登录、注册、找回密码的表单
- 在登录页面上爬取url链接（TF-IDF）
- 谷歌搜索引擎查找

### 尝试登录、注册、找回密码

- 识别表单并以可接受的值填写
- 识别Captcha并AZCaptcha自动化解决

### 识别是否登录成功或注册成功

集成的决策树分类器以识别是否操作成功

注册成功分类器是XGBoost决策树（网格搜索+4层交叉验证）

登录成功分类器是50棵树的XGBOOSt决策树

### 账户验证

注册时候需要验证码以验证是用户自己的邮箱

建立自己的邮箱服务器和邮箱域名，能够更好地掌控收件情况

作者只弄了电子邮件的验证，其他的（如手机号）就没弄

### 健全性检查

为了避免假阳性（认为在网站上有一个可控账户但是并没有）或假阴性（没有成功识别并注册账号），注册一个账号后要进行两次检查：

- 正确凭证登录
- 错误凭证登录