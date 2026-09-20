# Yomika Models

Yomika 原生推理模型文件的独立发布仓库。主程序（[Kelcoin/Yomika](https://github.com/Kelcoin/Yomika)）本体不携带大模型文件；这里以 GitHub Release 分发模型包，仓库本体只维护说明与发布清单。

## 仓库本体

- `manifest.json` — 当前发布清单（tag、资产列表、每个文件的安装路径 / 平铺下载名 / 字节数 / SHA256）。主程序按它下载、校验并安装模型。
- `upscale-manifest.json` — 超分模型的**可选清单**（见下节）。与上面那份不同：那个是一个整包，所有资产装进同一个目录；这份一条目一个可安装的模型。
- `README.md` — 本文件。

发布清单的 schema：

```json
{
  "tag": "models-v1",
  "assets": [
    {
      "role": "detector | inpainting | upscaler | license",
      "path": "mrcnn/weights.onnx",
      "downloadName": "mrcnn--weights.onnx",
      "size": 133568682,
      "sha256": "…64 位十六进制…",
      "companions": []
    }
  ]
}
```

- `path`：安装进模型目录后的相对路径（用 `/` 分隔）。
- `downloadName`：Release 资产的平铺文件名（GitHub Release 保留文件名但不保留目录）。
- 主程序读取清单的顺序：`raw.githubusercontent.com` 上的本体 `manifest.json` → 优先镜像前缀 → Release 资产 `models-v1.manifest.json` → 各种镜像。

## 发布流程

模型源文件（GPL-3.0 Aletheia-Lens 去码包等）保留在主仓库的本地 `docs/` 目录，不入库。发布：

```powershell
# 在主仓库根目录：生成清单（默认 models-fp16）并复制平铺文件
./tools/models/release-manifest.ps1 -Tag models-v1
# 更新本仓库的 manifest.json 后，创建 Release 并上传：
gh release create models-v1 --repo Kelcoin/Yomika-Models --title "…" --notes "…"
gh release upload models-v1 <manifest> <平铺文件…> --repo Kelcoin/Yomika-Models --clobber
```

上传完成后把 Release 里的 `models-v1.manifest.json` 与本体 `manifest.json` 保持一致。

## 超分模型（`upscale-manifest.json`）

漫画走 Real-CUGAN 2x，两项可选：**通用（保守版）** 不改线稿、**漫画（强降噪）** 清噪点与网点。这两项来自官方 2x 的**降噪强度**档位（官方那五档是降噪轴，不是内容轴——五个权重同一套训练集）：通用对应 `conservative`，漫画对应 `denoise3x`。

官方只发 PyTorch `.pth` 与 ncnn、没有 ONNX，所以这里的 ONNX 是**本项目从官方权重导出**的（权重未经改动），导出方式与逐像素比对记录在各模型的 `--provenance.json` 里。主程序在设置页把它们和已安装的模型一起列出来，点一下装到 `models/<id>/<version>/`。

（此前的 Real-ESRGAN 三项已从清单下线。）

清单 schema：

```json
{
  "tag": "models-v1",
  "models": [
    {
      "id": "realcugan-2x-conservative",
      "label": "Real-CUGAN 2x 通用（保守版）",
      "scale": 2,
      "version": "2022.02.27",
      "source": "https://github.com/bilibili/ailab/tree/main/Real-CUGAN",
      "minInputEdge": 20,
      "files": [
        { "name": "realcugan-2x-conservative.onnx", "size": 5272599, "sha256": "…64 位十六进制…" }
      ]
    }
  ]
}
```

- `id`：安装目录名，也是 Release 资产的基名（`<id>.onnx`、`<id>--LICENSE.txt`、`<id>--provenance.json`）。目录里必须有且只有一个 ONNX 文件。
- `scale`：ONNX 输出形状通常是动态的，图自己说不出放大倍数，只能由清单记着。
- `minInputEdge`：图能接受的最小边长（源像素）。Real-CUGAN 图内有 18px 反射 pad，每边小于 20 会直接报错，而分块器在页面右边缘会派发 9px 的细条——引擎按这个值把细条补大再裁回。
- `directUrl`（可选）：上游直链，安装时先试它，再试本仓库的 Release 资产与镜像。当前两项都在本仓库，用不到。
- 读取顺序同 `manifest.json`：`raw.githubusercontent.com` 上的本体文件 → 镜像前缀。

| 模型 | 对应官方档位 | 文件 | 体积 |
|---|---|---|---|
| `realcugan-2x-conservative` | `up2x-latest-conservative.pth` | `realcugan-2x-conservative.onnx` | 5,272,599 |
| `realcugan-2x-denoise3x` | `up2x-latest-denoise3x.pth` | `realcugan-2x-denoise3x.onnx` | 5,272,599 |

Real-CUGAN 为 MIT（Copyright (c) 2022 bilibili）。许可全文随包分发为 `<id>--LICENSE.txt`，来源、导出与校验记在 `<id>--provenance.json`。

## 当前模型（models-v1）

Aletheia-Lens 去码包（四组件齐备才算完整）：

| role | 安装路径 | 来源 |
|---|---|---|
| detector | `mrcnn/weights.onnx` | Aletheia-Lens（FP16 检测器，主仓库转换脚本减半） |
| inpainting | `deepcreampy/bar.onnx`、`deepcreampy/mosaic.onnx` | DeepCreamPy ONNX 导出 |
| upscaler | `esrgan/4x-Fatal-Pixels.onnx`（+`.data` 外部数据） | Aletheia-Lens |
| license | `LICENSE` | Aletheia-Lens GPL-3.0 全文 |

许可证：Aletheia-Lens 为 GPL-3.0，清单附带其全文；模型文件随包分发。waifu2x-upconv7（MIT）内置于主程序，不经本仓库分发。
