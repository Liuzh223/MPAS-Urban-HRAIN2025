# HRAIN2025: 30 km–500 m Hong Kong MPAS experiment

> **New to MPAS-Atmosphere?**
>
> This repository is a case-specific example, not a replacement for the
> official MPAS-Atmosphere tutorial. New users should first complete the
> [official MPAS-A tutorial](https://mpas-dev.github.io/atmosphere/tutorial.html)
> or the
> [MPAS-A Quick Start Guide](https://www2.mmm.ucar.edu/projects/mpas/site/documentation/users_guide/quick_start.html),
> and confirm that they can compile and run a standard MPAS-Atmosphere case.

This repository records the configuration, mesh-generation source, and
112-task graph partition used for the HRAIN2025 Hong Kong 30 km–500 m
variable-resolution experiment. It focuses on the settings that differ from a
standard MPAS tutorial: MPAS-Urban data, the Hong Kong mesh, the supplied GFS
initial condition, and the optional New Tiedtke grid-cutoff modification.

## HRAIN2025 case

- **Event:** The 2025 Black Rainstorm Event - `HRAIN2025`.
- **Timespan:** August 2-5, 2025, with the peak on August 5.
- **Observation highlights:** 358.8 mm on August 5 (355.7 mm at HKO
  headquarters), the highest single-day August rainfall since 1884; the fourth
  black rainstorm warning in eight days; more than 9,600 cloud-to-ground
  lightning strikes between 05:00 and 12:00; peak rainfall near 90 mm h-1.
- **Reported impacts:** at least 2 deaths and more than 140 injuries, 42 elevator
  entrapments, 175 fire-alarm activations, 36 fallen trees, 7 landslides and 69
  flooding cases; about 20% of flights were cancelled and more than 400 delayed.
- **Common simulation period:** `2025-08-03 00:00 UTC` to
  `2025-08-06 00:00 UTC` (3 days).
- **GFS initialization used by this tutorial:** `2025-08-03_00:00:00`.

## Choose the code for your purpose

### Standard MPAS-Urban learning and new experiments

Use the HKUST-MPAS team repository and its `hkust-dev` branch:

```bash
git clone --depth 1 --branch hkust-dev --single-branch \
  https://github.com/HKUST-MPAS/HKUST-MPAS.git
```

This is the default source recommended in the installation guide.

### HRAIN2025 case-specific CU-cutoff and ZT-compatible source

The HRAIN2025 comparison configuration uses a separate case-specific source.
It combines the case-author 10 km CU cutoff with the two active
`ZT = 0.1 * Z0` assignments used by the existing historical-result workflow.
Use this source only when those case-specific settings are required.

```bash
git clone --depth 1 --branch ntdk-grid-cutoff --single-branch \
  https://github.com/Liuzh223/HKUST-MPAS-Official.git \
  HKUST-MPAS-NTDK
```

`ntdk-grid-cutoff` is a case-author modification. It is not the official
HKUST-MPAS main branch and does not represent the default HKUST-MPAS physics
configuration. The word `Official` above is part of the repository name only.
There is no separate shell command named `cutoff`.

The source tested for this case is fixed at
[`57b42fa1d7649117bbf28a69c47697ea8756d268`](https://github.com/Liuzh223/HKUST-MPAS-Official/commit/57b42fa1d7649117bbf28a69c47697ea8756d268).
The `ZT = 0.1 * Z0` behavior originates from upstream HKUST-MPAS commit
[`562bdb4e5ef63527496b6374d52def06a01a84a0`](https://github.com/HKUST-MPAS/HKUST-MPAS/commit/562bdb4e5ef63527496b6374d52def06a01a84a0).

## Fastest way to prepare the HRAIN2025 input

Clone this case repository:

```bash
git clone https://github.com/Liuzh223/HRAIN2025.git
cd HRAIN2025
```

Download the ready-to-use grid and GFS intermediate file:

```bash
curl -fL -O \
https://github.com/Liuzh223/HRAIN2025/releases/download/v1.0/MPAS-Urban_HRAIN2025_30km-to-500m_grid-and-GFS2025080300_v1.0.tar.gz

echo "45fcfab80a3ecb5f8db558a4582d8910d1cac236f202ab499bb2b5e8e0e8143d  MPAS-Urban_HRAIN2025_30km-to-500m_grid-and-GFS2025080300_v1.0.tar.gz" \
  | sha256sum -c -

tar -xzf \
  MPAS-Urban_HRAIN2025_30km-to-500m_grid-and-GFS2025080300_v1.0.tar.gz

ls -lh 30km_500m.grid.nc GFS:2025-08-03_00
```

The archive contains only:

- `30km_500m.grid.nc`
- `GFS:2025-08-03_00`

It does not contain MPAS executables, standard static data, CGLC-LCZ data,
namelists, initialized files, model output, or additional GFS times.

## Experiment settings

- Initial time: `2025-08-03_00:00:00`
- Run duration: `3_00:00:00`
- Inner/outer nominal mesh spacing: 0.5 km / 30 km
- Vertical levels: 55
- Dynamics time step: 6 s
- Urban physics: enabled
- Default graph partition: 112 MPI tasks

## Initialize the case

The recommended beginner path is to use the grid from the v1.0 Release. Follow
the initialization guide to:

1. link `30km_500m.grid.nc` and `GFS:2025-08-03_00` into a run directory;
2. use the supplied 112-task graph partition;
3. create the static file with the MPAS static and CGLC-LCZ datasets;
4. create the `2025-08-03_00:00:00` initial-condition file.

Users who want to change the refinement region may regenerate the mesh with
`mesh/generate_hk_500m_mesh.py`. Mesh generation is optional and is not a
prerequisite for running the supplied case.

The Hong Kong boundary used by the case author was downloaded through the
[Aliyun DataV area selector](https://datav.aliyun.com/portal/school/atlas/area_selector)
and saved as `mesh/hongkong.geojson`. Aliyun DataV is a third-party service;
this link documents the case workflow and is not an official Hong Kong SAR
Government boundary source. Archive the exact GeoJSON used in a reproduction
and check the source terms before redistributing it. Replacing the boundary may
change the mesh and will not reproduce the archived grid exactly.

Reference initialization namelist:

- [`config/namelist.init_atmosphere`](https://github.com/Liuzh223/HRAIN2025/blob/main/config/namelist.init_atmosphere)

The linked namelist is configured for meteorological initialization. To create
the static file, make a separate copy and change:

1. In `&data_sources`, set `config_geog_data_path` to the real static-data
   directory and select the required CGLC-LCZ land-use dataset.
2. In `&preproc_stages`, use:
   `config_static_interp = true`, `config_native_gwd_static = true`,
   `config_vertical_grid = false`, `config_met_interp = false`,
   `config_input_sst = false`, and `config_frac_seaice = false`.
3. In `streams.init_atmosphere`, use `30km_500m.grid.nc` as input and
   choose a static output such as `30km_500m.static.nc`.

For meteorological initialization, restore the linked namelist and check:

1. `config_start_time` and `config_stop_time` are both
   `2025-08-03_00:00:00`.
2. `config_met_prefix = 'GFS'` matches `GFS:2025-08-03_00`.
3. `config_specified_zeta_levels` points to an existing
   `vertical_levels/urban_ZR_75.txt`.
4. In `&preproc_stages`, use:
   `config_static_interp = false`, `config_native_gwd_static = false`,
   `config_vertical_grid = true`, `config_met_interp = true`,
   `config_input_sst = false`, and `config_frac_seaice = true`.
   `config_input_sst` stays `false`; `config_frac_seaice` is `false` for
   static generation and `true` for initialization.
5. `config_block_decomp_file_prefix` resolves to
   `hk_hull_500m_graph.info.part.112` for 112 MPI tasks.
6. In `streams.init_atmosphere`, use the static file as input and select the
   intended `.init.nc` output filename.

## Run the case

Prepare the run directory after completing Guide 2. The initial-condition file
and `hk_hull_500m_graph.info.part.112` should already be in `$CASE`.

```bash
export MPAS_SRC="$HOME/MPAS/HKUST-MPAS"
# For the HRAIN2025 comparison/cutoff source, use instead:
# export MPAS_SRC="$HOME/MPAS/HKUST-MPAS-NTDK"

export HRAIN_REPO="$HOME/HRAIN2025"
export CASE="$HOME/MPAS_CASES/HRAIN2025_30km_500m"

test -d "$CASE"
cd "$CASE"

ln -sf "$MPAS_SRC/atmosphere_model" .
cp "$HRAIN_REPO/config/namelist.atmosphere" .
cp "$MPAS_SRC/streams.atmosphere" .
ln -sf "$MPAS_SRC"/default_inputs/stream_list.atmosphere.* .
ln -sf "$MPAS_SRC"/src/core_atmosphere/physics/physics_wrf/files/*TBL .
ln -sf "$MPAS_SRC"/src/core_atmosphere/physics/physics_wrf/files/*DATA* .
ln -sf "$MPAS_SRC"/src/core_atmosphere/physics/physics_noahmp/parameters/*TBL .

ls -lh atmosphere_model namelist.atmosphere streams.atmosphere \
  30km_500m.init.nc hk_hull_500m_graph.info.part.112
```

The explicit lookup-table and data links mirror the MPAS build/run setup.
They are listed here even though some installations or earlier tutorial steps
may already have created them.

The HRAIN2025 reference namelist explicitly selects the following case
configuration:

```fortran
&nhyd_model
    config_start_time   = '2025-08-03_00:00:00'
    config_run_duration = '3_00:00:00'
    config_dt           = 6
    config_len_disp     = 500.0
/

&physics
    config_microp_scheme       = 'mp_wsm6'
    config_convection_scheme   = 'cu_ntiedtke'
    config_lsm_scheme          = 'sf_noahmp'
    config_pbl_scheme          = 'bl_ysu'
    config_gwdo_scheme         = 'bl_ysu_gwdo'
    config_radt_cld_scheme     = 'cld_fraction'
    config_radt_lw_scheme      = 'rrtmg_lw'
    config_radt_sw_scheme      = 'rrtmg_sw'
    config_sfclayer_scheme     = 'sf_monin_obukhov_rev'
    config_urban_physics       = true
/
```

In physical terms, the case uses WSM6 microphysics, New Tiedtke cumulus,
Noah-MP land surface, YSU boundary layer and gravity-wave-drag options, the
cloud-fraction scheme, RRTMG longwave and shortwave radiation, the revised
Monin-Obukhov surface layer, and MPAS-Urban.

For the optional cutoff-capable derivative only, add:

```fortran
&physics
    config_cu_disable_dx = 10000.0
/
```

`config_cu_disable_dx = 10000.0` disables New Tiedtke convection where the
local mesh spacing is below 10 km. This option is available only in a build
that contains the case-author cutoff modification. Remove or comment it when
using the standard `hkust-dev` source.

Reference run namelist:

- [`config/namelist.atmosphere`](https://github.com/Liuzh223/HRAIN2025/blob/main/config/namelist.atmosphere)

Before running, edit or verify every case-specific item:

1. In `&nhyd_model`, set the start time, run duration, 6 s time step, and
   `config_len_disp = 500.0`.
2. In `&decomposition`, use
   `config_block_decomp_file_prefix = 'hk_hull_500m_graph.info.part.'`.
3. In `&physics`, verify all explicitly selected schemes and enable
   `config_urban_physics = true`.
4. Use `config_cu_disable_dx = 10000.0` only with a cutoff-capable build.
5. In `streams.atmosphere`, point the input stream to the `.init.nc` file,
   set the desired history filename and output interval, and verify that every
   referenced file exists relative to the run directory.

## Comparing with existing HRAIN2025 results

The HRAIN2025 historical experiment results have not yet been publicly
released. The following note is only for a future direct comparison with those
existing results; it is not required for learning MPAS or for a new independent
experiment.

Use the verified HRAIN2025 case-specific source at
[`57b42fa1d7649117bbf28a69c47697ea8756d268`](https://github.com/Liuzh223/HKUST-MPAS-Official/commit/57b42fa1d7649117bbf28a69c47697ea8756d268).
This single source contains both the 10 km CU cutoff and the HRAIN2025
`ZT` compatibility setting; the `rainfall` branch is not required.

In `src/core_atmosphere/physics/physics_wrf/module_sf_urban.F`, subroutine
`SFCDIF_URB` contains two active `ZT` updates: the first after `USTAR` is
calculated and the second inside the stability iteration. At both locations,
the final active expression is:

```fortran
ZT = 0.1 * Z0
```

Here, `Z0` is the momentum roughness length and `ZT` is the thermal
roughness length; this setting does not redefine or recalculate `Z0`. Its
upstream source is HKUST-MPAS commit
[`562bdb4e5ef63527496b6374d52def06a01a84a0`](https://github.com/HKUST-MPAS/HKUST-MPAS/commit/562bdb4e5ef63527496b6374d52def06a01a84a0).
The case-specific source already includes the setting, so no manual source edit
is needed. Check out the exact commit and rebuild `atmosphere_model`. Do not
confuse it with `Z0HC_TBL = 0.1 * Z0C_TBL`, which is a different calculation.
For new independent experiments, keep the implementation supplied by the
selected code version and record its branch or commit.

## Repository file structure

```text
HRAIN2025/
|-- .gitignore
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

The v1.0 Release is separate from the Git tree:

```text
MPAS-Urban_HRAIN2025_30km-to-500m_grid-and-GFS2025080300_v1.0.tar.gz
|-- 30km_500m.grid.nc
`-- GFS:2025-08-03_00
```

The commands in this guide use `$CASE` only as an example working directory.
Keep the directory organization from the official MPAS tutorial or from your
own system. The executables, streams files, vertical-level definition, static
data, static file, and initial-condition file are not stored in this Git
repository or in the v1.0 Release.

## File editing map

| Stage | File to edit | Main changes |
|---|---|---|
| Static interpolation | `namelist.init_atmosphere` | static path, CGLC-LCZ selection, stage flags |
| Static interpolation | `streams.init_atmosphere` | grid input and static output |
| Meteorological initialization | `namelist.init_atmosphere` | time, GFS, vertical grid, decomposition, stage flags |
| Meteorological initialization | `streams.init_atmosphere` | static input and initial-condition output |
| Atmosphere run | `namelist.atmosphere` | time, duration, `config_len_disp`, physics, optional cutoff |
| Atmosphere run | `streams.atmosphere` | initial-condition input, history output and interval |

## Documentation

Tutorial author: **LIU Zhuo**

Affiliation: **The Hong Kong University of Science and Technology (HKUST)**

- [Guide 1 - Install HKUST-MPAS and the MPAS-Urban datasets](docs/MPAS_URBAN_INSTALLATION_GUIDE.pdf)
- [Guide 2 - Prepare the mesh and initialize HRAIN2025](docs/MPAS_URBAN_INITIALIZATION_GUIDE.pdf)
- [Guide 3 - Run HRAIN2025 and interpret the case-specific switches](docs/MPAS_URBAN_RUNNING_GUIDE.pdf)

## Data and software provenance

MPAS and HKUST-MPAS source redistribution remains subject to the license in the
corresponding source repository. GFS input originates from NOAA/NCEP. The case
author obtained the Hong Kong GeoJSON through the third-party Aliyun DataV area
selector; the repository does not present that boundary as an official Hong
Kong SAR Government dataset. Retain the attribution required by each source,
check redistribution terms, and do not imply endorsement by NOAA, HKUST,
Aliyun, or the Hong Kong SAR Government.
