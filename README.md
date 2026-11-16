# ENSO-related monthly near-surface temperature anomalies

This package contains the gridded data underlying the masked reconstructions
for the 1982/83, 1997/98, 2015/16 and 2023/24 ENSO winters and the 2026/27
seasonal forecast, plus the fitted regression fields and the historical
HadISST EOFs and PCs.

## Files

| File | Main variables and dimensions |
| --- | --- |
| `era5_t2m_event_reconstruction.nc` | `t2m_reconstructed(year, mode, lagmonth, latitude, longitude)`, 5 × 2 × 13 × 721 × 1440. Also includes event `pc(year, mode)`, `is_forecast(year)` and calendar `time(year, lagmonth)`. |
| `era5_t2m_monthly_regression.nc` | `beta`, `p_value` and `significant`, each with dimensions `(mode, lagmonth, latitude, longitude)`, 2 × 13 × 721 × 1440. The coefficient and p-value fields cover the full grid. |
| `hadisst_djf_eof_pc.nc` | `pc(year, mode)` for 77 winters (76 historical and one forecast), `is_forecast(year)`, `eof(mode, latitude, longitude)` on the 40 × 160 tropical Pacific grid, `eof_variance_fraction(mode)` and `valid_sst_grid(latitude, longitude)`. |

## Coordinates and units

- `year` is the December year of a DJF winter. For example, `1982`
  denotes December 1982 to February 1983. The historical record runs from
  1950/51 through 2025/26 inclusive.
- `year=2026` denotes the 2026/27 DJF winter. In both files with a year
  dimension, `is_forecast=1` identifies the ECMWF seasonal forecast
  projection and `is_forecast=0` identifies HadISST-based PC amplitudes.
- `lagmonth=0` is January at the centre of DJF. Lags −7 through +5 run
  from June of the December year through June of the following year.
  For example, the 1982/83 window spans June 1982 through June 1983.
  The reconstruction file has decoded calendar dates on `time`, with
  each monthly mean labelled by the first day of its month.
- `mode=1,2` corresponds to EOF1/PC1 and EOF2/PC2. PCs are dimensionless,
  with unit sample standard deviation over the 76 historical winters.
- ERA5 fields use the original global 0.25° grid. Latitude runs from 90°N to
  90°S and longitude runs from 0°E to 359.75°E. T2m anomalies are in K.
  A temperature difference of 1 K equals a difference of 1 °C.
  `beta` is in K per one historical PC standard deviation.
- The HadISST EOF maps use the 1° grid within 20°S–20°N and 120°E–80°W,
  with longitudes on the 0–360° convention. `eof` gives the SST anomaly
  in °C associated with a one-standard-deviation PC amplitude.
  `valid_sst_grid=1` flags cells included in the EOF fit.

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
ordinary least squares. Each event's contribution is the historical DJF
PC for that event multiplied by the coefficient for the corresponding
mode and lag. Each figure shows one mode's contribution to T2m.

The supplied significance mask uses two-sided, pointwise Student t tests
of zero slope. There are 73 residual degrees of freedom after accounting
for an intercept, a linear trend and one PC in each fit. The tests assume
independent errors across winters and use unadjusted p values. The
binary field `significant` is 1 when the unrounded p value is below 0.05.
The reconstruction file stores excluded cells as missing values (NaN).
The same regression mask applies to all five winters for a given mode and lag.

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

## Example in Python

```python
import xarray as xr

events = xr.open_dataset("era5_t2m_event_reconstruction.nc")
reg = xr.open_dataset("era5_t2m_monthly_regression.nc")
eof_pc = xr.open_dataset("hadisst_djf_eof_pc.nc")

# PC1 contribution for January 1998
january_1998 = events.t2m_reconstructed.sel(year=1997, mode=1, lagmonth=0)

# Reproduce the masked field from the corresponding PC and regression slope
pc = eof_pc.pc.sel(year=1997, mode=1)
slope = reg.beta.sel(mode=1, lagmonth=0)
keep = reg.significant.sel(mode=1, lagmonth=0) == 1
january_1998_from_beta = (pc * slope).where(keep)

# PC1 contribution for January 2027, using the projected forecast PC
january_2027 = events.t2m_reconstructed.sel(year=2026, mode=1, lagmonth=0)
```

Open the files with a NetCDF4-capable reader. When sharing or publishing
derived results, cite the original data sources.

- [HadISST 1.1, Met Office Hadley Centre](https://www.metoffice.gov.uk/hadobs/hadisst/).
  Rayner et al. (2003), *Journal of Geophysical Research*, [doi:10.1029/2002JD002670](https://doi.org/10.1029/2002JD002670).
- [ERA5 monthly averaged data on single levels, Copernicus Climate Data Store](https://doi.org/10.24381/cds.f17050d7).
  Hersbach et al. (2020), *Quarterly Journal of the Royal Meteorological Society*, [doi:10.1002/qj.3803](https://doi.org/10.1002/qj.3803).
- [ECMWF seasonal forecast system 51, Copernicus Climate Data Store](https://cds.climate.copernicus.eu/datasets/seasonal-monthly-single-levels).
