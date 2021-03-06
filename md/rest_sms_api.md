# 短信服务 REST API 详解

[REST API](/rest_api.html) 可以让您用任何可以发送 HTTP 请求的设备来与 LeanCloud 进行交互,您可以使用短信服务 REST API 做很多事情,比如:

* 给指定手机号码发送短信验证码.
* 验证手机号和短信验证码是否匹配。
* 使用手机号和验证码进行登录。
* 通过合法的手机号和验证码来重置账户密码。
* 进行重要操作（例如支付）的验证确认等。

LeanCloud 的短信服务目前对**国际**、**国内**都是支持的。对于每个 LeanCloud 账户，我们提供 100 条国内短信的免费额度，以供大家进行简单测试。超过之后的部分是实时从短信账户进行扣费的，请务必保证短信账户余额充足。具体的价格请参看[这里](https://leancloud.cn/pricing.html)。

## 快速参考

所有的 API 访问都是通过 HTTPS 进行的. API 访问需要在 __https://leancloud.cn__ 域名下,相对路径前缀 __/1.1/__ 表明现在使用的是第 1.1 版的API。如需在线测试 API，请在浏览器中打开[https://leancloud.cn/apionline/](https://leancloud.cn/apionline/)。

LeanCloud 提供的与短信服务相关的 REST API 包括：


### 短信验证 API

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/requestSmsCode</td>
      <td>POST</td>
      <td>请求发送短信验证码</td>
    </tr>
        <tr>
      <td>/1.1/verifySmsCode/&lt;code&gt;</td>
      <td>POST</td>
      <td>验证短信验证码</td>
    </tr>
  </tbody>
</table>


### 用户

<table>
  <thead>
    <tr>
      <th>URL</th>
      <th>HTTP</th>
      <th>功能</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/1.1/usersByMobilePhone</td>
      <td>POST</td>
      <td>使用手机号码注册或登录</td>
    </tr>
    <tr>
      <td>/1.1/requestMobilePhoneVerify</td>
      <td>POST</td>
      <td>请求发送用户手机号码验证短信</td>
    </tr>
    <tr>
      <td>/1.1/verifyMobilePhone/&lt;code&gt;</td>
      <td>POST</td>
      <td>使用"验证码"验证用户手机号码</td>
    </tr>
    <tr>
      <td>/1.1/requestLoginSmsCode</td>
      <td>POST</td>
      <td>请求发送手机号码短信登录验证码</td>
    </tr>
    <tr>
      <td>/1.1/requestPasswordResetBySmsCode</td>
      <td>POST</td>
      <td>请求发送手机短信验证码重置用户密码</td>
    </tr>
    <tr>
      <td>/1.1/resetPasswordBySmsCode/&lt;code&gt;</td>
      <td>PUT</td>
      <td>验证手机短信验证码并重置密码。</td>
    </tr>
  </tbody>
</table>


### 请求和响应格式

请参考我们的 [REST API 总览文档](./rest_api.html#请求格式)。

### 验证码发送逻辑图

在短信验证码发送过程中，一共有三方参与：客户端、LeanCloud和电信运营商（移动、联通、电信），发送、验证短信验证码的过程如下图所示：

![image](images/leancloud_sms_sequence.png)

* 首先是应用客户端向 LeanCloud 请求往特定手机号码发送验证码；
* LeanCloud 云端收到请求后，生成验证码，然后将完整短信内容发送到运营商通道；
* 运营商下发短信（或语音）；
* 应用客户端收到验证码短信后，再向 LeanCloud 验证手机号码和短信验证码的合法性；

## 短信验证 API

在一些场景下，你可能希望用户验证手机号码后才能进行一些操作，例如充值等。这些操作跟账户系统没有关系，可以通过我们提供的的短信验证 API 来实现。

使用这些 API 需要在应用「控制台」->「设置」->「应用选项」->「短信」中开启`启用手机号码短信认证 （针对 requestSmsCode 和 verifySmsCode 接口）` 选项。如下图所示：

![image](images/leancloud_sms_setting1.png)

给某个手机号码发送验证短信，可通过如下请求完成：

```sh
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"mobilePhoneNumber": "186xxxxxxxx"}' \
  https://api.leancloud.cn/1.1/requestSmsCode
```

这里必须使用 POST 方式来发送请求，请求体里支持的参数有：

- mobilePhoneNumber：目标手机号码，必填。
- ttl：验证码有效时间，单位：分钟，选填，默认是 10 分钟。
- name：应用名字，选填，默认是 LeanCloud 控制台填写的应用名。
- op：操作类型，选填。

### 语音验证码

语音验证码，是通过电话直接呼叫用户的电话号码来播报验证码，它是一个 6 位的数字组合，语音只播报数字内容，不能添加其他任何内容。它可以作为一种备选方案，来解决因各种原因导致短信无法及时到达的问题。发送方式如下：

```sh
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"mobilePhoneNumber": "186xxxxxxxx", "smsType":"voice"}' \
  https://api.leancloud.cn/1.1/requestSmsCode
```

与上面的普通短信验证码相比，请求发送语音验证码的时候，要加上 `smsType` 这个请求参数，其值为 `voice` 。

`smsType` 允许的取值有：

 - `voice` 为发送语音验证码
 - `sms` 为发送普通短信验证码

此接口与之前的 [验证短信 API](#短信验证_API) 完全兼容，如果你不需要此服务，完全不需要修改之前的发送短信代码。

### 校验验证码

通过下面的 API 可以验证收到的 6 位数字验证码是否正确：

```sh
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  "https://api.leancloud.cn/1.1/verifySmsCode/6位数字验证码?mobilePhoneNumber=186xxxxxxxx"
```

其中 `verifySmsCode` 后面是手机收到的 6 位数字验证码。`mobilePhoneNumber` 是收到短信的手机号码。

**由于运营商和渠道的限制，短信验证码发送要求间隔至少一分钟，并且每天向同一手机号码发送次数不能超过 10 次，因此建议采用图形验证码、倒数计时等措施来控制频率，提示用户，防止短信轰炸等恶劣情况发生。**

### 国际短信
上面发送短信验证码和语音验证码，默认都是对国内号码。LeanCloud 现在已经开通了国际短信验证码服务（语音验证码海外还不可用）。要发送国际短信，只需要在发送 `https://api.leancloud.cn/1.1/requestSmsCode` 请求的时候，额外加上**countryCode**这一参数项即可。`countryCode`的取值范围请参考[countrycode.org](https://countrycode.org/)里 `ISO CODES` 一列，例如 `US` 表示美国，`CN` 代表中国。

下面的请求将给美国的手机号码("917646xxxxx")发送一条短信验证码：

```sh
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"mobilePhoneNumber": "917646xxxxx", "countryCode":"US"}' \
  https://api.leancloud.cn/1.1/requestSmsCode
```

除了增加的 `countryCode` 之外，发送国际短信和国内短信的请求参数都是一样的。

## 自定义短信模板

我们还支持通过 `requestSmsCode` 发送自定义模板的短信，但是**要求内容必须是验证码类或者通知类短信，不允许包含下载链接等推广信息**。模板的创建和修改都需要审核，并且要求创建或者修改的时候短信帐户至少有 200 元的非赠送余额。模板本身不扣费，短信发送才扣费。**通知类短信没有间隔和条数限制。**

短信模板可以在「应用控制台」->「消息」->「短信」->「设置」的「短信模板」里创建，创建后将自动提交审核，审核结果将通过邮件的形式发送到您的账号邮箱。每个应用限制创建 10 个模板，并且每个模板都需要经过审核才可以使用（审核在工作时间内通常在 1个小时内）。模板一经审核，就可以马上使用，

后续你可以创建同名模板来替换当前使用模板，新模板也同样需要审核。审核通过，即可替换老模板。

如果您创建了短信模板，可以指定 `template` 参数指定模板名称来使用您的模板，并且可以传入变量渲染模板，比如下面例子里的 `date`：

```sh
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"mobilePhoneNumber": "186xxxxxxxx", "template":"activity","date":"2014 年 10 月 31 号"}' \
  https://api.leancloud.cn/1.1/requestSmsCode
```

目前我们仅允许两类自定义短信：验证类短信和通知类短信，不允许发送推广营销类短信。

短信模板的语法遵循 [Handlebars](http://handlebarsjs.com/)，举例：

<pre ng-non-bindable ><code>
Hi {{username}},
欢迎注册{{name}}应用，您可以通过验证码:{{code}}，进行注册。本条短信将在{{ttl}}分钟后自行销毁。请尽快使用。
以上。
</code></pre>

其中：
* code 我们帮你生成的验证码，可以通过 `/1.1/verifySmsCode/:code` 校验。
* ttl  短信有效期，单位分钟，默认为 10 分钟。
* name 应用名称

这三个内置字段会自动填充，你当然也可以添加自定义变量，形如 `{{var}}`。

短信签名是指短信内容里 `【】` 括起来的短信发送方名称，如果没有明确在模板里指定，默认就是你的应用名称。**短信签名不能超过 10 个字符，应用名称可以在应用设置里修改，并且短信签名必须出现在短信内容的开头或者结尾。**

##用户账户与手机号码验证

LeanCloud 提供了内建的账户系统，方便开发者快速接入，并且，我们也支持账户系统与手机号码绑定的一系列功能，譬如：

###使用手机号码一键注册或登录

现在很多应用都喜欢让用户直接输入手机号码注册，如果手机号码存在则自动登录，我们也提供了一个新 API `POST /usersByMobilePhone` 来处理:

```sh
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  -d '{"mobilePhoneNumber":"186xxxxxxxx","smsCode":"6 位短信验证码"}' \
  https://api.leancloud.cn/1.1/usersByMobilePhone
```

其中 `mobilePhoneNumber` 就是手机号码，而 `smsCode`是使用[短信验证 API](#短信验证_API)发送到手机上的 6 位验证码字符串。如果不传入 `username`，默认用户名将是手机号码。

注册或者登录成功后，返回的应答跟登录接口类似：

```json
{
  "username": "186xxxxxxxx",
  "mobilePhone": "186xxxxxxxx",
  "createdAt": "2011-11-07T20:58:34.448Z",
  "updatedAt": "2011-11-07T20:58:34.448Z",
  "objectId": "51c3ba66e4b0f0e851c1621b",
  "sessionToken": "pnktnjyb996sj4p156gjtp4im"
  ……其他属性
}
```

如果是第一次注册，将默认设置 `mobilePhoneVerified` 属性为 `true`。

### 手机号码验证

在应用设置的应用选项里你还可以选择开启注册手机码号验证，当注册的时候用户填写`mobilePhoneNumber`字段，  LeanCloud  将向该手机号码发送一条附带验证码的验证短信，用户在输入验证码后调用  LeanCloud  的 API 验证通过后，用户的`mobilePhoneNumberVerified`属性将设置为`true`。

假设你在开启注册手机号码验证选项后，注册下列用户：

```sh
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"username":"cooldude6","password":"p_n7!-e8","mobilePhoneNumber":"186xxxxxxxx"}' \
  https://api.leancloud.cn/1.1/users
```

那么在注册成功后， LeanCloud  将向`186xxxxxxxx`发送一条验证短信。开发者提供一个输入框让用户输入这个验证短信中附带的验证码，之后调用下列 API 来确认验证码正确：

```sh
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{}' \
  https://api.leancloud.cn/1.1/verifyMobilePhone/6位数字验证码
```

其中 URL 中的 `verifyMobilePhone` 后面就是6位验证数字。

验证成功后 ，用户的`mobilePhoneNumberVerified`将变为true，并会触发调用云引擎的`AV.Cloud.onVerified(type, function)`方法，type 被设置为 `sms`。

### 请求手机号码验证

用户除了被动等待收到验证码短信之外，或者短信意外没有收到的情况下，开发者可以主动要求发送验证码短信：

```sh
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"mobilePhoneNumber": "186xxxxxxxx"}' \
  https://api.leancloud.cn/1.1/requestMobilePhoneVerify
```

### 手机号码＋验证码登录

在验证过手机号码后，用户可以采用短信验证码登录，来避免繁琐的输入密码的过程，请求发送登录验证码：

```sh
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"mobilePhoneNumber": "186xxxxxxxx"}' \
  https://api.leancloud.cn/1.1/requestLoginSmsCode
```

在用户收到短信验证码之后，可以输入该验证码加上手机号码来登录应用：

```sh
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'mobilePhoneNumber=186xxxxxxxx' \
  --data-urlencode 'smsCode=123456' \
  https://api.leancloud.cn/1.1/login
```

也可以采用手机号码和密码的方式登录：

```sh
curl -X GET \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -G \
  --data-urlencode 'mobilePhoneNumber=186xxxxxxxx' \
  --data-urlencode 'password=p_n7!-e8' \
  https://api.leancloud.cn/1.1/login
```

### 手机号码＋验证码重置用户密码

如果用户使用手机号码注册，您也许希望也能通过手机短信来实现`忘记密码`功能，通过：

```sh
curl -X POST \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"mobilePhoneNumber": "186xxxxxxxx"}' \
  https://api.leancloud.cn/1.1/requestPasswordResetBySmsCode
```

发送一条重置密码的短信验证码到注册用户的手机上，需要传入注册时候的`mobilePhoneNumber`。

用户收到验证码后，调用`PUT /1.1/resetPasswordBySmsCode/:code`来设置新的密码（其中 URL 中的 `code` 就是6位验证数字）：

```sh
curl -X PUT \
  -H "X-AVOSCloud-Application-Id: {{appid}}" \
  -H "X-AVOSCloud-Application-Key: {{appkey}}" \
  -H "Content-Type: application/json" \
  -d '{"password": "new password"}' \
  https://api.leancloud.cn/1.1/resetPasswordBySmsCode/收到的6位验证码
```

修改成功后，就可以用新密码登录了。

## 常见问题 FAQ
### 短信 100 条的免费额度是针对每个账户还是每个应用？

每个 LeanCloud 账户拥有 100 条免费的短信测试额度，**不是每个应用**。使用完毕就需要付费。可以通过 [充值](/bill.html#/bill/charge) 菜单来购买短信。

**注意：为了不影响其他服务的使用，我们将短信费用和普通服务费用分开计算，所以充值的时候请明确选择「购买短信」，万一充错了，我们也支持用账户余额来购买短信。**

### 短信的到达率如何？

我们通过接入多个短信提供商来提升普通短信的到达率。从我们实际使用来看，整体的到达率在 97%－98% 之间。为了保证重要操作的验证信息可以 100% 送达，我们也推出了语音验证码服务（保证百分百到达），因此，除了普通的文本短信之外，还可以使用语音短信或者其他备份手段来向用户提供应用的重要功能。

### 我怎么知道单条短信发送成功与否
LeanCloud 通过运营商通道发送的每一条短信，都可以通过「接收回执」来确认发送结果。一般有三种状态：

* 成功。这表示收到了目标手机发回来的短信接收回执，消息已经确认无误送达。
* 失败。这表示出现内部错误，导致发送失败。内部错误则包括目标号码因为某种原因进入发送方黑名单、短时间内发送过量短信、甚至是政策调整等等情况。
* 等待。表示运营商服务器尚未收到目标手机的接收回执，发送结果可能成功也可能失败，当前不可知。

我们控制台「消息」->「短信」->「发送记录」里列出了每条消息和它的发送状态，大家可以从那里清楚地查看结果。

### 短信签名是什么？必须的吗？
根据运营商的要求，短信签名是必须的。短信签名是指短信里 `【】` 括起来的短信发送方名称。我们只允许自定义短信模板的用户自定义签名，其他用户都使用应用名称。
**短信签名限制在 10 个字符内，应用名称可以在应用设置里修改，并且短信签名必须出现在短信内容的开头或者结尾。**

### 短信有什么限制吗？

针对验证码类短信，有两个限制：

* 针对同一个手机号码，一分钟只能发送一条验证码短信。
* 每天只能发送 10 条验证短信给同一个手机号码。

以上限制对测试手机号码也同样有效。

通知类短信没有流控限制。但是我们禁止下列短信：

* 推广营销用途
* 包含不明下载地址
* 涉及色情、政治等法律禁止情况

如果涉及上述不正当用途，我们将停止您的短信使用权限。

### 短信支持港澳台和国外吗？

我们已经支持。

### 短信余额不足有预警通知吗？

有。请在 [开发者帐户](/settings.html#/setting/info) 里填写手机号码，我们将提前通过邮件和短信的方式向您 发送告警信息。

