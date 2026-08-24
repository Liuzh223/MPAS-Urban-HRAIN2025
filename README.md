# HRAIN2025: 30 km–500 m Hong Kong MPAS experiment

This directory archives the configuration, mesh-generation source, and 112-way
graph partition for the HRAIN2025 MPAS-Atmosphere experiment over Hong Kong.
The namelists retain the experiment settings while using the tutorial's portable
run-directory conventions.

## Contents

- `config/namelist.init_atmosphere`: MPAS-Init configuration.
- `config/namelist.atmosphere`: MPAS-Atmosphere configuration.
- `mesh/generate_hk_500m_mesh.py`: mesh-generation script for a 30 km global
  mesh with 500 m refinement around Hong Kong.
- `mesh/hk_hull_500m_graph.info.part.112`: METIS partition assignment for 112
  MPI tasks.

## Ready-to-use grid and GFS file

The public [v1.0 Release](https://github.com/Liuzh223/HRAIN2025/releases/tag/v1.0)
provides the grid and WPS intermediate file for the
`2025-08-03_00:00:00` initial time:

```bash
curl -fL -O https://github.com/Liuzh223/HRAIN2025/releases/download/v1.0/MPAS-Urban_HRAIN2025_30km-to-500m_grid-and-GFS2025080300_v1.0.tar.gz

echo "45fcfab80a3ecb5f8db558a4582d8910d1cac236f202ab499bb2b5e8e0e8143d  MPAS-Urban_HRAIN2025_30km-to-500m_grid-and-GFS2025080300_v1.0.tar.gz" \
  | sha256sum -c -

tar -xzf MPAS-Urban_HRAIN2025_30km-to-500m_grid-and-GFS2025080300_v1.0.tar.gz
```

The archive contains only:

- `30km_500m.grid.nc`
- `GFS:2025-08-03_00`

The graph partition, namelists, vertical-level file, MPAS static datasets,
executables, initialized `.init.nc`, and model output are not included in the
Release archive.

## Experiment summary

- Initialization time: `2025-08-03_00:00:00`
- Model initial time: `2025-08-03_00:00:00`
- Model run duration: `3_00:00:00`
- Nominal inner/outer mesh spacing: 0.5 km / 30 km
- Vertical levels: 55
- Dynamics time step: 6 s
- Urban physics: enabled
- Graph partitions: 112

## Explicit physics configuration

The model namelist does not depend on `config_physics_suite`. It explicitly
selects the schemes used by the experiment:

- WSM6 microphysics
- New Tiedtke convection
- Noah-MP land surface model
- YSU planetary boundary layer and revised Monin–Obukhov surface layer
- YSU orographic gravity-wave drag
- Cloud-fraction diagnosis and RRTMG longwave/shortwave radiation

The optional NTDK grid cutoff is implemented in
[HKUST-MPAS-Official](https://github.com/Liuzh223/HKUST-MPAS-Official),
branch `ntdk-grid-cutoff`. Its exact option is `config_cu_disable_dx`.
The default value is `0.0`, so the cutoff is disabled by default. This
experiment sets `config_cu_disable_dx = 10000.0`, disabling New Tiedtke
convection wherever the local mesh spacing is below 10 km. The option is used
only with `config_convection_scheme = 'cu_ntiedtke'`.

## Tutorial run-directory layout

Relative namelist paths are evaluated from the directory where the MPAS
executable is launched. The configuration assumes this layout:

```text
<tutorial-root>/
├── mpas_static/
└── run/
    ├── namelist.init_atmosphere
    ├── namelist.atmosphere
    ├── 30km_500m.grid.nc
    ├── GFS:2025-08-03_00
    └── hk_hull_500m_graph.info.part.112
```

Copy the files under `config/` and the included partition file into `run/`
before launching MPAS. In this layout, `../mpas_static` resolves to the
intended tutorial resources. The custom vertical-level file referenced by
`namelist.init_atmosphere` is distributed with the
[HKUST MPAS-Urban project](https://github.com/HKUST-MPAS/HKUST-MPAS#vertical-grid-considerations-for-urban-simulations)
and is not duplicated here.

### GFS initial-condition files

GFS intermediate files are not stored in the Git source tree. The v1.0 Release
provides `GFS:2025-08-03_00` for the configured initial time. For a different
initial time or an initialization requiring more valid times, process the
matching GFS GRIB2 data with WPS `ungrib.exe`, use the prefix `GFS`, and
place every required `GFS:<valid-time>` file in the run directory before
running MPAS-Init.

## Decomposition file

Both MPAS-Init and MPAS-Atmosphere use the prefix
`hk_hull_500m_graph.info.part.`. When launched with 112 MPI tasks, both resolve
to the included `hk_hull_500m_graph.info.part.112` file.

## Mesh-generation requirements

Install JIGSAW, METIS (`gpmetis`), `mpas_tools`, and the Python packages imported
by `mesh/generate_hk_500m_mesh.py`.

### Download `jigsaw_util.py`

The helper is maintained by Pedro S. Peixoto in the public MPAS-BR repository:

- Source: <https://github.com/pedrospeixoto/MPAS-BR/blob/master/grids/utilities/jigsaw/jigsaw_util.py>

Download it into the local `mesh/` directory:

```bash
curl -L \
  https://raw.githubusercontent.com/pedrospeixoto/MPAS-BR/master/grids/utilities/jigsaw/jigsaw_util.py \
  -o mesh/jigsaw_util.py
```

### Obtain a Hong Kong GeoJSON

Download a Hong Kong administrative-boundary GeoJSON and save it as
`mesh/hongkong.geojson`. A recommended authoritative source is the Hong Kong
Government's District Boundary dataset:

- Dataset page: <https://data.gov.hk/en-data/dataset/hk-had-json1-hong-kong-administrative-boundaries/resource/855f034a-c330-435c-a911-1d63538a6d55>

The dataset may contain all 18 districts as separate features. No manual merge
is required because the mesh script unions all features before constructing the
refinement boundary. The GeoJSON should use longitude/latitude coordinates;
other declared coordinate reference systems are converted to EPSG:4326.

If an administrative-boundary file is unavailable, the following command
creates an approximate circular Hong Kong refinement region. The circle is
constructed in UTM zone 50N so that its 42 km radius is defined in metres:

```bash
python - <<'PY'
import geopandas as gpd
from shapely.geometry import Point

center = gpd.GeoSeries(
    [Point(114.15, 22.35)],
    crs="EPSG:4326",
).to_crs("EPSG:32650")
geometry = center.buffer(42_000).to_crs("EPSG:4326").iloc[0]
data = gpd.GeoDataFrame(
    {
        "name": ["Hong Kong approximate circular region"],
        "radius_km": [42],
    },
    geometry=[geometry],
    crs="EPSG:4326",
)
data.to_file("mesh/hongkong.geojson", driver="GeoJSON")
PY
```

This circle is only a functional fallback. It changes the shape of the
high-resolution region and will not reproduce the archived mesh or its METIS
partition exactly.

### Generate the mesh

From the repository root, run:

```bash
python mesh/generate_hk_500m_mesh.py \
  --geojson mesh/hongkong.geojson \
  --jigsaw-util mesh/jigsaw_util.py \
  --output-dir generated_mesh
```

The default partition output is `hk_hull_500m_graph.info.part.112`. The default
global sampling field is memory intensive and should be generated on a
high-memory compute node.

## Provenance

The configuration files originate from the HRAIN2025 GFS 30 km–500 m MPAS
experiment.
