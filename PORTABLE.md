# Clash Verge Rev · 便携版（非官方）

> 上游官方从 **v2.5.4** 起删除了便携模式（`.config/PORTABLE`），且官方不再发布便携版。
> 本仓库是 [clash-verge-rev/clash-verge-rev](https://github.com/clash-verge-rev/clash-verge-rev) 的非官方便携构建，
> **由 GitHub Actions 每日自动检查上游新 tag、rebases 补丁并构建发布**。与上游代码仅差一个可审计的小补丁。

## 下载

见 [Releases](../../releases)：`Clash.Verge_<版本>_x64-portable.zip`

## 使用

1. 解压到任意**本地 NTFS** 目录，例如 `D:\Tools\ClashVerge`
2. 双击 `Clash Verge.exe`
3. 所有数据（订阅、设置、内核配置）保存在 `<解压目录>\.config\` 中，整个文件夹可拷贝/备份
4. 不依赖 `%APPDATA%`，U 盘随身模式可用（服务模式限制见下）

## 与官方正式版的区别

| 项目 | 官方版 | 本便携版 |
| --- | --- | --- |
| 便携模式（`.config/PORTABLE`） | ❌ v2.5.4 起移除 | ✅ 已恢复 |
| 数据目录 | `%APPDATA%\io.github.clash-verge-rev.clash-verge-rev` | `<程序目录>\.config\io.github.clash-verge-rev.clash-verge-rev` |
| 应用内自更新 | ✅（可能把你覆盖成非便携版） | ❌ **已禁用** |
| 其余功能 | 全部 | 与上游同步（每日自动跟进） |
| 签名/公证 | 官方签名 | 无签名，SmartScreen 可能提示 |

## 注意事项

- **服务模式**（需要管理员权限的 TUN / 系统代理服务）：便携目录必须放在**本地 NTFS 盘**、由当前用户拥有，
  否则上游的所有权校验（`AppDataRootNotOwned`）会拒绝启动内核。U 盘 / exFAT 请在设置里改用 **Sidecar** 运行模式。
- **WebView2 Runtime**：不随便携版附带，Windows 10 1803+ / Windows 11 一般自带。
- **自动更新**：已禁用。升级方式 = 下载新的 `portable-vX.Y.Z` 压缩包解压覆盖旧目录（`.config` 会保留）。
- 官方不认可本构建，出问题请对比上游正式版行为。

## 这个仓库是怎么运作的

`portable` 分支 = 上游 release tag + 一个补丁 commit（`git log v2.5.10..portable` 可见全部改动，也保留在根目录 `portable.patch` 里）：

1. `src-tauri/src/utils/dirs.rs` — 恢复 `init_portable_flag()`：exe 同目录存在 `.config\PORTABLE` 时，数据目录指向 `<exe>\.config\io.github.clash-verge-rev.clash-verge-rev`
2. `src-tauri/src/lib.rs` — 在所有数据目录访问之前初始化该标记
3. `src-tauri/tauri.conf.json` — `createUpdaterArtifacts=false`，自更新端点指向不可达地址（防止被官方版覆盖）

CI（`.github/workflows/portable-build.yml`）每天运行：

1. 检查上游最新稳定版 tag（如 `v2.5.11`）
2. 若分支落后，把补丁 commit rebase 到新 tag 上并 force-push；冲突时自动开 issue
3. 若该版本尚无 `portable-v<版本>` release，则构建 NSIS 安装包 → 静默安装到暂存目录 → 删卸载器 → 注入空的 `.config\PORTABLE` → 打包 zip 并发布 release

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
