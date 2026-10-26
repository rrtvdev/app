---
name: rrtv-download
description: Find or download 人人视频 (RRTV) installers for Windows or Mac, and guide Android/iOS users to the official download page. Use for download or install requests, including “安装到本机”. Not for video downloads or backend development.
---

# 人人视频安装包下载

固定项目：https://github.com/rrtvdev/rrtv 。直接提供或下载安装包。

## 判断用户设备

支持这样的提示词：

> 帮我把 GitHub 上的 rrtvdev 的 rrtv 安装包安装到本机。

“本机”指用户要安装应用的设备。优先使用用户说明或可信设备上下文；只有确认工具运行在用户目标设备上时，才用工具检测系统。不能把云端、服务器或容器的系统当成用户设备。无法判断时询问“你要安装到手机/平板，还是 Windows/Mac 电脑？”；只说电脑版时确认 Windows 还是 Mac。

## 手机和平板

手机、平板、Android、iPhone、iPad 或 iOS 请求，直接返回：

> 请在手机或平板浏览器中打开 [人人视频官网下载页](https://rrsp.app/)，按照页面提示下载并安装。

无需再区分安卓或 iOS，也不下载仓库里的历史 APK。指定移动端版本时说明可用版本以官网为准。

## Windows 和 Mac

读取固定清单：https://raw.githubusercontent.com/rrtvdev/rrtv/main/downloads.json 。若不可访问，可读取同仓库的 https://github.com/rrtvdev/rrtv/blob/main/downloads.json 。不要靠搜索同名仓库选包；用户明确指定其他来源时尊重其要求。

- Windows/Win 对应 `windows`，Mac/macOS 对应 `macos`。
- 指定版本时，在该系统的 `files` 中匹配 `version`；Windows 四段构建号匹配 `file_version`。未收录则告知用户。
- 未指定版本时，用 `latest_by_platform[os]` 找到对应 `id`。这里的最新仅指仓库收录版本，不跨系统比较版本号，也不使用仓库级 `releases/latest` 选包。
- 直接使用该记录的 `download_url`。链接应指向 `https://github.com/rrtvdev/rrtv/releases/download/` 下的安装包附件。

用户要链接时给可点击的下载地址；要求下载时保存到指定位置，否则使用用户目标设备的下载目录。同名文件另存，避免覆盖。下载工具报告完成后，简短返回平台、版本和实际保存路径，提示用户打开安装包安装。

不执行大小或 SHA-256 校验，不要求用户运行校验命令。下载失败就说明失败；无法在用户设备上保存时给直链，不声称已下载到用户本机。本 Skill 不自动运行安装器。

清单无法读取时提供固定仓库链接并说明原因，不猜测版本或链接。安装包来源以清单为准；未知的架构和兼容性不作承诺。新增版本读取仓库清单即可，Skill 规则更新需要更新本地副本。
