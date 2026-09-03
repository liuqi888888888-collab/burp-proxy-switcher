# Burp Proxy Switcher

一个 Manifest V3 浏览器扩展(Chrome / Edge),一键把浏览器流量切换到 Burp Suite 代理。

## 功能

- 一键开/关代理,工具栏角标显示状态,快捷键 `Alt+Shift+B`
- 固定代理模式:默认 `127.0.0.1:8080`,可自定义地址/端口/绕过列表
- PAC 模式:支持 PAC URL 或内联 PAC 脚本,按域名选择性转发到 Burp
- 关闭时可选直连或跟随系统代理
- 一键打开 Burp CA 证书下载页(`http://burp/cert`)

## 安装

1. 打开 `chrome://extensions`(Edge 为 `edge://extensions`)
2. 右上角开启 **开发者模式**
3. 点击 **加载已解压的扩展程序**,选择本目录

## 使用

1. 在 Burp 中确认代理监听已开启(默认 `127.0.0.1:8080`)
2. 点击扩展图标,打开开关(或按 `Alt+Shift+B`)
3. 此时浏览器流量即进入 Burp 的 Proxy 历史

### HTTPS 抓包

1. 代理开启状态下,访问 `http://burp/cert`(或点弹窗中的下载 Burp CA 证书)
2. 将下载的证书导入到 **受信任的根证书颁发机构**(当前用户即可)
3. 刷新目标页面即可看到解密后的 HTTPS 流量

### 只转发部分域名(示例)

选择 PAC 脚本内容 模式,填入:

```js
function FindProxyForURL(url, host) {
  if (dnsDomainIs(host, '.target.com')) {
    return 'PROXY 127.0.0.1:8080';
  }
  return 'DIRECT';
}
```

## 常见问题

- **ERR_PROXY_CONNECTION_FAILED**:Burp 未启动,或地址/端口与 Burp 监听不一致。
- **部分请求抓不到**:Chrome 的 HTTP/3(QUIC)不走代理。到 `chrome://flags/#enable-quic` 设为 Disabled;同时可在 `chrome://settings/security` 关闭使用安全 DNS。
- **角标显示 `!`**:代理设置被企业策略或其他扩展(如 SwitchyOmega)占用,需先关闭冲突来源。
- **证书报错**:确认已导入根证书;个别站点(证书固定、客户端内置校验)仍可能拒绝。
- 仅支持 Chromium 系浏览器;Firefox 请使用 FoxyProxy。
- 扩展只修改浏览器代理,不改系统代理;抓移动端/其他进程流量请手动设置系统或设备代理。

## 目录结构

```
burp-proxy-switcher/
├── manifest.json      # MV3 清单
├── background.js      # Service Worker:代理设置、角标、快捷键
├── popup.html         # 弹窗界面
├── popup.js           # 弹窗逻辑
├── icons/             # 图标(由脚本生成)
├── tools/
│   └── gen-icons.mjs  # 图标生成脚本(无依赖)
└── README.md
```

重新生成图标:`node tools/gen-icons.mjs`
