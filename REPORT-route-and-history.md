# 688 安装包怎么用 —— 历史做法全盘点 + 这次的路线

> 2026-10-04 整理。数据来源：全量对话记忆 + 所有仓库实查。

## 一、三个时期的做法（时间线）

### 时期1：7月初，hlpatch 刚建仓（你说的"仓库终端"= GitHub Codespaces）
1. 你把 `base.apk` + `split_config.arm64_v8a.apk` 放进 Codespaces 终端
2. 终端里 `unzip` 解包，抽出两个关键文件：
   - `libil2cpp.so`（游戏大脑库）
   - `global-metadata.dat`（大脑的说明书，类/字段/偏移全靠它）
3. `gh release create` 上传到仓库 release 存档
4. **结果：Il2CppDumper 被游戏保护（Coneshell）挡住，说明书文件本身也被游戏加密（熵 7.99，魔数被抹）——离线解密这条路当场就死了**
5. 最后改走运行时：SO 插件在手机上问游戏进程直接要（游戏运行时自己会解开说明书）

### 时期2：9月18日，686
1. 你传的 APK 首传是**空壳**（大小对、内容全零）→ **覆盖重传一次**才落盘成功
2. APK 存进 `uma-apk-archive`（私有仓）v686 release
3. 解出 `libil2cpp.so` 225MB：241 个 il2cpp 接口符号全在、没加固壳 → **但说明书 global-metadata.dat 没在包里**（官方藏了）
4. 给 SO 加了运行时全量偏移导出端点（v3.29.0, dump_offsets 模块）

### 时期3：10月2日，3.28.2 真机实测（最近一次成功的偏移采集）
完整流程已归档：`uma-3.28.2-offset-ledger` → `2026-10-02/` 目录：
- **必须先开嗅探** `GET /api/sniff/toggle?enabled=1`，不开嗅探 → 所有地址 0x0、一切端点报 booting，啥都拿不到
- 开了之后 5 个 hook（compress/decompress/post/makemd5/computehash）自动装上、地址变真实值
- 字段偏移：`uma_get_fields` **逐类小分量读**（一次全量枚举会闪退）
- 产出：21 个类字段偏移全表（34KB）+ 5 个 hook 真实地址 + 结构步长真值（ObscuredInt=0x14 等）
- 采集环境：SO /health 报 3.28.2、类枚举总量 31420

## 二、两个 "meta" 不是一回事（回应"文件夹里明明有 meta"）

| 文件 | 是什么 | 状态 |
|---|---|---|
| `meta` / `meta-plain.db`（风起文件夹里那俩） | 游戏**资源清单库**（36.8 万条资源路径/CRC/依赖图），sqlite3mc chacha20 加密 | ✅ 9月21日已解密，密钥在账本，哈希校验一致，**跟偏移无关** |
| `global-metadata.dat`（IL2CPP 说明书） | 算偏移**必须要**的文件 | ❌ 官方没放进 APK，**APK 里翻不到** |

我说"没有"指的是后者。所以：**安装包 + meta ≠ 能算偏移**。缺的那块只能从运行中的游戏里拿。

## 三、现成轮子清单（复用，不重造）

| 轮子 | 状态 | 位置 |
|---|---|---|
| APK 归档流程（release 附件绕 100MB 限制） | ✅ 成熟 | uma-apk-archive v686 |
| 688 公开仓 + release | ✅ 今天已建好 | xf8410/uma-jp-688-apk（release v688 空，等文件） |
| 运行时偏移 dump 全流程 + 踩坑记录 | ✅ 10-02 刚跑通 | offset-ledger /2026-10-02 |
| meta 解密（密钥+DDL） | ✅ 已完成 | offset-ledger meta-decrypt-and-schema.txt |
| 离线 CI 解密偏移 | ❌ **从来没成过** | Coneshell + metadata 加密，两条都挡死 |

## 四、当前卡点（就一个）

688 两个 APK 是**空壳**：字节数对（142MB + 82MB）但内容全零，1970 占位时间戳——跟 686 首传一模一样的毛病。
`meta` / `meta-plain.db` 云盘里是完整好文件，**不用重传**。
→ **你只需覆盖重传 688 那两个 APK**，我盯到 zip 头（PK）出现就立刻传 release。

## 五、这次 688 的路线（全部复用老轮子）

1. **你重传 688 两个 APK** → 我验证落盘 → 传 `uma-jp-688-apk` release → 解出 libil2cpp.so 验符号完整性 → 归档
2. **偏移不靠解密 APK**：装回 10-02 那个能用的 SO（不是今天这个禁 hook 的修复版）→ 开嗅探 → 按账本流程逐类 dump → 出 688 偏移全表
3. 拿到偏移全表 → 填进修复版的验证清单（hook_abi 的 verified_profiles）→ 重编 SO → hook 就能挂上、数据就能读到
4. **待确认一句话**：10-02 采集那天，你手机游戏是 688 还是 686？如果是 688 → 账本里 21 个类的偏移直接就是 688 的，第 2 步省一大半
