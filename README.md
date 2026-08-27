# MPAS-Urban HRAIN2025: 30 km-500 m Hong Kong experiment

This repository provides the configuration, reference files, and tutorial
documents for an MPAS-Urban HRAIN2025 case initialized at
`2025-08-03_00:00:00`.

## Before you begin

> **New to MPAS-Atmosphere?**
>
> This is a case-specific example, not a replacement for the official
> MPAS-Atmosphere tutorial. First complete the
> [official MPAS-A tutorial](https://mpas-dev.github.io/atmosphere/tutorial.html)
> or the
> [MPAS-A Quick Start Guide](https://www2.mmm.ucar.edu/projects/mpas/site/documentation/users_guide/quick_start.html),
> and confirm that you can compile and run a standard MPAS-Atmosphere case.

The three PDF guides contain the complete commands, namelist changes, checks,
and explanations. This README is the entry point and map of the case.

## Choose your path first

Choose one source version before following the installation guide.

| Your goal | Source version | Suggested local directory |
|---|---|---|
| Learn MPAS-Urban or start a new independent experiment | [`HKUST-MPAS/HKUST-MPAS`, branch `hkust-dev`](https://github.com/HKUST-MPAS/HKUST-MPAS/tree/hkust-dev) | `$HOME/MPAS/HKUST-MPAS` |
| Reproduce the case-author HRAIN2025 configuration | [`Liuzh223/HKUST-MPAS-LIU`](https://github.com/Liuzh223/HKUST-MPAS-LIU), branch [`ntdk-grid-cutoff`](https://github.com/Liuzh223/HKUST-MPAS-LIU/tree/ntdk-grid-cutoff), pinned commit [`7c1610b...`](https://github.com/Liuzh223/HKUST-MPAS-LIU/commit/7c1610b8f41781c2433f780bd7496da63b25a920) | `$HOME/MPAS/HKUST-MPAS-NTDK` |

The standard `hkust-dev` branch is the default recommendation for learning and
new experiments. The case-specific source is a user-maintained derivative for
HRAIN2025. Its `ntdk-grid-cutoff` branch is not the official HKUST-MPAS physics
configuration.

`HKUST-MPAS-LIU` is the GitHub repository name, while `HKUST-MPAS-NTDK` is the
suggested local directory name used by these guides for the optional cutoff
build.

Both choices above are based on MPAS-Atmosphere v8.2.2. The pinned HRAIN2025
commit is a case-specific derivative, not a whole-tree upgrade to MPAS v8.3.

> **Compilation is required.** Checking out a branch or commit does not update
> an existing executable. Compile the selected source to generate
> `init_atmosphere_model` and `atmosphere_model`. Recompile whenever you change
> the branch, commit, or relevant source-code settings.

## Follow the tutorial in this order

1. Clone this case repository and download the v1.0 Release input using the
   commands below.
2. Follow [Guide 1 - Install HKUST-MPAS and the MPAS-Urban datasets](docs/MPAS_URBAN_INSTALLATION_GUIDE.pdf)
   to compile the source version you selected.
3. Follow [Guide 2 - Prepare the mesh and initialize HRAIN2025](docs/MPAS_URBAN_INITIALIZATION_GUIDE.pdf)
   to generate the static and initial-condition files.
4. Follow [Guide 3 - Run HRAIN2025](docs/MPAS_URBAN_RUNNING_GUIDE.pdf)
   to prepare the independent case directory and launch the atmosphere model.

> **Beginner shortcut:** use the supplied grid and GFS intermediate file. You
> do not need to regenerate the mesh or run WPS for the tutorial date.

Static-field generation for this large global variable-resolution mesh may
require several hours. Guide 2 provides planning estimates, checks, and ideas
for inspecting the mesh while the job runs.

## Download the case and supplied input

- **Case repository:** [Liuzh223/MPAS-Urban-HRAIN2025](https://github.com/Liuzh223/MPAS-Urban-HRAIN2025)
- **Release page:** [HRAIN2025 v1.0](https://github.com/Liuzh223/MPAS-Urban-HRAIN2025/releases/tag/v1.0)

The Git repository contains the documentation and small configuration files.
The separate Release contains the large grid and GFS intermediate file.

```bash
export MPAS_ROOT="$HOME/MPAS"
mkdir -p "$MPAS_ROOT"
cd "$MPAS_ROOT"

git clone https://github.com/Liuzh223/MPAS-Urban-HRAIN2025.git

mkdir -p "$MPAS_ROOT/HRAIN2025_INPUT"
cd "$MPAS_ROOT/HRAIN2025_INPUT"

curl -fL -O \
  https://github.com/Liuzh223/MPAS-Urban-HRAIN2025/releases/download/v1.0/MPAS-Urban_HRAIN2025_30km-to-500m_grid-and-GFS2025080300_v1.0.tar.gz

tar -xzf MPAS-Urban_HRAIN2025_30km-to-500m_grid-and-GFS2025080300_v1.0.tar.gz
ls -lh 30km_500m.grid.nc GFS:2025-08-03_00
```

For an optional file-integrity check, see Guide 2.

The supplied `GFS:2025-08-03_00` file is already in WPS intermediate format;
do not run `ungrib.exe` on it again. For another initialization date, prepare
all matching intermediate files as explained at the end of Guide 2.

### Soil data for this case

The soil data used in HRAIN2025 come from the updated global WRF soil dataset
described by [Dy and Fung (2016)](https://doi.org/10.1002/2015JD024558). This
dataset is not included in this repository. To make a simulation fully
comparable with HRAIN2025, consult the paper and contact its authors to obtain
the same dataset and processing details.

If the Dy and Fung (2016) dataset is unavailable, you may use the official MPAS
BNU soil data. These data are publicly downloadable and are similar to, but not
identical to, the soil dataset used in HRAIN2025. MPAS v8.3+ provides the
`config_soilcat_data` option. The pinned HRAIN2025 source remains based on
v8.2.2 but adds this option from MPAS v8.3+. Standard `hkust-dev` v8.2.2 does
not provide this option and can use only its default STATSGO soil-category data.

```bash
export MPAS_ROOT="$HOME/MPAS"
cd "$MPAS_ROOT/DATA/mpas_static"

curl -fL -O \
  https://www2.mmm.ucar.edu/projects/mpas/bnu_soiltype_top.tar.bz2
tar -xjf bnu_soiltype_top.tar.bz2

test -s "$MPAS_ROOT/DATA/mpas_static/bnu_soiltype_top/index"
```

For MPAS v8.3+ or the pinned HRAIN2025 source, remove the leading `!` only when
you choose BNU for the static stage:

```fortran
! config_soilcat_data = 'BNU'  ! optional BNU data
```

Regenerate the static file after changing the soil input. Guide 2 explains the
version requirements and initialization details.

## What the downloads contain

The Git repository contains:

```text
MPAS-Urban-HRAIN2025/
|-- README.md
|-- config/
|   |-- namelist.init_atmosphere
|   `-- namelist.atmosphere
|-- docs/
|   |-- MPAS_URBAN_INSTALLATION_GUIDE.pdf
|   |-- MPAS_URBAN_INITIALIZATION_GUIDE.pdf
|   `-- MPAS_URBAN_RUNNING_GUIDE.pdf
`-- mesh/
    |-- generate_hk_500m_mesh.py
    `-- hk_hull_500m_graph.info.part.112
```

The v1.0 Release archive contains only the supplied case input:

```text
MPAS-Urban_HRAIN2025_30km-to-500m_grid-and-GFS2025080300_v1.0.tar.gz
|-- 30km_500m.grid.nc
`-- GFS:2025-08-03_00
```

The repository and Release do **not** contain MPAS executables, standard MPAS
static data, CGLC-LCZ data, the Dy and Fung (2016) dataset, official BNU
soil-category data,
`vertical_levels/urban_ZR_75.txt`, generated static or initial-condition
files, model output, or GFS files for other times.

## Recommended local workspace

Keep the source builds, datasets, case input, and independent run directory
under one MPAS root:

```text
$HOME/MPAS/
|-- HKUST-MPAS/                 # standard compiled source
|-- HKUST-MPAS-NTDK/            # optional HRAIN2025 cutoff build
|-- DATA/
|   `-- mpas_static/            # standard, CGLC-LCZ, and optional BNU data
|-- MPAS-Urban-HRAIN2025/       # this Git repository
|-- HRAIN2025_INPUT/            # Release grid and GFS file
`-- HRAIN2025_30km_500m/        # independent case directory
```

The guides define `MPAS_BUILD` as a tutorial shell variable pointing to the
compiled source selected for one case. It is not an official MPAS environment
variable or namelist option.

For the standard build:

```bash
export MPAS_BUILD="$HOME/MPAS/HKUST-MPAS"
test -x "$MPAS_BUILD/atmosphere_model"
```

For the optional HRAIN2025 cutoff build, set `MPAS_BUILD` to
`$HOME/MPAS/HKUST-MPAS-NTDK`. The executable, streams files, stream lists,
physics tables, and data links for one case must all come from the same build.
Use separate case directories when comparing source versions.

## HRAIN2025 at a glance

- **Event:** the Hong Kong Black Rainstorm Event of August 2-5, 2025, with the
  peak on August 5.
- **Simulation period:** `2025-08-03 00:00 UTC` to
  `2025-08-06 00:00 UTC`.
- **Initialization:** `2025-08-03_00:00:00` from the supplied GFS intermediate
  file.
- **Mesh:** global variable-resolution mesh, approximately 30 km outside and
  500 m over the innermost refinement region.
- **Default decomposition:** 112 MPI tasks.

| Setting | HRAIN2025 value |
|---|---|
| Run duration | `3_00:00:00` |
| Vertical levels | 55 |
| Dynamics time step | 6 s |
| `config_len_disp` | `500.0` m |
| Urban physics | enabled |
| Optional CU cutoff | `config_cu_disable_dx = 10000.0`, case-specific source only |

Guide 3 contains the complete physics configuration and required symbolic
links. With the standard `hkust-dev` source, remove or comment out
`config_cu_disable_dx`; this switch is available only in the cutoff-capable
case-specific build. In that build, `10000.0` disables New Tiedtke convection
where local grid spacing is below 10 km; setting the value to `0.0` disables the
cutoff itself, so New Tiedtke is not suppressed by this threshold.

Soil-data choices and static-file generation are explained in Guide 2.

## Reference configuration files

| Stage | Main files | Instructions |
|---|---|---|
| Installation | build configuration and MPAS-Urban datasets | Guide 1 |
| Static interpolation | `namelist.init_atmosphere`, `streams.init_atmosphere` | Guide 2 |
| Meteorological initialization | `namelist.init_atmosphere`, `streams.init_atmosphere` | Guide 2 |
| Atmosphere run | `namelist.atmosphere`, `streams.atmosphere` | Guide 3 |

Reference namelists:

- [`config/namelist.init_atmosphere`](config/namelist.init_atmosphere)
- [`config/namelist.atmosphere`](config/namelist.atmosphere)

The BNU line in the initialization namelist is commented out by default. Enable
it only with MPAS v8.3+ or the pinned HRAIN2025 source and only when you have
chosen the BNU data for static-file generation.

## Case-specific reproducibility settings

This section is only for reproducing the case-author HRAIN2025 configuration.
These are case-specific choices, not general MPAS-Urban recommendations.

- Use the pinned source commit
  [`7c1610b8f41781c2433f780bd7496da63b25a920`](https://github.com/Liuzh223/HKUST-MPAS-LIU/commit/7c1610b8f41781c2433f780bd7496da63b25a920).
- The pinned source contains the 10 km CU cutoff used for this case. Set
  `config_cu_disable_dx = 0.0` when the cutoff should be inactive.
- For the soil input, follow "Soil data for this case" above and Guide 2.
- It also contains the two active `ZT = 0.1 * Z0` assignments used by the
  historical experiment, so no manual source edit or rainfall-branch checkout
  is required.

The `ZT` implementation originates from upstream HKUST-MPAS commit
[`562bdb4e5ef63527496b6374d52def06a01a84a0`](https://github.com/HKUST-MPAS/HKUST-MPAS/commit/562bdb4e5ef63527496b6374d52def06a01a84a0).
It changes the thermal roughness-length treatment and does not redefine or
recalculate `Z0`.

The corresponding historical simulation results have not yet been publicly
released. These settings document provenance for possible future comparison;
they are not required for learning MPAS or running a new independent
experiment.

## Optional mesh generation

For the supplied case, mesh generation is optional. Use
`mesh/generate_hk_500m_mesh.py` only when changing the refinement region or
independently testing the mesh-generation workflow.

The Hong Kong boundary used by the case author was downloaded through the
[Aliyun DataV area selector](https://datav.aliyun.com/portal/school/atlas/area_selector)
and stored locally as `mesh/hongkong.geojson`. Aliyun DataV is a third-party
service and is not an official Hong Kong SAR Government boundary source.
Archive the exact GeoJSON used for a reproduction, check its terms before
redistribution, and note that replacing the boundary may change the mesh.

## Documentation note

Tutorial author: **LIU Zhuo**

Affiliation: **The Hong Kong University of Science and Technology (HKUST)**

OpenAI ChatGPT/Codex assisted with the organization, language editing, and
formatting of this tutorial. All technical settings, commands, source-code
references, and case-specific scientific decisions remain the responsibility
of the author, LIU Zhuo. AI assistance is not treated as a technical or
scientific source.
