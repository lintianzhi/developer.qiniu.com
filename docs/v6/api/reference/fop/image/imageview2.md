---
layout: docs
title: 图片处理（imageView2）
order: 250
---

<a id="imageView2"></a>
# 图片处理（imageView2）

<a id="imageView2-description"></a>
## 描述

imageView2是原imageView接口的更新版本，实现略有差异，功能更为丰富。  
同样，只需要填写几个参数即可对图片进行缩略操作，生成各种缩略图。  

<a id="imageView2-specification"></a>
## 接口规格

```
ImageView2Spec = imageView2/<mode>/w/<Width | LongEdge>/h/<Height | ShortEdge>
                           /q/<Quality>
                           /format/<Format>
```

其中 `<mode>` 分为如下几种情况：  

模式                            | 说明
:------------------------------ | :----------------------------------------------------------------------------
`/0/w/<LongEdge>/h/<ShortEdge>` | 等比缩略，新图不大于<LongEdge>x<ShortEdge>限定的矩形框，不裁剪。
`/1/w/<Widht>/h/<Height>`       | 等比缩略，新图不小于<Width>x<Height>限定的矩形框，居中裁剪。
`/2/w/<Widht>/h/<Height>`       | 等比缩略，新图不大于<Width>x<Height>限定的矩形框，不裁剪。
`/3/w/<Widht>/h/<Height>`       | 等比缩略，新图不小于<Width>x<Height>限定的矩形框，不裁剪。
`/4/w/<LongEdge>/h/<ShortEdge>` | 等比缩略，新图不小于<LongEdge>x<ShortEdge>限定的矩形框，不裁剪。
`/5/w/<LongEdge>/h/<ShortEdge>` | 等比缩略，新图不小于<LongEdge>x<ShortEdge>限定的矩形框，居中裁剪。

注意：  
1. 可以仅指定w参数或h参数；  
2. 新图的宽/高/长边/短边，不会比原图大，即本接口总是缩小图片。 

参数名称            | 必填  | 说明
:------------------ | :---- | :--------------------------------------------------------------------------------
`/q/<Quality>`      |       | 新图的图像质量，取值范围：1-100，缺省为85。<br>如原图质量小于指定值，则按原值输出。
`/format/<Format>`  |       | 新图的输出格式，取值范围：jpg，gif，png，webp等，缺省为原图格式。

<a id="imageView2-request"></a>
## 请求

<a id="imageView2-request-syntax"></a>
### 请求报文格式

```
GET <ImageDownloadURI>?<ImageView2Spec> HTTP/1.1
Host: <ImageDownloadHost>
```

<a id="imageView2-request-header"></a>
### 请求头部

头部名称       | 必填 | 说明
:------------- | :--- | :------------------------------------------
Host           | 是   | 下载服务器域名，可为七牛三级域名或自定义二级域名，参考[域名绑定][cnameBindingHref]

---

<a id="imageView2-response"></a>
## 响应

<a id="imageView2-response-syntax"></a>
### 响应报文格式

```
HTTP/1.1 200 OK
Content-Type: <ImageMimeType>

<ImageBinaryData>
```

<a id="imageView2-response-header"></a>
### 响应头部

头部名称       | 必填 | 说明
:------------- | :--- | :------------------------------------------
Content-Type   | 是   | MIME类型，成功时为图片的MIME类型，失败时为application/json
Cache-Control  |      | 缓存控制，失败时为no-store，不缓存

<a id="imageView2-response-content"></a>
### 响应内容

■ 如果请求成功，返回图片的二进制数据。  

■ 如果请求失败，返回包含如下内容的JSON字符串（已格式化，便于阅读）：  

```
{
	"code":     <HttpCode  int>, 
    "error":   "<ErrMsg    string>",
}
```

字段名称     | 必填 | 说明                              
:----------- | :--- | :--------------------------------------------------------------------
`code`       | 是   | HTTP状态码，请参考[响应状态](#imageView2-response-status)
`error`      | 是   | 与HTTP状态码对应的消息文本

<a id="imageView2-response-code"></a>
### 响应状态码

HTTP状态码 | 含义
:--------- | :--------------------------
200        | 缩略成功
400	       | 请求报文格式错误
404        | 资源不存在
599	       | 服务端操作失败。<br>如遇此错误，请将完整错误信息（包括所有HTTP响应头部）[通过邮件发送][sendBugReportHref]给我们。

---

<a id="imageView2-remarks"></a>
## 附注

- imageView2生成的图片会被七牛云存储缓存以加速下载，但不会持久化。需要持久化的缩略图，请参考[触发持久化处理][pfopHref]。  
- 如果原图带有[EXIF][exifHref]信息且包含`Orientation`字段，imageView2缺省根据此字段的值进行自动旋转修正。
- 具备处理动态gif图片的能力。
- 当一张含有透明区域的图片，转换成不支持透明的格式（jpg, bmp, etc...）时，透明区域填充白色。
- 当处理并输出多帧gif图片时，可能处理所需的时间较长并且输出的图片体积较大，建议使用[预转持久化处理][persistentOpsHref]或[触发持久化处理][pfopHref]进行转码。

<a id="imageView2-samples"></a>
## 示例

1. 裁剪正中部分，等比缩小生成200x200缩略图：  

	```
    http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView2/1/w/200/h/200
	```

	![查看效果图](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView2/1/w/200/h/200)


2. 宽度固定为200px，高度等比缩小，生成200x133缩略图：  

	```
    http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView2/2/w/200
	```

	![查看效果图](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView2/2/w/200)

3. 高度固定为200px，宽度等比缩小，生成300x200缩略图：  

	```
    http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView2/2/h/200
	```

	![查看效果图](http://qiniuphotos.qiniudn.com/gogopher.jpg?imageView2/2/h/200)

---

<a id="imageView2-internal-resources"></a>
## 内部参考资源

- [域名绑定][cnameBindingHref]
- [触发持久化处理][pfopHref]
- [预转持久化处理][persistentOpsHref]

[cnameBindingHref]:  http://kb.qiniu.com/53a48154                     "域名绑定"
[pfopHref]:          ../pfop/pfop.html                                "触发持久化处理"
[persistentOpsHref]: ../../security/put-policy.html#put-policy-struct "预转持久化处理"
[exifHref]:          exif.html                                        "EXIF信息"

[sendBugReportHref]: mailto:support@qiniu.com?subject=599错误日志 "发送错误报告"
