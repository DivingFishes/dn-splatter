# DN-Splatter 快速上手指南

这份指南面向“先把数据下载下来，再尽快跑通一次实验验证”的场景。优先路径是 `pixi`，因为仓库已经把示例下载和训练流程写成了任务。

## 1. 推荐的最短路径

如果你只想先验证项目能否正常运行，按这个顺序做：

```bash
pixi install
pixi run example
```

`example` 会串起三个步骤：

1. 下载 Omnidata 的法线权重。
2. 下载一个 MuSHRoom 的示例数据集 `koivu_iphone`。
3. 启动一次 `dn-splatter` 训练。

如果你更习惯 Conda，也可以按 README 的传统方式安装：

```bash
conda activate nerfstudio
pip install setuptools==69.5.1
pip install -e .
```

## 2. 仓库里最重要的入口

这个项目的核心入口可以按用途理解：

1. 训练方法注册在 `dn_splatter/dn_config.py`。
2. 数据集解析器在 `dn_splatter/data/`。
3. 数据下载脚本在 `dn_splatter/data/download_scripts/`。
4. 法线、深度、网格和评估脚本分别在 `dn_splatter/scripts/` 和 `dn_splatter/eval/`。
5. 网格导出命令由 `gs-mesh` 暴露，实际实现位于 `dn_splatter/export_mesh.py`。

## 3. 环境要求

仓库依赖的关键版本大致是：

1. Python 3.10。
2. `nerfstudio == 1.1.3`。
3. `gsplat == 1.0.0`。
4. CUDA 11.8 / PyTorch 2.2.x 这一套组合更贴近 `pixi.toml`。

如果你用 `pixi`，这些依赖基本都会自动处理；如果你用现成的 `nerfstudio` 环境，至少要保证 `ns-train`、`ns-eval`、`ns-process-data` 这些命令可用。

## 4. 数据下载

### 4.1 Omnidata 法线权重

```bash
python dn_splatter/data/download_scripts/download_omnidata.py
```

默认会下载到 `omnidata_ckpt/omnidata_dpt_normal_v2.ckpt`。

### 4.2 MuSHRoom 示例数据

```bash
python dn_splatter/data/download_scripts/mushroom_download.py --room-name koivu --sequence iphone
```

默认会放到当前目录下的 `datasets/`，并解压成对应房间的 `iphone` / `kinect` / `faro` 目录。

### 4.3 其他数据集

仓库除了 MuSHRoom 示例外，还明确支持这些数据源：

1. `Replica`：室内 RGB-D 数据集，对应 `replica` dataparser。
2. `ScanNet++`：室内采集数据集，对应 `scannetpp` dataparser。
3. `Neural-RGBD`：对应 `nrgbd` dataparser。
4. `SDFStudio / DTU`：对应 `gsdf` dataparser。
5. `COLMAP / 自定义 Nerfstudio 数据`：对应 `coolermap` 或 `normal-nerfstudio` dataparser。

常用下载/准备脚本如下：

1. `python dn_splatter/data/download_scripts/replica_download.py`
2. `python dn_splatter/data/download_scripts/nrgbd_download.py`
3. `python dn_splatter/data/download_scripts/dtu_download.py`
4. `python dn_splatter/data/mushroom_utils/reference_depth_download.py`

如果你已经有自己的数据，通常不用这些脚本，直接按 Nerfstudio 的数据约定整理即可；如果是原始图片集，优先走 `coolermap`。

### 4.4 项目里实际用到的数据集

这个仓库的代码和示例里，最常见的验证路径是：

1. `MuSHRoom`：仓库最推荐的示例数据，`pixi run example` 默认就走它。
2. `Replica`：用于室内场景训练与评估。
3. `ScanNet++`：用于更大的真实室内场景。
4. `Neural-RGBD`：用于带 RGB-D 的数据流程。
5. `DTU / SDFStudio`：用于 `gsdf` 流程。
6. `COLMAP` 数据：任意已经做过 SfM 的图片集，走 `coolermap`。
7. `Nerfstudio` 标准数据格式：走 `normal-nerfstudio`。

