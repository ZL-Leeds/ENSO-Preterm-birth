# ENSO-related monthly near-surface temperature anomalies

This package contains the monthly temperature contributions associated with
two tropical Pacific SST EOF modes. It includes complete reconstructions
for every winter from 1980/81 to 2026/27, significance-masked reconstructions
for four historical ENSO winters and the 2026/27 seasonal forecast, the
monthly regression coefficients and their significance information, and
the HadISST EOF patterns and PC time series.

## Files

| File | Main variables and dimensions |
| --- | --- |
| `era5_t2m_reconstruction_PC1_1980-2026_unmasked.nc` | Complete PC1 temperature contribution, `t2m_reconstructed(year, lagmonth, latitude, longitude)`, 47 × 13 × 721 × 1440. Includes `pc(year)`, `is_forecast(year)` and `time(year, lagmonth)`. File size 1,899,604,930 bytes. |
| `era5_t2m_reconstruction_PC2_1980-2026_unmasked.nc` | Complete PC2 temperature contribution, with the same variables and dimensions as the PC1 file. File size 1,928,682,721 bytes. |
| `era5_t2m_event_reconstruction.nc` | Significance-masked contributions for 1982/83, 1997/98, 2015/16, 2023/24 and 2026/27. Contains `t2m_reconstructed(year, mode, lagmonth, latitude, longitude)`, 5 × 2 × 13 × 721 × 1440, plus `pc(year, mode)`, `is_forecast(year)` and `time(year, lagmonth)`. |
| `era5_t2m_monthly_regression.nc` | `beta`, `p_value` and `significant`, each with dimensions `(mode, lagmonth, latitude, longitude)`, 2 × 13 × 721 × 1440. The coefficient and p-value fields cover the full grid. |
| `hadisst_djf_eof_pc.nc` | `pc(year, mode)` for 77 winters (76 historical and one forecast), `is_forecast(year)`, `eof(mode, latitude, longitude)` on the 40 × 160 tropical Pacific grid, `eof_variance_fraction(mode)` and `valid_sst_grid(latitude, longitude)`. |

The two unmasked reconstruction files contain all 47 consecutive winter
labels from 1980 to 2026. The five-event reconstruction contains winter
labels 1982, 1997, 2015, 2023 and 2026. The EOF/PC file contains all 77
winter labels from 1950 to 2026. The regression file contains one set
of monthly coefficients estimated over the historical record.

## Coordinates and units

- `year` is the December year of a DJF winter. For example, `1982`
  denotes December 1982 to February 1983. The historical EOF and regression
  fitting period runs from 1950/51 through 2025/26 inclusive.
- `year=2026` denotes the 2026/27 DJF winter. In the reconstruction and PC
  files, `is_forecast=1` identifies the ECMWF seasonal forecast projection;
  `is_forecast=0` identifies HadISST-based PC amplitudes. The 47-year
  reconstructions contain 46 historical winters and one forecast winter.
- `lagmonth=0` is January at the centre of DJF. Lags −7 through +5 run
  from June of the December year through June of the following year.
  For example, the 1982/83 window spans June 1982 through June 1983.
  The first window in the continuous files spans June 1980 to June 1981,
  and the final window spans June 2026 to June 2027.
- `time(year, lagmonth)` records the actual calendar month, labelled by its
  first day. It is stored as integer days since 1950-01-01, using the
  proleptic Gregorian calendar. Readers such as xarray decode it to dates.
- `mode=1,2` corresponds to EOF1/PC1 and EOF2/PC2. Each continuous
  reconstruction file contains one mode, identified by its filename
  and global attribute `eof_mode`. Its `pc` variable has dimensions
  `(year)`. The five-event and EOF/PC files have `pc(year, mode)`.
- PCs are dimensionless, with unit sample standard deviation over the
  76 historical winters. Their original scale is retained when selecting
  the 1980/81–2026/27 subset.
