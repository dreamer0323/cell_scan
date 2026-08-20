# 输入数据目录

把你要处理的显微镜图片（`.tif` / `.tiff` / `.png`）放在这里。

## 获取练习图片

1. **最快（本地生成）**：运行隔壁 `Segmentum` 项目的示例生成脚本，会产出带细胞的模拟图像：
   ```bash
   cd ../Segmentum
   pip install -r requirements.txt
   python generate_samples.py
   ```
   然后把生成的图片复制到这里。

2. **公开数据集**：Broad Bioimage Benchmark Collection
   - 网站：https://bbbc.broadinstitute.org/
   - 适合入门的：`BBBC005`（细胞核分割）、`BBBC019`、`BBBC039` 等

3. **skimage 自带示例**（先快速验证流程）：
   ```python
   from skimage import data
   from skimage.io import imsave
   img = data.cell()  # 细胞图像（灰度）
   imsave('data/example_cells.tif', img)
   ```

> 本仓库已内置一张示例图 `data/sample_cells.tif`（来自 skimage 自带数据），
> 克隆下来后可直接运行 `python segment_cells.py data/sample_cells.tif` 体验完整流程。

## 输出

处理结果（CSV、标签图）会写到 `../output/`（或当前目录，取决于 `-o` 参数）。