如果你的目标是“先下载一个能跑通的样例”，优先用 MuSHRoom；如果你的目标是“验证自己的数据”，优先判断它是否可以直接映射到 `coolermap`、`mushroom` 或 `normal-nerfstudio`。

### 4.5 MuSHRoom 跑通验证 Todolist

如果你的目标是先在 MuSHRoom 上跑通一次验证，建议按这个顺序做：

1. 确认环境可用：`conda activate dn-splatter`，然后检查 `ns-train` 是否在 PATH 中。
2. 下载 MuSHRoom 数据集：优先先下 `koivu` 的 `iphone` 序列；如果你准备跑更完整的流程，再补 `kinect` 或 `faro`。
3. 下载 Omnidata 法线先验：这是法线监督最常见的预处理依赖。
4. 可选：下载 Faro reference depth。只有当你要用 MuSHRoom 的 Faro 扫描参考深度时才需要。
5. 可选：如果你想用 iPhone 序列的 COLMAP 初始化点云，再跑 `poses_to_colmap_sfm.py` 生成或补齐初始点云。
6. 不需要单独下载 DN-Splatter 的训练模型权重。这个项目的常规验证是直接用代码和数据跑 `ns-train`，不是先拿一个官方 checkpoint 再微调。
7. 先用最小训练命令跑通一轮，再考虑网格导出和评估。

最小验证优先级可以理解成：数据集 > Omnidata 法线先验 > 训练命令 > 可选的 Faro / COLMAP 辅助步骤。

### 4.6 当前这次验证的实际进展

如果你现在要继续推进“先在 MuSHRoom 上跑通验证”这个任务，可以把当前状态理解成下面这样：

1. 已完成环境配置，`ns-train`、`ns-eval` 等基础命令可用。
2. 已检查并整理了 MuSHRoom 备份数据的目录结构，当前使用的是 `activity` 这份本地数据。
3. 已完成四个 capture 的 `normals_from_depth` 预处理。
4. 已重新下载并校验 ZoeDepth 权重，单目深度预处理可以继续跑。
5. `mono_depth` 仍在生成中，当前还没有成功落盘。

当前还剩下这些事情：

1. 等待 `mono_depth` 对四个 capture 全部生成完成。
2. 复查 `mono_depth` 文件数量是否和图片数量对齐。
3. 如果 `depth` 目录里存在异常帧，再决定是否需要补齐或剔除。
4. 确认后再进入训练验证阶段，也就是启动一次最小 `ns-train`。

当前这份备份数据的状态可以概括为：`images` 和 `depth` 已基本齐备，`normals_from_depth` 已生成完成，但 `mono_depth` 还未生成成功。

## 5. 预处理常用步骤

### 5.1 生成单目法线

仓库支持两种常见法线来源：Omnidata 和 DSINE。

Omnidata：

```bash
python dn_splatter/scripts/normals_from_pretrain.py \
  --data-dir [DATA_ROOT] \
  --resolution low
```

DSINE：

```bash
python dn_splatter/scripts/normals_from_pretrain.py \
  --data-dir [DATA_ROOT] \
  --normal-format dsine
```

默认输出通常会放在 `[DATA_ROOT]/normals_from_pretrain/`。

### 5.2 把图片集转换成 COLMAP 格式

如果数据没有相机位姿，可以先做 COLMAP：

```bash
python dn_splatter/scripts/convert_colmap.py --image-path [DATA_ROOT/images] --use-gpu
```

### 5.3 对齐单目深度

当你已经有 COLMAP 结果，并且想生成可用于训练的尺度对齐深度，可以用：

```bash
python dn_splatter/scripts/align_depth.py --data [DATA_ROOT]
```

如果你只想生成单目深度而不做 SfM 对齐：

```bash
python dn_splatter/scripts/align_depth.py --data [DATA_ROOT] --skip-colmap-to-depths --skip_alignment
```

## 6. 训练验证

### 6.1 通用训练模板

```bash
ns-train dn-splatter \
  --pipeline.model.use-depth-loss True \
  --pipeline.model.depth-lambda 0.2 \
  --pipeline.model.use-normal-loss True \
  --pipeline.model.normal-supervision mono \
  [DATAPARSER] --data [DATA_ROOT]
```