- ERA5 fields use the original global 0.25° grid. Latitude runs from 90°N to
  90°S and longitude runs from 0°E to 359.75°E. T2m anomalies are in K.
  A temperature difference of 1 K equals a difference of 1 °C.
  `beta` is in K per one historical PC standard deviation.
- The HadISST EOF maps use the 1° grid within 20°S–20°N and 120°E–80°W,
  with longitudes on the 0–360° convention. `eof` gives the SST anomaly
  in °C associated with a one-standard-deviation PC amplitude.
  `valid_sst_grid=1` flags cells included in the EOF fit.

The continuous reconstruction files store `t2m_reconstructed` as float32,
`pc` as float64, `year`, `lagmonth` and `time` as int32, the spatial
coordinates as float32, and `is_forecast` as int8. Their additional
`copied_from_event_file(year)` flag is zero throughout, because these
complete fields were recalculated from the full coefficients. All grid
cells in these two temperature variables are finite. The files use
lossless NetCDF4 compression.

## Analysis

The HadISST 1.1 DJF fields were formed from equally weighted monthly SSTs.
At each eligible grid point we subtracted the 76-winter mean and removed
a linear trend over winter year. The covariance EOFs were fitted by SVD
after weighting anomalies by the square root of the cosine of latitude.
EOF1 and EOF2 explain 68.81% and 9.04% of the weighted detrended variance,
respectively. The maps and PCs carry the same signs and scaling as the
temperature regressions.

For each ERA5 grid point and lag, the 76 monthly T2m samples were detrended
along winter year and centred by subtracting the mean for that lag. The
resulting T2m anomalies were regressed separately on each DJF PC by
ordinary least squares. The two PCs are orthogonal over this historical
sample, and the separate and joint regression slopes agree to numerical
precision.

For each winter, the temperature contribution from each mode is its DJF
PC multiplied by the coefficient for that mode and lag:

```text
T_PCk(year, lagmonth, latitude, longitude)
    = pc_k(year) * beta_k(lagmonth, latitude, longitude)
```

Each winter has one PC amplitude per mode, applied to its 13 monthly
regression patterns. The two unmasked files provide this calculation
at every grid cell for all 47 winters. Their global attribute
`significance_mask_applied=0` records this treatment.

The supplied significance information uses two-sided, pointwise Student
t tests of zero slope. There are 73 residual degrees of freedom after
accounting for an intercept, a linear trend and one PC in each fit.
The tests assume independent errors across winters and use unadjusted
p values. In `era5_t2m_monthly_regression.nc`, the binary variable
`significant` is 1 when the unrounded p value is below 0.05. This mask
depends on mode, lag and location and applies to every winter.

The five-event file, `era5_t2m_event_reconstruction.nc`, applies this
mask and stores excluded cells as missing values (NaN). The continuous
unmasked fields retain the full regression contribution, with
significance available separately for users who wish to filter a figure.
For the five original events, values at significant grid cells agree
between the event file and the complete fields.

The complete PC1 and PC2 contributions can be added to obtain the
two-mode ENSO-related temperature contribution. A historical residual
can then be calculated from an ERA5 T2m anomaly field prepared using
the same detrending, monthly centring, grid and winter alignment:

```text
residual = ERA5_T2m_anomaly - T_PC1 - T_PC2
```

These residuals provide historical samples of variability left
unexplained by the two EOF regressions. Calculating them requires the
corresponding observed ERA5 anomaly fields. The observed anomaly and
residual fields are separate from the reconstruction variables supplied
here. Historical residual sampling can use the 46 winters from 1980/81
through 2025/26.

## 2026/27 seasonal forecast

