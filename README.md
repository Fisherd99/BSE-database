# BSE-database

Si 体系 ABACUS + LibRPA 双重 k 网格 GW/BSE 计算的输入与日志归档。

本批次包含粗（稀疏）k 网格 **7³、8³、9³**（频率数 `nfreq=16`，即 nf16）三套数据，
统一设置 `use_shrink_chi=T`、`use_shrink_abfs=T`，在稀疏网格上完成 GW，
再插值到密集 **14³** 网格计算单重态 TDA BSE（4 占据带 + 4 非占据带，共 43904 态）。

## 目录结构

```
data/
  k777_nf16/
  k888_nf16/
  k999_nf16/
    gw/                     GW(LibRPA) 阶段输入、能量表与日志
      KS_band_spin_1.dat     KS 能带（14³，spin 1）
      EXX_band_spin_1.dat    精确交换能带（14³，spin 1）
      GW_band_spin_1.dat     GW 能带（14³，spin 1）
      energy_qp              QP 能量（无扩展名）
      librpa.in              LibRPA 输入参数
      librpajob.log          LibRPA rank0 主标准输出
    bse/                    BSE(ABACUS 内嵌 LibRPA) 阶段输入与日志
      INPUT                  ABACUS 输入卡（另含一份内容相同的 INPUT_bse）
      KPT                    k 网格
      STRU                   结构
      Si_gga_8au_100Ry_3s3p2d.orb                 数值原子轨道基组
      Si_ONCV_PBE-1.0.upf                        模守恒赝势
      Si_3s3p2d1f1g_pca1e-6.abfs                 ABF（辅助）基组
      abacusjob-bse.log      BSE 步 ABACUS 标准输出（含内嵌 LibRPA 信息）
```

每个网格共 13 个文件，三套合计 39 个文件。

## 说明

- `KS/EXX/GW_band_spin_1.dat` 为同一 14³ 能量表的 `.dat` 版本，
  与运行目录 `OUT.librpa/` 内同名 `.txt` 内容逐字节相同。
- 体积较大的 LibRPA MPI 分片详细日志 `librpa_para_nprocs_16_myid_*.out`
  （每套 16 个）未纳入本仓库；rank0 主日志 `gw/librpajob.log` 已包含 LibRPA 主标准输出。
- **8³/nf16 的 BSE 阶段通过续算完成**（原 run 的严格时间戳校验未过），
  因此 `data/k888_nf16/bse/` 的输入与 `abacusjob-bse.log` 取自续算目录
  `continuations/k888-nf16-selected8-from-r01-cont01/bse/`；
  其 `gw/` 能量表与日志取自原 run `runs/k888-nf16-shrinkchiT-r01/gw/`，
  两处的 14³ 能量表内容一致。7³、9³ 的 gw 与 bse 文件均取自各自原 run 目录。
- 归档时已将原运行目录中的符号链接解引用为实体文件。

## 校验

每个文件的 SHA256、字节数与相对路径见 [`MANIFEST.sha256`](MANIFEST.sha256)。
可用 `sha256sum -c MANIFEST.sha256` 校验。

## 来源环境

- ABACUS v3.11.0-beta10
- LibRPA（CMake 信息见各 `librpajob.log`）
- 16 MPI rank × 8 OpenMP 线程
