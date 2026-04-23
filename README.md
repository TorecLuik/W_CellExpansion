# W_CellExpansion

BIAFLOWS workflow for 2D Cell Expansion, algorithm by Ron Hoebe, Amsterdam UMC.

## What it does

This workflow takes a **nuclei label mask** as input and expands each nucleus outward by up to a configurable number of pixels to approximate the full cell body (cytoplasm). Each expanded region inherits the label value of its nearest nucleus, producing matching nuclei, cell, and cytoplasm label masks.

Typical use case: you have a nuclei segmentation (e.g. from Cellpose or StarDist) and want to derive cell and cytoplasm regions for downstream measurements such as granule counting.

## BIAFLOWS integration

This is a standard [BIAFLOWS](https://biaflows.neubias.org/) workflow. It uses the BIAFLOWS Python helpers (`biaflows-utilities`) to read input images from `--infolder` and write results to `--outfolder`. All user-facing parameters are defined in [`descriptor.json`](descriptor.json).

This workflow can be run:
- via **BIOMERO** on a SLURM cluster (primary use case) — BIOMERO passes the folder arguments and workflow parameters automatically,
- locally with **Docker or Singularity** — pass `--infolder`, `--outfolder`, `--gtfolder` and any workflow parameters yourself (see [Usage](#usage)).

## Input

- A 2D nuclei label mask image (grayscale, uint8, uint16, or uint32). Background must be 0; each nucleus is a unique non-zero integer label.
- Supported formats: TIFF and other formats readable by imageio.

## Output

- **Cells label mask** — nucleus + expanded cytoplasm region per cell.

## Parameters

All parameters are defined in `descriptor.json`.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `max_pixels` | 25 | Maximum expansion distance in pixels from the nucleus boundary. |
| `discard_cells_without_cytoplasm` | true | If enabled, nuclei that have no background pixels within `max_pixels` (i.e. entirely surrounded by other nuclei) are removed from the output. |

## Usage

### With Singularity (BIOMERO / SLURM)

```bash
singularity run "$IMAGE_PATH/$SINGULARITY_IMAGE" \
    --infolder "$DATA_PATH/data/in" \
    --outfolder "$DATA_PATH/data/out" \
    --gtfolder "$DATA_PATH/data/gt" \
    --local -nmc \
    --max_pixels 25 --discard_cells_without_cytoplasm true
```

### With Docker

```bash
docker run --rm \
    -v /path/to/input:/data/in \
    -v /path/to/output:/data/out \
    torecluik/w_cellexpansion \
    --infolder /data/in --outfolder /data/out \
    --local -nmc \
    --max_pixels 25 --discard_cells_without_cytoplasm true
```

## Building the container

```bash
docker build -t torecluik/w_cellexpansion .
# Convert for Singularity/Apptainer:
singularity build w_cellexpansion.sif docker-daemon://torecluik/w_cellexpansion:latest
```

## Notes

- If the input filename contains the word `Nuclei`, the output file will be named with `Nuclei` replaced by `Cells` (e.g. `myImage_Nuclei.tif` → `myImage_Cells.tif`). If not, the output keeps the same filename as the input.
- uint32 label masks exported from OMERO are fully supported.

## Author

Algorithm by Ron Hoebe (<r.a.hoebe@amsterdamumc.nl>), Amsterdam UMC.

## License

GNU General Public License v3.0 (copyleft) — see [LICENSE](LICENSE).
