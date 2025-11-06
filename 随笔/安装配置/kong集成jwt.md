### 1、创建一个Consumers

在Kong Manafer管理界面进入Consumers页面

![](.\img\consumers.jpg)

点击new consumer创建Consumers

![](.\img\new consumer.jpg)

返回上一页

![](.\img\consumer bord.jpg)





点击创建好了的选项进入详情，再点击credentials选项创建jwt

![](.\img\create jwt.jpg)

点击new jwt credential开始创建，key和secret很重要

![](.\img\jwt form.jpg)

### 2、 创建jwt

在Plugins页面点击new plugin进去点击jwt插件

![](.\img\jwt plugin.jpg)

在插件详情页面设置Header Names为项目请求头携带的信息，我这边请求头携带的是x-token就设置为x-token，

设置完成保存返回即可

![](.\img\jwt setting.jpg)

如果在项目中有设置jwt验证，需要将consumer的key设置为项目的Issuer的值，我的项目为例：

```
claims := models.CustomClaims{
		ID:          uint(user.Id),
		NickName:    user.NickName,
		AuthorityId: uint(user.Role),
		StandardClaims: jwt.StandardClaims{
			NotBefore: time.Now().Unix(),               //签名生效时间
			ExpiresAt: time.Now().Unix() + 60*60*24*30, //过期时间
			Issuer:    "mall",
		},
	}
```

consumer的Secret设置为项目jwt的key，我的项目为例

![](.\img\secret.jpg)

以上设置完成就可以测试了，可以用这个网站测试：https://jwt.io/。PAYLOAD添加iss，VERIFY SIGNATURE添加secret

![](.\img\jwt.jpg)

> ps. 请求测试的时候xtoken要加Bearer
>
> ![](.\img\bearer.jpg)
>
> 由于我的项目里的jwt没有Bearer，所以需要修改一下，JWT验证的时候添加以下一行代码
>
> ![](.\img\debug.jpg)
>
> 