# A Large-Scale Measurement of Website Login Policies（32nd USENIX）

 <center>Suood Al Roomi, *Georgia Institute of Technology, Kuwait University;* Frank Li, *Georgia Institute of Technology*</center>

![image-20240124132504870](D:\Desktop\github.io\files\workingMD\paper1.assets\image-20240124132504870.png)

## 目的与结果

寻求一个更好的能够对真实的网站登陆策略更加了解的口令策略自动调查工具，提供比以前研究大到两到三个数量级的群体特征。

## 思路

### 网站登陆政策

研究将登录分为六个阶段

- `Login Page visit` 页面是否会通过https来提供资源服务
- `Account Credential Entry`口令框内是否支持复制粘贴。有一些论文建议允许（复杂密码的可用性+口令管理器）
- `Password Transmission`提交登录表单之后，ID+password是否通过TLS安全信道传输的。如果不是，安全凭证可能会受损
- `Password Storage/Retrival`口令在服务器是否以明文（或者是其他的可逆的形式）存储？
- `Password Validation`主要是去看，用户提供的口令Password@a和存储的口令Password@a匹配的情况。Typo-tolerant的口令策略是说是否允许常见的错误，如a不小心碰到s；不小心reversed或者是不小心遗漏某些字母等情况。打字容忍性需要在口令可用性和安全性之间trade off
- `Login Failures`①有些登陆失败的反馈信息“密码错误”可以让攻击者进行用户名枚举，因此最好不要这样子②拉入黑名单“一个小时内不允许登录”，防止暴力猜测攻击。

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

集成的决策树分类器以识别

### 账户验证