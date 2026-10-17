# Yomika Models

Yomika 原生推理模型文件的独立发布仓库。主程序（[Kelcoin/Yomika](https://github.com/Kelcoin/Yomika)）本体不携带大模型文件；模型包以 GitHub Release 分发，标签固定为 `models-v1`。仓库本体只维护这份发布说明与两份清单，不存放模型文件本身。

## 仓库本体

- `upscale-manifest.json`：超分模型清单。主程序按它列出可在线下载的超分模型，逐文件校验后装进 `models/<id>/<version>/`。
- `dlssnr-manifest.json`：DLSSNR 片段清单。主程序按它列出可下载的片段变体，校验后把选中的那一份装成数据目录下的 `dlssnr/nvngx_dlssnr.dll`。
- `README.md`：本文件。

主程序读取清单的顺序：`raw.githubusercontent.com` 上的仓库正文原始文件 → 依次套用镜像前缀（`gh-proxy.com`、`ghfast.top`、`ghproxy.net`）。清单里的 `tag` 决定模型资产从哪个 Release 取；资产下载同样先直连、再逐个镜像，并且落盘的字节必须与清单里的 `size` / `sha256` 完全一致，对不上就不安装。清单以仓库正文那一份为准；如果另外把副本挂进了 Release，那份副本必须与正文逐字节一致（主程序不读这份副本）。

### `upscale-manifest.json` 的字段

```json
{
  "tag": "models-v1",
  "models": [
    {
      "id": "…",
      "label": "设置页显示的名字",
      "scale": 2,
      "version": "…",
      "source": "上游出处，只作记录",
      "minInputEdge": 20,
      "files": [
        { "name": "…", "size": 0, "sha256": "…64 位十六进制…" }
      ]
    }
  ]
}
```

- `tag`、`models` 必填，`models` 不能为空。
- `id`：同时是安装目录名；`version` 也是目录名的一部分。两者都必须是安全的单段路径。
- `label`、`scale`：设置页显示用；`scale` 会写进安装后的 `yomika-model.json`。ONNX 的输出形状是动态的，从文件本身问不出放大倍数，所以只能写在清单里。
- `source`、`minInputEdge`、`files[].directUrl`：可选。`directUrl` 是上游直链，必须先于本仓库的 Release 资产尝试；`minInputEdge` 是图能接受的最小输入边长。
- `files[].name`：既是安装进模型目录的文件名，也是 Release 资产的平铺文件名；`size` 必须大于 0，`sha256` 必须是 64 位十六进制。
- 一个模型必须且只能列出 1 份 `.onnx` 图，同一模型内的文件名不得重复。

### `dlssnr-manifest.json` 的字段

```json
{
  "tag": "models-v1",
  "license": { "name": "…", "asset": "…", "size": 0, "sha256": "…" },
  "variants": [
    {
      "id": "…",
      "label": "…",
      "note": "…",
      "series": [50],
      "asset": "…",
      "size": 0,
      "sha256": "…"
    }
  ]
}
```

- `tag`、`variants` 必填，`variants` 不能为空。
- `variants[].id`：变体标识（`official` / `community` / `sf-v2`），不得重复；`label` 不能为空。
- `variants[].asset`：Release 资产的平铺文件名，下载时也当落盘名用，必须是安全的单段路径。
- `variants[].note`、`variants[].series`：只作说明与推荐提示，`series` 写这一条面向哪几代卡。
- `license`：可选。片段内嵌的厂商许可里要求随包分发全文的那一份，字段与变体一致（`name` / `asset` / `size` / `sha256`）。

## 当前发布内容（models-v1）

### 超分模型（`upscale-manifest.json`）

两个模型都取自 bilibili/ailab 的 Real-CUGAN（`https://github.com/bilibili/ailab/tree/main/Real-CUGAN`），版本 `2022.02.27-dml1`，每套三件：ONNX 图、许可全文、DML 改写记录。

| 模型 | 说明 | 文件（= 平铺下载名） | 字节数 | SHA256 |
|---|---|---|---|---|
| `realcugan-2x-conservative` | Real-CUGAN 2x 通用（保守版） | `realcugan-2x-conservative-dml1.onnx` | 5270956 | `727f94a02e30bfbeffb9033e9cf8703ebc73757c87ed006095ca2769746e8a5f` |
| `realcugan-2x-conservative` | 许可全文 | `realcugan-2x-conservative--LICENSE.txt` | 1065 | `8cad8cfdf94baaf23519061af913770e52476ddec2a311e9510582e7bed13cba` |
| `realcugan-2x-conservative` | DML 改写记录 | `realcugan-2x-conservative-dml1--provenance.json` | 3024 | `84bb65a93aa72eef9a906ee85bded4bdbd9b23c65e2baadcfbcc11f4f971419b` |
| `realcugan-2x-denoise3x` | Real-CUGAN 2x 漫画（强降噪） | `realcugan-2x-denoise3x-dml1.onnx` | 5270956 | `931c20fa7e701e118e67602157bfa9fa97f8da0f9a74222ab2e16ea71172a833` |
| `realcugan-2x-denoise3x` | 许可全文 | `realcugan-2x-denoise3x--LICENSE.txt` | 1065 | `8cad8cfdf94baaf23519061af913770e52476ddec2a311e9510582e7bed13cba` |
| `realcugan-2x-denoise3x` | DML 改写记录 | `realcugan-2x-denoise3x-dml1--provenance.json` | 3015 | `27676fd66169f0ededf714aea0cd44ab0490b9a6ad5a87f51eea65ab8e190576` |

### DLSSNR 片段（`dlssnr-manifest.json`）

三份的文件版本各不相同：官方 310.8.0.0（签名有效）、社区 310.8.3.0（Lecram 改写，签名哈希对不上）、FP16 310.9.16.0（社区 compat-fp16，未签名）。下载后落到数据目录的 `dlssnr/nvngx_dlssnr.dll`。

| 变体 | 说明 | 文件（= 平铺下载名） | 字节数 | SHA256 |
|---|---|---|---|---|
| `official` | NVIDIA 官方原版（RTX 50 系原生） | `nvngx_dlssnr--official.dll` | 165840496 | `e16bcf15e16e13f527491cdf7845b2fe6521a738d8f7c9c721866a8496e1fc8e` |
| `community` | 社区兼容版（RTX 40 / 50 系，Lecram 310.8.3） | `nvngx_dlssnr--community-rtx40-50.dll` | 165840496 | `f95feb54137ea11979f9b4ec4f00afd84b5c98a5624d3388fbf6a87714a39fcc` |
| `sf-v2` | FP16 兼容版（RTX 20 / 30 系，社区 compat-fp16 310.9.16） | `nvngx_dlssnr--sf-v2.dll` | 276162560 | `cdfa6c1ec784867182164c7836f324f087b5c9f374d0c9fe46f54999feabf3ed` |
| 许可 | NVIDIA DLSS SDK License | `nvidia-dlss-license.txt` | 26620 | `d4216e39ebef5f9b50a6712ebb37beeb5379862a67733a9999c651f21592aaf0` |

## 刻意不在这里的内容

- waifu2x-upconv7（MIT）：内置于主程序，不经本仓库分发。
- Anime4K：是编译进主程序的着色器链，不是模型文件，清单里没有它。
- 去码整档：Aletheia-Lens 去码包已于 2026-10-14 随该功能下线退役，不再发布；它原先那份打包清单也已从本仓库删除。

## 发布流程

### 源头

超分模型与它们的两份记录放在主仓库的 `docs/models/`（本地工作目录，不进主仓库提交），超分清单以 `docs/models/upscale-manifest.json` 为准。DLSSNR 清单以主仓库 `src-tauri/resources/dlssnr/dlssnr-manifest.json` 为准，它同时被主程序用 `include_str!` 编成内置副本，改这里等于改主程序。DLSSNR 的三份片段二进制在主仓库 `docs/DLSSNR-DLL-Options-310.8.0.0/`，NVIDIA 许可正本在 `src-tauri/vendor/ngx/NVIDIA-DLSS-LICENSE.txt`。

### 平铺下载名

GitHub Release 保留文件名，不保留目录，所以每份资产在 Release 里都必须是平铺且唯一的名字：

- 超分：清单 `files[].name` 本身就是平铺名，例如 `realcugan-2x-conservative-dml1.onnx`、`realcugan-2x-conservative--LICENSE.txt`。清单没有单独的 `downloadName` 字段，名字一开始就要取平铺形式，`--` 只用来把模型名与记录类型分开。
- DLSSNR：清单 `variants[].asset` 与 `license.asset` 就是平铺名，`--` 把原名与变体名分开。

### 命令

```powershell
# 1. 两份清单以主仓库为准，同步到本仓库正文（逐字节一致），然后提交到 main：
#    主仓库 docs\models\upscale-manifest.json                 -> 本仓库 upscale-manifest.json
#    主仓库 src-tauri\resources\dlssnr\dlssnr-manifest.json    -> 本仓库 dlssnr-manifest.json

# 2. 创建 Release（已存在时跳过这一步）
gh release create models-v1 --repo Kelcoin/Yomika-Models `
  --title "Yomika 模型包 models-v1" `
  --notes "Yomika 超分模型与 DLSSNR 片段"

# 3. 上传当前全部平铺资产（在放平铺文件的目录里执行，文件名必须与上表一致）
gh release upload models-v1 --repo Kelcoin/Yomika-Models --clobber `
  realcugan-2x-conservative-dml1.onnx `
  realcugan-2x-conservative--LICENSE.txt `
  realcugan-2x-conservative-dml1--provenance.json `
  realcugan-2x-denoise3x-dml1.onnx `
  realcugan-2x-denoise3x--LICENSE.txt `
  realcugan-2x-denoise3x-dml1--provenance.json `
  nvngx_dlssnr--official.dll `
  nvngx_dlssnr--community-rtx40-50.dll `
  nvngx_dlssnr--sf-v2.dll `
  nvidia-dlss-license.txt
```

### 发布后自检

```powershell
# 每个资产算一遍 SHA256，逐条与清单对
Get-FileHash -Algorithm SHA256 .\realcugan-2x-conservative-dml1.onnx
# Release 里的资产名与数量
gh release view models-v1 --repo Kelcoin/Yomika-Models
```

许可证：Real-CUGAN 的许可见各模型目录下的 `--LICENSE.txt`，随包分发；DLSSNR 片段内嵌 NVIDIA DLSS SDK 许可，全文以 `nvidia-dlss-license.txt` 随包分发。waifu2x-upconv7（MIT）内置于主程序，不经本仓库分发。
