# 688 路线报告 v2 —— 按 hlpatch 仓库 git 迭代记录实锤修正

> 2026-10-04。这版把上一版报告的错误修正了：数据源 = hlpatch 仓库 317 个 commit 的 git 历史 + reverse/ 目录实物，不是记忆。

## 一、hlpatch 仓库创建初期真实流程（git 记录，时间精确到分钟）

**2026-07-10 19:27** commit `92ff38a`「添加赛马娘 v2.28.5 APK 文件」
- `base.apk` + `split_config.arm64_v8a.apk` 两个拆分包，**git LFS 方式推进 hlpatch 仓库根目录**
- `.gitattributes` 声明：`*.apk filter=lfs diff=lfs merge=lfs -text`（绕 100MB 限制的就是 LFS）

**19:32** `64779c2` 更新 APK 文件（覆盖一次——跟这次 688 空壳需要覆盖重传一个套路）

**2026-07-11 10:26** `425b5f7`「赛马娘v2.28.5全量逆向分析报告 (20+文件)」
- APK 传进仓库后第二天出全量报告，工具就两个 Python 脚本，**全部还在仓库 reverse/ 目录**：
  - **`extract_metadata.py`**（84行）：手工扫描 APK 的 zip 结构 → 按文件名找 metadata → 找不到就按魔数 `AF 1B B1 FA` 扫所有大文件 → 解压出 `global-metadata.dat`
  - **`analyze.py`**（756行）：吃 `data/il2cpp_dump/dump_all_methods_ALL.json`（27,695 类 / 160,909 方法）→ 产出 20+ 份报告（14 剧本全解、ID 映射、ObscuredInt 布局、全类 dump）
- **重要事实：v2.28.5 时代 global-metadata.dat 在 APK 里是明文，直接解析成功**（53,684 类型定义 / 336,729 方法全出来）

## 二、上一版报告的勘误

| 上一版说法 | 实际（git 记录） |
|---|---|
| "离线解密从来没成过" | **错。v2.28.5 时代明文 metadata 离线解析成功过**，产出 27,695 类全表，至今在 reverse/ 目录 |
| （没提工具） | `extract_metadata.py` + `analyze.py` 是现成轮子，可直接复用 |
| "必须运行时 dump" | 676 之后官方才把 metadata 从 APK 里藏掉，那时离线才死的 |

版本演化实锤：
- **v2.28.5（7月）**：metadata 明文在包里 → 离线解析 ✓
- **676/686（9月）**：官方藏掉 metadata → 离线死 → 转运行时 SO dump
- **3.28.2（10月2日）**：运行时 dump 成熟（offset-ledger 21 类全表 + 踩坑账）
- **688（现在）**：**先跑一遍 extract_metadata.py 看官方这版藏没藏**——藏了再走运行时，不重造轮子

## 三、这次 688 照抄初期的执行单

1. 688 两个 APK 落盘（现在还是空壳，等覆盖重传）→ sha256 校验
2. 传 `uma-jp-688-apk` release 归档（同时按初期 LFS 方式留一份在仓库——.gitattributes 已含 `*.apk filter=lfs`）
3. 解出 `libil2cpp.so` 验符号完整性（686 验过 241 个符号全在，688 预期一样）
4. **跑 `reverse/extract_metadata.py`（路径改成 688 APK）**：
   - 找到明文 metadata → 离线全量解析出 688 类结构（跟初期一样出全表）
   - 找不到 → 确认 688 也藏了 → 走运行时：开嗅探 + 逐类 dump（10-02 流程）
5. 产出 688 偏移全表 → 填进修复版 hook 验证清单 → 重编 SO → hook 上、数据通

## 四、现状卡点（不变）

688 两个 APK 是空壳（142MB + 82MB 全零字节），等覆盖重传。
`meta` / `meta-plain.db` 是完整好文件，不用动。
