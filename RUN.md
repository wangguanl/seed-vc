# 运行命令

- 项目：Seed-VC（零样本语音/歌声转换 + 实时 GUI）
- 生成时间：2026-09-08
- 运行方式：直接运行（uv + Python 3.10 + Gradio）
- 硬件评估：**满足**（启动后空闲约占 **~3.5GB** 显存装 V2；V1 首次推理再懒加载；策略：先开最强，OOM 再降级）
- 当前实例：`http://127.0.0.1:7861/`（7860 被占用已顺延）

## 环境准备

```powershell
cd E:\AI\local-voice\seed-vc
$env:UV_INDEX_URL = 'https://pypi.tuna.tsinghua.edu.cn/simple'
$env:HF_ENDPOINT = 'https://hf-mirror.com'
$env:Path = "E:\Programs\ffmpeg-master-latest-win64-gpl\bin;$env:Path"

uv venv --python 3.10
# 必须用带 +cu124 的 Windows 轮子直链；仅 --index-url 时 uv 常会解析成 2.4.0+cpu
uv pip install --reinstall `
  "https://download.pytorch.org/whl/cu124/torch-2.4.0%2Bcu124-cp310-cp310-win_amd64.whl" `
  "https://download.pytorch.org/whl/cu124/torchvision-0.19.0%2Bcu124-cp310-cp310-win_amd64.whl" `
  "https://download.pytorch.org/whl/cu124/torchaudio-2.4.0%2Bcu124-cp310-cp310-win_amd64.whl"
uv pip install -r requirements-run.txt
uv pip install numpy==1.26.4
```

说明：`requirements.txt` 混有 nightly 伪行与稳定版钉死，本机用 `requirements-run.txt`（去掉冲突的 torch 行）。装完依赖后务必确认 `torch.__version__` 含 `+cu124` 且 `torch.cuda.is_available()` 为 True。

## 启动

- 推荐：`pwsh -NoProfile -File .\start.ps1`
- 说明：菜单默认 **[1] 集成 UI（V1+V2，最强）**；OOM 时改开仅 V2 / 仅 V1 VC
- 等价手动命令：

```powershell
$env:HF_ENDPOINT = 'https://hf-mirror.com'
$env:GRADIO_SERVER_PORT = '7860'   # 占用则换端口
# [1] 最强：集成
uv run python app.py --enable-v1 --enable-v2
# [2] 降一级：仅 V2
uv run python app_vc_v2.py
# [3] V1 语音
uv run python app_vc.py --fp16 True
# [4] V1 歌声
uv run python app_svc.py --fp16 True
# [5] 实时 tiny GUI
uv run python real-time-gui.py
```

## 验证

- 浏览器打开日志中的 `http://127.0.0.1:<port>/`
- 用 `examples/source` + `examples/reference` 各选一段，转换成功且无 CUDA OOM
- `nvidia-smi` 观察峰值显存

## 备注

- 镜像：HF=`https://hf-mirror.com`；PyPI=`清华`；torch=`download.pytorch.org/whl/cu124`
- ffmpeg：`E:\Programs\ffmpeg-master-latest-win64-gpl\bin`
- 降级顺序：集成 V1+V2 → 仅 V2 → V1 VC → SVC/实时
- 本文件与 `start.ps1` / `requirements-run.txt` 为本机记录，默认不提交 git
