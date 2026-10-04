# 赛马娘日服 688 拆分 APK 归档

ウマ娘（jp.co.cygames.umamusume）日服 v688 安装包，两个拆分包：

| 文件 | 字节 | 说明 |
|---|---|---|
| jp.co.cygames.umamusume-688-142494628-1790738012.apk | 142,494,628 | 主包 |
| jp.co.cygames.umamusume-688-config.arm64_v8a-82794578-1790738012.apk | 82,794,578 | config.arm64_v8a |

文件放 release 附件（git 单文件 100MB 限制，release 上限 2GB）。

## 用途

- 提取 lib/arm64-v8a/libil2cpp.so（IL2CPP 逆向）
- 跑 CI 解密偏移 / dump 类结构

## 校验（落盘后填）

```
sha256(主包)     = 638b50163639b0f48a093a37e2fb79a196a0681fc8636318132cad93a53348bd（空壳期，重传后更新）
sha256(config包) = 9c5e06093f08b00b7e2802abaf654e8d04c730440d5abae7220c7d88e28c7837（空壳期，重传后更新）
```
