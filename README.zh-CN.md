# Nikon 1 自制固件

[English](README.md) | 简体中文

这个项目为尼康 1 系列相机添加一些原厂未提供的功能。目前支持 J5，提供纯机械镜头功能、FT1 功能扩展和更短的间隔拍摄设置。

## 支持的机型

| 机型 | 版本 | 新增功能 | 使用说明 |
|---|---|---|---|
| Nikon 1 J5 | patched v10 | Wi-Fi/Fn2 按钮自定义、纯机械镜头支持、FT1 部分对焦区域解锁、1 秒间隔拍摄及 FT1 镜头兼容性预览 | [J5 使用说明](cameras/J5/README.zh-CN.md) |

FT1 功能仍有兼容性限制，包括触摸快门可能持续等待合焦的问题。刷入前请阅读 [J5 使用说明](cameras/J5/README.zh-CN.md)。

## 下载与安装

在 [Releases](https://github.com/XCQi/Nikon-1-custom-firmware/releases) 下载对应机型的 BIN 文件，按机型说明刷入即可，无需编译或自行打补丁。每个机型的文件夹中都有功能介绍、刷机步骤、回退方法和更新记录。

## 开发记录

修改思路和技术细节见[开发文档](docs/DEVELOPMENT.zh-CN.md)。

## 赞助

固件完全免费。如果这些功能对你有帮助，也欢迎自愿赞助，支持后续开发。

[通过 PayPal 赞助](https://paypal.me/CircuseeX)

<details>
<summary>微信、支付宝赞助</summary>

微信赞赏码：

<a href="assets/support/wechat.jpg"><img src="assets/support/wechat.jpg" alt="微信赞赏码" width="240"></a>

支付宝收款码：

<a href="assets/support/alipay.png"><img src="assets/support/alipay.png" alt="支付宝收款码" width="240"></a>

点击图片可查看原图。

</details>

## 关于项目

这是个人开发的非官方固件，与尼康官方无关。使用前请阅读对应机型的说明，了解已测试的功能和限制。原始固件的版权归尼康所有。
