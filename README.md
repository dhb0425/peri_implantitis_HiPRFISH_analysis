# Microscale Spatial Dysbiosis in Oral biofilms Associated with Disease

This repository contains the specific code used to generate the figures in "Microscale Spatial Dysbiosis in Oral biofilms Associated with Disease"

## Acknowledgement

This code makes use of open source packages including `numpy`, `pandas`, `aicsimageio`, `scikit-image`, `scikit-learn`, `PySAL`, `OpenCV`, `scipy`, `turbustat`, `umap`,`networkx`, `numba`, `xml`, `cython`, and `matplotlib`.

This code also makes use of [code](https://github.com/proudquartz/hiprfish) developed in [Shi, et al. 2020](https://doi.org/10.1038/s41586-020-2983-4). 

## HPC Setup (Linux/Slurm — Recommended)

The conda environments in this repo were built on Linux and are **not compatible with macOS** (Linux-specific build hashes, CUDA dependency). Run on a Linux HPC cluster.

### Step-by-step on HPC

**1. Clone the repo**
```bash
git clone https://github.com/benjamingrodner/peri_implantitis_HiPRFISH_analysis.git
cd peri_implantitis_HiPRFISH_analysis
```

**2. Place your CZI image files**

Put all `.czi` files for a sample in `data/`. Each sample needs one file per laser channel (488, 514, 561 nm — and optionally 633 nm for 7-bit barcoding). Example for a 3-channel sample:
```
data/
  2022_12_16_harvardwelch_patient_19_tooth_30_aspect_MB_depth_sub_fov_01_las_488.czi
  2022_12_16_harvardwelch_patient_19_tooth_30_aspect_MB_depth_sub_fov_01_las_514.czi
  2022_12_16_harvardwelch_patient_19_tooth_30_aspect_MB_depth_sub_fov_01_las_561.czi
```

Full datasets are available on Zenodo (see Data Availability in the paper).

**3. Edit `env_image_processing.yml` before creating the environment**

Remove the last line (`prefix: /home/bmg224/...`) and remove `cudatoolkit=10.0.130=0` from the dependencies (only needed for GPU; the segment step runs on CPU).

**4. Create the conda environment**
```bash
conda env create -f env_image_processing.yml
conda activate hiprfish_imaging_py38
```

**5. Compile the Cython neighbor detection module**
```bash
cd functions
cythonize -i neighbor2d.pyx
cd ..
```

**6. Update `input_table_test.csv`**

List one CZI file path per line (one row per channel file). Paths are relative to the repo root. Example:
```
filenames
data/2022_12_16_harvardwelch_patient_19_tooth_30_aspect_MB_depth_sub_fov_01_las_488.czi
data/2022_12_16_harvardwelch_patient_19_tooth_30_aspect_MB_depth_sub_fov_01_las_514.czi
data/2022_12_16_harvardwelch_patient_19_tooth_30_aspect_MB_depth_sub_fov_01_las_561.czi
```

**7. Update `config.yaml`**

Key fields to verify:
- `input_table: input_table_test.csv`
- `output_dir: ../outputs` (results saved one level above repo root)
- `ref_dir: ../HiPRFISH_reference_spectra` (required for classification — see below)

**8. Create output directory**
```bash
mkdir -p ../outputs
```

### Running the pipeline

**Segmentation only (no reference spectra needed):**

Produces segmentation masks (`.npy`), cell properties + raw spectra (`.csv`), and visualization images (`.png`) in `../outputs/`.
```bash
snakemake -s Snakefile_image_processing --configfile config.yaml -j NUM_CORES -p --until segment
```

**Full image processing (requires HiPRFISH reference spectra):**

Reference spectra are not included in this repo. Obtain from the [hiprfish repo](https://github.com/proudquartz/hiprfish) or contact the authors. Place in `../HiPRFISH_reference_spectra/`.
```bash
snakemake -s Snakefile_image_processing --configfile config.yaml -j NUM_CORES -p
```

### Key outputs saved per tile

| File | Location | Contents |
|---|---|---|
| `*_seg.npy` | `../outputs/.../segs/` | Cell segmentation mask (integer label array) |
| `*_props.csv` | `../outputs/.../segs/` | Cell centroids, areas, raw fluorescence spectra |
| `*_seg_plot.png` | `../outputs/.../plots/` | Segmentation visualization |
| `*_rgb_plot.png` | `../outputs/.../plots/` | False-color RGB composite of laser channels |
| `*_cell_scinames.npy` | `../outputs/.../classif/` | Genus label per cell (requires ref spectra) |
| `*_centroid_sciname.csv` | `../outputs/.../classif/coords.../` | Cell coordinates with genus assignments |

---

## Image processing

Create a conda environment using:

```
conda env create -f env_image_processing.yml
conda activate env_image_processing
```

Cythonize [functions/neighbor2d.pyx](https://github.com/benjamingrodner/peri_implantitis_HiPRFISH_analysis/blob/main/functions/neighbor2d.pyx) as in this [tutorial](https://docs.cython.org/en/latest/src/quickstart/build.html). 

List input image filenames in the format of [input_table_test.csv](https://github.com/benjamingrodner/peri_implantitis_HiPRFISH_analysis/blob/main/input_table_test.csv)

Adjust [config.yaml](https://github.com/benjamingrodner/peri_implantitis_HiPRFISH_analysis/blob/main/config.yaml) to point to the input table, functions, genus barcode, and output directories. You can also adjust image processing parameters here. 

Run the image processing pipeline

```
snakemake -s Snakefile_image_processing --configfile config.yaml -j NUM_CORES -p
```
## Cell density power spectrum

Create a new conda environment:

```
conda env create --f env_turbustat.yml
conda activate env_turbustat
```

Run the power spectrum pipeline

```
snakemake -s Snakefile_turbustat --configfile config.yaml -j NUM_CORES -p
```

For a more hands on look at the power spectrum analysis see [nb_turbustat.ipynb](https://github.com/benjamingrodner/peri_implantitis_HiPRFISH_analysis/blob/main/nb_turbustat.ipynb)

## Spatial analysis

Create a new conda environment:

```
conda env create --f env_spatial.yml
conda activate env_spatial
```

Run the power spectrum pipeline

```
snakemake -s Snakefile_spatial --configfile config.yaml -j NUM_CORES -p
```

Some of the plotting is done manually in [nb_spatial_stats.ipynb](https://github.com/benjamingrodner/peri_implantitis_HiPRFISH_analysis/blob/main/nb_spatial_stats.ipynb)