The ECMWF system 51 forecast was initialised on 1 November 2026 at 00:00 UTC.
The DJF SST field is the equal-weight mean of 51 members and the December,
January and February monthly fields. We subtracted the historical HadISST
DJF mean and extrapolated fitted linear trend at each grid point to 2026/27.
Both EOF amplitudes were fitted together by latitude-weighted least squares
over the 6,078 grid points shared by the ECMWF and HadISST masks. The
projected PC1 and PC2 are +4.35299352 and −2.89442094 in historical
standard-deviation units. The 76 observed winters define the EOFs, PC
scaling, regression coefficients and significance mask.

The forecast-based reconstruction applies these projected PCs to the fixed
historical regression patterns from June 2026 to June 2027. These fields
represent the estimated ENSO contribution, including statistical
reconstructions for months outside the ECMWF forecast window. The SST
anomalies use the HadISST baseline, so ECMWF model bias may remain.
Projected PC1 exceeds the historical maximum of +2.61, so the corresponding
temperature amplitudes extrapolate the historical linear fit.

## Examples in Python

The examples assume the files are in the Python working directory.
Adjust paths to their locations in your downloaded package.

```python
import xarray as xr

pc1 = xr.open_dataset("era5_t2m_reconstruction_PC1_1980-2026_unmasked.nc")
pc2 = xr.open_dataset("era5_t2m_reconstruction_PC2_1980-2026_unmasked.nc")
events = xr.open_dataset("era5_t2m_event_reconstruction.nc")
reg = xr.open_dataset("era5_t2m_monthly_regression.nc")
eof_pc = xr.open_dataset("hadisst_djf_eof_pc.nc")

# Complete PC1 contribution for January 1998
january_1998_pc1 = pc1.t2m_reconstructed.sel(year=1997, lagmonth=0)

# Reproduce the complete field from the corresponding PC and regression slope
pc = eof_pc.pc.sel(year=1997, mode=1)
slope = reg.beta.sel(mode=1, lagmonth=0)
january_1998_from_beta = pc * slope

# Apply the stored significance mask for a figure
keep = reg.significant.sel(mode=1, lagmonth=0) == 1
january_1998_significant = january_1998_pc1.where(keep)

# Read the equivalent masked contribution from the five-event file
january_1998_event = events.t2m_reconstructed.sel(
    year=1997, mode=1, lagmonth=0
)
january_1998_event_from_beta = january_1998_from_beta.where(keep)

# PC1 contribution for January 2027, using the projected forecast PC
january_2027 = events.t2m_reconstructed.sel(year=2026, mode=1, lagmonth=0)

# Sum both complete contributions for the 2026/27 winter window
forecast_enso = (
    pc1.t2m_reconstructed.sel(year=2026)
    + pc2.t2m_reconstructed.sel(year=2026)
)

# Select the historical contributions for residual calculations
historical_enso = (
    pc1.t2m_reconstructed.sel(year=slice(1980, 2025))
    + pc2.t2m_reconstructed.sel(year=slice(1980, 2025))
)
```

## Validation and data sources

Every grid in the continuous reconstructions was checked against the
corresponding PC and full regression coefficient. For the five original
events, significant grid values were checked against the event
reconstruction. Winter labels, monthly time endpoints and forecast flags
were checked. Downloads were verified against server SHA256 checksums.

Open the files with a NetCDF4-capable reader. When sharing or publishing
derived results, cite the original data sources.

- [HadISST 1.1, Met Office Hadley Centre](https://www.metoffice.gov.uk/hadobs/hadisst/).
  Rayner et al. (2003), *Journal of Geophysical Research*, [doi:10.1029/2002JD002670](https://doi.org/10.1029/2002JD002670).
- [ERA5 monthly averaged data on single levels, Copernicus Climate Data Store](https://doi.org/10.24381/cds.f17050d7).
  Hersbach et al. (2020), *Quarterly Journal of the Royal Meteorological Society*, [doi:10.1002/qj.3803](https://doi.org/10.1002/qj.3803).
- [ECMWF seasonal forecast system 51, Copernicus Climate Data Store](https://cds.climate.copernicus.eu/datasets/seasonal-monthly-single-levels).