### 6.2 常见 dataparser

1. `mushroom`：MuSHRoom。
2. `replica`：Replica。
3. `scannetpp`：ScanNet++。
4. `nrgbd`：Neural-RGBD。
5. `gsdf`：SDFStudio / DTU。
6. `coolermap`：通用 COLMAP 数据。
7. `normal-nerfstudio`：更通用的 Nerfstudio 数据格式。

### 6.3 推荐的最小验证命令

如果你想快速验证训练链路，优先用仓库自带的示例任务：

```bash
pixi run example
```

如果你已经下载了 `koivu_iphone` 示例，也可以直接自己跑：

```bash
ns-train dn-splatter \
  --pipeline.model.use-depth-loss True \
  --pipeline.model.depth-lambda 0.2 \
  --pipeline.model.use-depth-smooth-loss True \
  --pipeline.model.use-normal-loss True \
  --pipeline.model.normal-supervision mono \
  mushroom --data datasets/room_datasets/koivu --mode iphone
```

### 6.4 AGS-Mesh

AGS-Mesh 是仓库里的另一个方法入口，训练方式类似，只是方法名换成 `ags-mesh`：

```bash
ns-train ags-mesh \
  --pipeline.model.use-depth-loss True \
  --pipeline.model.depth-lambda 0.2 \
  --pipeline.model.use-normal-loss True \
  --pipeline.model.normal-supervision mono \
  [DATAPARSER] --data [DATA_ROOT] \
  --load-depth-confidence-masks True
```

### 6.5 dn-splatter-big

如果你想尝试更大的高斯数版本，把方法名换成 `dn-splatter-big` 即可。

## 7. 网格导出

仓库提供 `gs-mesh` 命令，推荐优先试 `o3dtsdf`：

```bash
gs-mesh o3dtsdf --load-config [PATH_TO_CONFIG] --output-dir [OUTPUT_DIR]
```

其他可选项包括 `dn`、`tsdf`、`sugar-coarse`、`gaussians`、`marching`。

如果你更关心平滑网格，也可以直接用 IsoOctree 脚本：

```bash
python dn_splatter/scripts/isooctree_dn.py <root_folder> \
  --transformation_path <pose_json_path> \
  --output_mesh_file <output_path/output.ply>
```

## 8. 评估

### 8.1 标准图像 / 深度评估

```bash
ns-eval --load-config [PATH_TO_CONFIG] --output-path [JSON_OUTPUT_PATH]
```

如果还要把渲染结果输出出来，可以再加：

```bash
--render-output-path [PATH_TO_IMAGES]
```

### 8.2 网格评估

MuSHRoom：

```bash
python dn_splatter/eval/eval_mesh_mushroom_vis_cull.py \
  --gt_mesh_path [GT_MESH] \
  --pred_mesh_path [PRED_MESH] \
  --device [iphone/kinect]
```

其他数据集或自定义数据：

```bash
python dn_splatter/eval/eval_mesh_vis_cull.py \
  --gt-mesh-path [GT_MESH] \
  --pred-mesh-path [PRED_MESH] \
  --transformation_file [TRANSFORM_JSON] \
  --dataset_path [DATASET_DIR]
```

## 9. 实际使用时最容易踩的坑

1. `normal-format` 要和法线来源匹配，`omnidata` 和 `dsine` 的坐标系不一样。
2. 训练法线监督前，先确认法线文件已经生成到对应数据目录里。
3. `align_depth.py` 依赖 COLMAP 输出，如果没有 `colmap/sparse/0` 结构，直接跑通常会失败。
4. 如果你在深度对齐脚本里遇到 Torch 的尺寸类型报错，README 提到过降级到 Torch 2.0.1 可能有帮助。
5. 只想先确认环境没问题，优先跑 `pixi run example`，比直接上大数据集更稳。

## 10. 建议的操作顺序

如果你的目标是“下载数据并验证实验”，建议按这个顺序做：

1. `pixi install`
2. `pixi run example`
3. 如果示例正常，再切换到你的数据集下载脚本或自有数据目录。
4. 先跑一次 `ns-train`，再做 `ns-eval` 和 `gs-mesh`。
