# Clash Verge Rev · 便携版（非官方）

> 上游官方从 **v2.5.4** 起删除了便携模式（`.config/PORTABLE`），且官方不再发布便携版。
> 本仓库是 [clash-verge-rev/clash-verge-rev](https://github.com/clash-verge-rev/clash-verge-rev) 的非官方便携构建，
> **由 GitHub Actions 每日自动检查上游新 tag、rebases 补丁并构建发布**。与上游代码仅差一个可审计的小补丁。

## 下载

见 [Releases](../../releases)：
- Windows：`Clash.Verge_<版本>_x64-portable.zip`
- Linux：`Clash.Verge_<版本>_linux-x64-portable.tar.gz`（x86_64）

## 使用

**Windows**
1. 解压到本地目录，例如 `D:\Tools\ClashVerge`（NTFS 最佳；exFAT/FAT32 盘也可以，见下方说明）
2. 双击 `clash-verge.exe`
3. 所有数据（订阅、设置、内核配置）保存在 `<解压目录>\.config\` 中，整个文件夹可拷贝/备份
4. 不依赖 `%APPDATA%`，U 盘随身模式可用（exFAT 下服务模式不可用，见下方说明）

**Linux**
1. 解压到本地目录，例如 `~/Tools/clash-verge`，运行 `./Clash\ Verge/clash-verge`
2. 运行时依赖（Debian/Ubuntu）：`sudo apt install libwebkit2gtk-4.1-dev libayatana-appindicator3-dev librsvg2-dev libxslt1.1 patchelf`（也可以只装这几个的 runtime 包：`libwebkit2gtk-4.1-0 libayatana-appindicator3-1`）
3. 数据保存在 `<解压目录>/.config/io.github.clash-verge-rev.clash-verge-rev/`，整文件夹可拷贝/备份，不依赖 `~/.local/share`
4. 服务模式（TUN/接管流量）：需要先运行 `clash-verge-service-install`（需 root/polkit），同 Windows 一样是否可用取决于文件系统；普通桌面环境推荐 **Sidecar** 运行模式

## 与官方正式版的区别

| 项目 | 官方版 | 本便携版 |
| --- | --- | --- |
| 便携模式（`.config/PORTABLE`） | ❌ v2.5.4 起移除 | ✅ 已恢复 |
| 数据目录 | `%APPDATA%\io.github.clash-verge-rev.clash-verge-rev` | `<程序目录>\.config\io.github.clash-verge-rev.clash-verge-rev` |
| 应用内更新检查 | ✅（会覆盖成非便携版） | ✅ **会提示并跳转本仓库 Releases 下载**（不静默安装） |
| 其余功能 | 全部 | 与上游同步（每日自动跟进） |
| 签名/公证 | 官方签名 | 无签名，SmartScreen 可能提示 |

## 注意事项

- **运行模式（NTFS vs exFAT/FAT32）**：
  - **NTFS 等支持 ACL 的卷**：行为与上游一致，全功能，含服务模式。
  - **exFAT / FAT32 等不支持 ACL 的卷**（常见于 U 盘）：本 fork 自 portable-v2.5.10 起可以直接启动。
    启动时通过 `GetVolumeInformationW` 的 `FILE_PERSISTENT_ACLS` 标志识别这类卷，自动跳过基于 SID 的
    属主校验（上游的校验在无 ACL 的卷上永远失败，会报 `current-user lock belongs to a different Windows user`）。
    此时"单例"仍然有效（系统文件锁，同一时间只能运行一个实例），但：
    - **服务模式不可用**（拥有者令牌认证同样依赖 SID），请在设置里改用 **Sidecar** 运行模式；
    - `.config` 没有任何权限位保护，同机其他账号可读写你的配置/订阅，**仅限自己使用**；
    - 未知文件系统如无法确认支持 ACL，会按 NTFS 的严格路径处理（fail-closed）。
- **WebView2 Runtime**：不随便携版附带，Windows 10 1803+ / Windows 11 一般自带。
- **更新方式**：应用会检测新版（检查源为本仓库 `releases/latest/download/update.json`），点"更新"按钮会跳转到本仓库 Releases 页面下载新的 `portable-vX.Y.Z` 压缩包，解压覆盖旧目录即可（`.config` 会保留）。
  之所以不做应用内自动安装，是因为便携版（zip）无法走 Clash Verge 的 NSIS 自更新通道，强行做会装成另一份非便携官方版。
- 官方不认可本构建，出问题请对比上游正式版行为。

## 这个仓库是怎么运作的

`portable` 分支 = 上游 release tag + 一个补丁 commit（`git log v2.5.10..portable` 可见全部改动，也保留在根目录 `portable.patch` 里）：

1. `src-tauri/src/utils/dirs.rs` — 恢复 `init_portable_flag()`：exe 同目录存在 `.config\PORTABLE` 时，数据目录指向 `<exe>\.config\io.github.clash-verge-rev.clash-verge-rev`
2. `src-tauri/src/lib.rs` — 在所有数据目录访问之前初始化该标记
3. `src-tauri/tauri.conf.json` — `createUpdaterArtifacts=false`，自更新端点指向本仓库的 `releases/latest/download/update.json`
4. `src/components/setting/mods/update-viewer.tsx` — 点"更新"按钮跳转到本仓库便携版 release 页面
5. `src-tauri/src/core/owner_identity.rs` — 在不支持 ACL 的卷（exFAT/FAT32）上跳过基于 SID 的属主校验，改为普通文件创建/锁定

CI（`.github/workflows/portable-build.yml`）每天运行：

1. 检查上游最新稳定版 tag（如 `v2.5.11`）
2. 若分支落后，把补丁 commit rebase 到新 tag 上并 force-push；冲突时自动开 issue
3. Windows（`update-and-release` job）：若该版本尚无 `portable-v<版本>` release，则构建 NSIS 安装包 → 静默安装到暂存目录 → 删卸载器 → 注入空的 `.config\PORTABLE` → 打包 zip 并发布 release（同时上传 `update.json` 供应用内检查更新）
4. Linux（`build-linux` job，依赖前者先完成）：若该版本 release 尚无 `Clash.Verge_<版本>_linux-x64-portable.tar.gz`，则用 ubuntu-22.04 构建 `clash-verge`（`--no-bundle`），连同 mihomo 内核、service sidecar、`resources/`、`.config/PORTABLE` 一起打 tar.gz 并上传到同一 release

## 手动构建

```bash
git clone <本仓库> && cd clash-verge-rev-portable
git checkout portable
pnpm i
pnpm run prebuild x86_64-pc-windows-msvc
pnpm tauri build --target x86_64-pc-windows-msvc
```

然后在 `exe` 同目录建 `.config\PORTABLE` 空文件即可。手动构建如需保留便携模式，请勿自行恢复官方默认的更新端点。

## 许可证

GPL-3.0，与上游一致。This is an unofficial build, not affiliated with the upstream project.
