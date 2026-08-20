# cell_scan — cellpose 学习项目

> 📚 **学习路径 · 阶段 1/4**：经典图像处理路线。阶段总览见 [`../README.md`](../README.md)。
> 前置知识见 [`../00-prerequisites/`](../00-prerequisites/)。
> 本仓库为「细胞扫描」学习路径的第 1 站：**经典图像处理路线**的细胞分割/计数脚本
> （用于显微镜图像中的细胞/细胞核识别与统计），之后进阶 [Cellpose](../cellpose/) 深度学习分割。

> 定位：这是你「细胞扫描」学习路径的第一站 —— 先理解传统算法如何处理**粘连细胞**这一核心难点，再进阶到 [Cellpose](../cellpose/) 等深度学习方法。

---

## 技术栈

| 类别 | 依赖 | 作用 |
|---|---|---|
| 语言 | Python 3.9+ | — |
| 数值计算 | NumPy | 数组运算 |
| 科学计算 | SciPy | 距离变换 `distance_transform_edt` |
| 图像处理 | scikit-image | 阈值、形态学、连通域、区域测量 |
| 数据表格 | pandas | 结果整理与 CSV 输出 |
| 图像 IO | tifffile | 读取/写出 `.tif` 显微图像 |

## 项目架构

```
cell-segmentation-demo/
├── segment_cells.py      # 主脚本（全部逻辑，单文件）
├── requirements.txt      # 依赖清单
├── data/                 # 放入你的输入图片
├── output/               # 输出的 CSV / 标签图
└── README.md             # 本文档
```

### 数据处理流水线

```
输入图像 (.tif/.tiff/.png)
    │
    ▼
① 通道提取     ── 多通道图取指定通道 / RGB 转灰度
    │
    ▼
② 阈值分割     ── Otsu / Li / Triangle（分离前景与背景）
    │
    ▼
③ 去除小物体   ── 过滤噪声（min_area）
    │
    ├── 方法一：basic（简单阈值法，适合分离良好的细胞）
    │       连通域标记 → 直接得到每个细胞
    │
    └── 方法二：watershed（分水岭，适合粘连细胞）★默认
            ④ 距离变换 → ⑤ 局部极大值找细胞中心
            → ⑥ 以中心为标记做分水岭分割
    │
    ▼
⑦ 区域测量     ── 面积、平均强度、质心、长短轴、偏心率
    │
    ▼
⑧ 结果输出     ── cell_counts.csv + 可选保存标签图
```

**核心难点**：当细胞相互粘连时，简单的连通域会把多个细胞算成一个。
`watershed` 方法通过距离变换找每个细胞的"中心种子"，再让区域从种子向外生长到彼此边界，从而把粘连细胞切开。

## 快速开始

```bash
# 0. 仓库自带示例图 data/sample_cells.tif，克隆后可直接运行：
python segment_cells.py data/sample_cells.tif --save-labels -o output/result.csv

# 1. 创建虚拟环境（建议）
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # macOS / Linux

# 2. 安装依赖
pip install -r requirements.txt

# 3. 单张图片分割（默认分水岭）
python segment_cells.py data/你的图片.tif

# 4. 批量处理整个文件夹
python segment_cells.py data/ --batch --save-labels

# 5. 切换算法 / 调参
python segment_cells.py data/img.tif --method basic --threshold otsu --min-area 50
python segment_cells.py data/img.tif --method watershed --min-distance 10 --channel 0
```

结果输出到 `cell_counts.csv`（也可用 `-o 自定义.csv` 指定），包含每张图的：
细胞数 `cell_count`、平均面积 `mean_area`、面积标准差 `std_area`、平均强度 `mean_intensity`。

## 命令行参数参考

| 参数 | 默认值 | 说明 |
|---|---|---|
| `input` | 必填 | 图片文件 或 文件夹路径 |
| `-o, --output` | `cell_counts.csv` | 结果 CSV 路径 |
| `--method` | `watershed` | `watershed`（粘连细胞）或 `basic`（分离良好） |
| `-c, --channel` | 无 | 多通道图的通道索引 |
| `--min-area` | `50` | 最小细胞面积（像素），滤噪声 |
| `--min-distance` | `10` | 分水岭：细胞中心最小间距（像素） |
| `--threshold` | `otsu` | 阈值法：`otsu` / `li` / `triangle` / 数值 |
| `--save-labels` | 关 | 同时保存每个细胞编号的标签图（`_labels.tif`） |
| `--batch` | 关 | 批量模式，处理整个文件夹 |

## 获取练习数据

- 最快：运行隔壁 [Segmentum](../Segmentum/) 的 `python generate_samples.py` 生成示例细胞图像
- 公开数据集：Broad Bioimage Benchmark Collection（[BBBC](https://bbbc.broadinstitute.org/)）提供标准显微图像
- 自己的数据：用 [labelme](https://github.com/wkentaro/labelme) 标注后训练更精准的深度学习模型（见下）

## 下一步（深度学习进阶）

传统分水岭对噪声、密度高、形态复杂的图像鲁棒性有限。当你理解本脚本后，推荐：

1. 阅读 [cellpose/notebooks/Cellpose_cell_segmentation_2D_prediction_only.ipynb](../cellpose/notebooks/Cellpose_cell_segmentation_2D_prediction_only.ipynb)
2. 对比思考：Cellpose 用 **U-Net 预测梯度流（flow fields）** 把像素聚类成实例，与这里的距离变换+分水岭是同一思路的"可学习版本"
3. 用 [cell-segmentation-pipeline](../cell-segmentation-pipeline/) 的 Notebook 走完整深度学习流水线
