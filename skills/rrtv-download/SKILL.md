---
name: rrtv-download
description: Download 人人视频 (RRTV) installers, install Windows silently or copy the Mac app into Applications when requested, launch the installed desktop app after successful installation, and guide Android/iOS users to the official download page. Use for download or install requests, including “安装到本机”. Not for video downloads or backend development.
---

# 人人视频安装包下载

固定项目：https://github.com/rrtvdev/rrtv 。提供或下载安装包；用户要求安装时，Windows 使用静默参数，Mac 自动挂载 DMG 并复制应用完成安装，随后自动打开已安装的应用一次。

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

用户要链接时给可点击的下载地址；要求下载时保存到指定位置，否则使用用户目标设备的下载目录。同名文件另存，避免覆盖。下载工具报告完成后，按请求处理：仅要下载则返回平台、版本和实际保存路径；Windows 安装请求继续下面的静默安装步骤；Mac 安装请求继续下面的自动安装步骤。

不执行大小或 SHA-256 校验，不要求用户运行校验命令。下载失败就说明失败；无法在用户设备上保存时给直链，不声称已下载到用户本机。仅要求下载时不安装或启动应用。

### Windows 静默安装

用户要求安装 Windows 版，且工具已确认运行在用户目标 Windows 设备上时，下载后按该记录的 `install.silent_arguments` 执行安装。当前 1.25.0 为 NSIS，参数是大写 `/S`。使用实际保存路径，不假定文件仍使用原始名称。例如 PowerShell 中 `$installer` 为下载所得 EXE 的完整路径：

```powershell
$process = Start-Process -FilePath $installer -ArgumentList "/S" -Verb RunAs -Wait -PassThru
$process.ExitCode
```

此包要求管理员权限；如出现 Windows 授权提示，需用户确认。等待安装器结束，根据退出码和可观察的安装结果报告成功、失败或尚无法确认；不能仅凭下载完成或进程启动就声称安装完成。用户取消授权或安装失败时说明实际结果，不自动重复安装。确认安装成功后继续“安装后自动打开”，不再让用户手动找图标。

无法在用户目标 Windows 设备执行时，给下载链接及使用其文件路径的静默安装命令，不声称已安装。新版本没有 `install` 静默参数时不要猜测参数。[NSIS 参数文档](https://nsis.sourceforge.io/Docs/Chapter3.html#installerusage)。

### Mac 自动安装

用户要求安装 Mac 版，且工具已确认运行在用户目标 Mac 上时，下载完成后继续安装，不停留在“请手动打开 DMG”。按该记录的 `install` 配置处理；当前包是 App 拷贝型 DMG，应用为 `人人视频 for Mac.app`，Bundle ID 为 `com.rrtv.www.RRTV-MAC`，版本为 `1.2.3`（构建 `1`），要求 macOS 12.0 或更新版本。

1. 使用实际下载路径。确认系统满足 `minimum_macos`，默认目标目录为 `/Applications`；该目录不可写时，创建并使用用户的 `~/Applications`，不为此自动提权。清单中的 `~` 必须先展开为目标用户主目录的绝对路径，再赋给 `$target_dir`；不要把含字面量 `~` 的变量传给复制命令。
2. 用 `hdiutil attach -readonly -nobrowse -noautoopen -plist "$dmg"` 挂载，从返回的 plist 读取本次镜像的实际挂载点，不硬编码 `/Volumes` 下的名称。定位 `install.app_bundle`，不执行镜像中其他文件。
3. 将应用复制到目标目录。下面的 `$mount_point` 和 `$target_dir` 分别为本次实际挂载点、选定的安装目录：

   ```sh
   ditto "$mount_point/人人视频 for Mac.app" "$target_dir/人人视频 for Mac.app"
   ```

   安装前检查目标同名应用；Bundle ID、版本和构建号均相同时，直接复用并继续“安装后自动打开”。旧版同一应用需要更新时，先关闭运行中的应用并将旧 Bundle 移到唯一备份路径，再复制新 Bundle，避免直接叠加覆盖旧文件。不要删除用户数据；复制失败则移开本次不完整副本并恢复备份。不同 Bundle ID 的同名应用不覆盖，说明冲突。
4. 确认复制命令成功且目标应用存在，读取目标 `Info.plist` 确认名称、Bundle ID 和版本后报告实际安装路径。无论复制成功或失败，都用 `hdiutil detach "$mount_point"` 卸载本次镜像；卸载失败则说明实际情况，不强制卸载其他卷。

保留 macOS 安全属性和系统检查，不清除隔离属性、不关闭 Gatekeeper。安装成功后继续“安装后自动打开”，使用已经复制到目标目录的应用，不从挂载镜像启动。若系统不兼容、操作失败或无法在用户目标 Mac 执行，说明具体原因并提供下载链接/操作步骤，不声称已安装。后续版本按清单里的应用名称和安装方式处理，不猜测缺失的 `install` 配置。

### 安装后自动打开

Windows 和 Mac 安装成功后，默认在用户目标设备上打开应用一次；复用已安装的相同版本时也执行此步骤。这里不设置开机或登录自启动，不添加登录项、启动文件夹或计划任务。用户明确要求安装后不要打开时跳过；仅要链接或下载、安装失败、目标设备不可访问时不启动。

- Windows：在安装生成的用户或公共开始菜单、桌面快捷方式中定位人人视频，读取其实际目标确认是已安装应用，排除安装器、卸载器和更新入口；如果没有快捷方式，可从该应用安装信息中确认主程序路径，不猜测 EXE 名称或安装目录；入口缺失或不唯一时说明问题。在普通用户 PowerShell/桌面会话中打开该快捷方式，例如 `$shortcutPath` 为已确认的 `.lnk` 完整路径：

  ```powershell
  Start-Process -FilePath $shortcutPath
  ```

  应用启动不使用安装时的 `-Verb RunAs`。如果安装器已经启动应用或应用已在运行，复用现有实例，不重复启动。
- Mac：用实际安装路径打开应用，包括回退到用户 `~/Applications` 或复用既有安装的情况。`$target_dir` 必须是已展开的绝对路径：

  ```sh
  open "$target_dir/人人视频 for Mac.app"
  ```

  不使用 `open -n` 创建重复实例，不以管理员身份启动应用。

根据实际窗口或进程状态确认是否已打开，分别报告安装路径与启动结果。若启动失败、找不到入口或出现系统首次打开确认，说明“已安装，启动尚未完成”和具体原因；保留系统安全检查，不重复安装、不声称已启动。

清单无法读取时提供固定仓库链接并说明原因，不猜测版本或链接。安装包来源以清单为准；未知的架构和兼容性不作承诺。新增版本读取仓库清单即可，Skill 规则更新需要更新本地副本。
