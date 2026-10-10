# GitHub 云编译 RMG 临时 Root 载荷 —— 操作指南（SM-S9380 / 6.6.77）

> 目标：在不解锁 Bootloader、不触发 Knox、不刷机的前提下，用 Root-My-Galaxy（CVE-2026-43499 / GhostLock）给本机 **Galaxy S25 Ultra 国行 (SM-S9380, 固件 S9380ZCU5BYI3, 内核 6.6.77)** 拿到**临时 root**（重启即失效，需重新运行）。
>
> 本指南假设你已拿到 `rmg-bundle.zip`（由助手打包好的完整仓库）。

---

## 一、一次性准备（GitHub 网页端，约 3 分钟）

1. 打开 https://github.com 并登录你的账号。
2. 右上角 **「+」→ New repository**：
   - Repository name：`Root-My-Galaxy-Payloads`（或任意名）
   - 勾选 **Private（私有）** —— 你的内核偏移不必公开
   - **不要**勾 Initialize with README（保持空仓库）
   - 点 **Create repository**
3. 在空仓库页面，点 **「uploading an existing file」** 或把 `rmg-bundle.zip` 解压后的**全部内容直接拖进网页上传框**（GitHub 支持拖文件夹，会保留目录结构）。
   - 需要包含的关键文件：`Makefile`、`src/`、`support/targets-v3.json`、`kernelsu/ksud-s25u-kdp`、`.github/workflows/build.yml`、`artifacts/pa3q-S9380ZCU5BYI3/cve-2026-43499-app.so`。
   - 确认提交（Commit）。

> 说明：仓库里 `target.h` / `p0_fingerprint.h` 已是**从你自己的 boot.img 实测提取**的偏移，无需再改。

---

## 二、运行云编译（每次想拿 root 都不需要，编译一次即可，但 .so 只跟这台机器的固件绑定）

1. 进入仓库的 **Actions** 标签页。
   - 首次会提示 **「Enable workflows」/ 允许运行 Actions**，点允许。
2. 左侧选 **Build RMG payload (pa3q-S9380ZCU5BYI3)**，点 **Run workflow → Run workflow**（默认参数即可，目标 `pa3q-S9380ZCU5BYI3`、API `35`）。
3. 等待 2–4 分钟，状态变绿 ✅。点进去 → **Artifacts** 区下载 `cve-2026-43499-app-release`（里面就是 `cve-2026-43499-app.release.so`，**严格 104128 字节**）。

---

## 三、刷入 / 运行（手机端）

1. 手机安装 **Root-My-Galaxy (RMG) App**。
2. 打开 App → 进入 **Advanced / 高级模式**。
3. 选择 **Sideload / 手动加载 payload**，选刚才下载的 `cve-2026-43499-app.release.so`。
4. 点运行。利用的是内核 UAF 漏洞，**无需解锁 BL、无需 Magisk、重启后 root 消失**（再次运行 App 即可重新获取）。

---

## 四、常见问题

- **App 里没自动识别我的机型/内核？** 正常。官方 feed 只有 6.6.98 的 S25U 条目，你是 6.6.77，所以走 **Advanced 手动 sideload**，不要等自动匹配。
- **编译报 size mismatch / 不是 104128？** 说明 `target.h` 里某个偏移导致代码膨胀或链接异常。把 Actions 日志发我，我核偏移。
- **刷完重启 root 没了？** 这是设计行为（临时 root），不是失败。重新打开 App 跑一次即可。
- **Knox / 保修？** 该方案不解锁 Bootloader、不修改 boot/recovery 分区，理论上不触发 Knox 熔断；但仍属利用内核漏洞，风险自担。

---

## 五、文件清单（本仓库已就绪）

| 文件 | 作用 |
|---|---|
| `src/targets/pa3q-S9380ZCU5BYI3/target.h` | 本机实测内核偏移（已校验） |
| `src/targets/pa3q-S9380ZCU5BYI3/p0_fingerprint.h` | P0 物理内存指纹（32×8，已校验） |
| `support/targets-v3.json` | feed 条目（可选；sideload 用不到） |
| `.github/workflows/build.yml` | 云编译工作流（ubuntu + NDK r27c） |
| `kernelsu/ksud-s25u-kdp` | S25U 共用的 KernelSU 二进制 |
| `artifacts/pa3q-S9380ZCU5BYI3/cve-2026-43499-app.so` | 已编译好的 104128 字节载荷（兜底） |
