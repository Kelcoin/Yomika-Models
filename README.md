# Yomika Models

Yomika 原生推理模型文件的独立发布仓库。主程序（[Kelcoin/Yomika](https://github.com/Kelcoin/Yomika)）本体不携带大模型文件；这里以 GitHub Release 分发模型包，仓库本体只维护说明与发布清单。

## 仓库本体

- `manifest.json` — 当前发布清单（tag、资产列表、每个文件的安装路径 / 平铺下载名 / 字节数 / SHA256）。主程序按它下载、校验并安装模型。
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

## 当前模型（models-v1）

Aletheia-Lens 去码包（四组件齐备才算完整）：

| role | 安装路径 | 来源 |
|---|---|---|
| detector | `mrcnn/weights.onnx` | Aletheia-Lens（FP16 检测器，主仓库转换脚本减半） |
| inpainting | `deepcreampy/bar.onnx`、`deepcreampy/mosaic.onnx` | DeepCreamPy ONNX 导出 |
| upscaler | `esrgan/4x-Fatal-Pixels.onnx`（+`.data` 外部数据） | Aletheia-Lens |
| license | `LICENSE` | Aletheia-Lens GPL-3.0 全文 |

许可证：Aletheia-Lens 为 GPL-3.0，清单附带其全文；模型文件随包分发。waifu2x-upconv7（MIT）内置于主程序，不经本仓库分发。
