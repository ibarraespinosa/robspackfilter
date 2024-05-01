R Tools for Obspack, Footprints and Receptors (rtorf)
================

<figure>
<img
src="https://img.shields.io/github/commit-activity/y/noaa-gml/rtorf"
alt="GitHub commit activity" />
<figcaption aria-hidden="true">GitHub commit activity</figcaption>
</figure>

[![R build
status](https://github.com/ibarraespinosa/rtorf/workflows/draft-pdf/badge.svg)](https://github.com/ibarraespinosa/rtorf/actions)

[![R-CMD-check](https://github.com/noaa-gml/torf/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/noaa-gml/torf/actions/workflows/R-CMD-check.yaml)

[NOAA Obspack](https://gml.noaa.gov/ccgg/obspack/) is a collection of
green house gases observations

`rtorf` only depends on `data.table`, which is basically parallel C, so
it can be installed in any machine.

## installation

``` r
remotes::install_github("noaa-gml/rtorf")
```

``` r
library(rtorf)
library(data.table)
```

## ObsPack summary

The first step consists in constructing a summary for the ObsPack. This
is required to read the data, but also, identify `agl`, which is present
in some of the file names. This function returns a `data.frame`.
Optionally, the user can indicate a path to store the `data.frame`.
`obs_summary` also prints a summary of the data. The second argument is
the categories, and by default includes the categories shown below, to
account for all the files. Then the summary `data.frame` contains the
columns `id` as the full path to each file, `name` which is the name or
relative path of the file, `n` just an id, `sector` such as tower, and
the column `agl` which indicates the `agl` indicated in the name of the
file if available. To read the documentation of this function, the user
must run `?obs_summary`.

> This example reads text files but it will be updated to read NetCDF

``` r
cate = c("aircraft-pfp",
         "aircraft-insitu",
         "aircraft-flask",
         "surface-insitu",
         "surface-flask", 
         "surface-pfp",   
         "tower-insitu",  
         "aircore",       
         "shipboard-insitu",
         "shipboard-flask") 

obs <- "Z:/torf/obspack_ch4_1_GLOBALVIEWplus_v5.1_2023-03-08/data/nc/"
index <- obs_summary(obs = obs, categories = cate)
```

    ## Number of files of index: 429
    ##               sector   N
    ##  1:     aircraft-pfp  40
    ##  2:  aircraft-insitu  15
    ##  3:    surface-flask 106
    ##  4:   surface-insitu 174
    ##  5:   aircraft-flask   4
    ##  6:          aircore   1
    ##  7:      surface-pfp  33
    ##  8:     tower-insitu  51
    ##  9:  shipboard-flask   4
    ## 10: shipboard-insitu   1
    ## 11:    Total sectors 429
    ## Detected 190 files with agl
    ## Detected 239 files without agl

There are 429 files in the ObsPack directory. The printed information
also shows the total at the bottom, as the sum of the individual file by
sector. This is to ensure that the sum of files is equal to the total
number of files found, shown at the top. furthermore, the printed
information also shows that there are 109 files with the `agl`
explicitly mentioned in the name of the file.

### Read data

Once the summary is built, the function `obs_read` will read the files
available in the index file previously generated. Here we selected the
category “tower-insitu”. The argument verbose prints which files are
being read each time, by default. At the end, this function prints the
total number of observations by type of altitude (agl or asl).

``` r
df <- obs_read_nc(index = index,
                  categories = "tower-insitu")
```

Sometimes we need more information about the site. For instance, what do
the observations start and end. Then, we added the function `obs_table`,
which calculates statistics summary of “time” and other numeric
variables by file name, sector, site, altitude and mode.

``` r
dft <- obs_table(df = df)
```

Now we can visualize the average of observations by site.

``` r
library(sf)
sdft <- st_as_sf(dft[stat == "mean"], 
                 coords = c("longitude", "latitude"), 
                 crs = 4326)
sdft$value <- sdft$value*1e09
plot(sdft["value"], 
     reset = FALSE, 
     axes = TRUE, 
     graticule = TRUE,
     pch = 16, 
     cex = 1.5, 
     pal = cptcity::lucky(colorRampPalette = T), 
     main = NULL)
```

    ## Colour gradient: jjg_ccolo_hana_r_a_i_n_y, number: 3973

``` r
maps::map(add = T)
```

![](README_files/figure-gfm/sf-1.png)<!-- -->

Let us randomly select a couples of sites from the database.

``` r
usites <- unique(df$site_name)[sample(seq_along(unique(df$site_name)), 2)]
usites
```

    ## [1] "Tacolneston"        "Danville, Virginia"

``` r
knitr::kable(dft[site_name %in% usites])
```

| site_name          | site_latitude | site_longitude | site_country   | site_code |     value |       time | time_decimal | latitude | longitude | stat   | timeUTC             |
|:-------------------|--------------:|---------------:|:---------------|:----------|----------:|-----------:|-------------:|---------:|----------:|:-------|:--------------------|
| Danville, Virginia |       36.7058 |       -79.4369 | United States  | DVV       | 1.847e-06 | 1468449000 |         2017 |    36.71 |   -79.437 | min    | 2016-07-13 22:30:00 |
| Danville, Virginia |       36.7058 |       -79.4369 | United States  | DVV       | 1.945e-06 | 1483676100 |         2017 |    36.71 |   -79.437 | q1     | 2017-01-06 04:15:00 |
| Danville, Virginia |       36.7058 |       -79.4369 | United States  | DVV       | 1.966e-06 | 1494000000 |         2017 |    36.71 |   -79.437 | median | 2017-05-05 16:00:00 |
| Danville, Virginia |       36.7058 |       -79.4369 | United States  | DVV       | 1.970e-06 | 1493261865 |         2017 |    36.71 |   -79.437 | mean   | 2017-04-27 02:57:45 |
| Danville, Virginia |       36.7058 |       -79.4369 | United States  | DVV       | 1.994e-06 | 1504986300 |         2018 |    36.71 |   -79.437 | q3     | 2017-09-09 19:45:00 |
| Danville, Virginia |       36.7058 |       -79.4369 | United States  | DVV       | 2.356e-06 | 1514763000 |         2018 |    36.71 |   -79.437 | max    | 2017-12-31 23:30:00 |
| Tacolneston        |       52.5177 |         1.1386 | United Kingdom | TAC       | 1.830e-06 | 1343302200 |         2013 |    52.52 |     1.139 | min    | 2012-07-26 11:30:00 |
| Tacolneston        |       52.5177 |         1.1386 | United Kingdom | TAC       | 1.934e-06 | 1418884200 |         2015 |    52.52 |     1.139 | q1     | 2014-12-18 06:30:00 |
| Tacolneston        |       52.5177 |         1.1386 | United Kingdom | TAC       | 1.963e-06 | 1489437000 |         2017 |    52.52 |     1.139 | median | 2017-03-13 20:30:00 |
| Tacolneston        |       52.5177 |         1.1386 | United Kingdom | TAC       | 1.972e-06 | 1492570417 |         2017 |    52.52 |     1.139 | mean   | 2017-04-19 02:53:37 |
| Tacolneston        |       52.5177 |         1.1386 | United Kingdom | TAC       | 1.998e-06 | 1566329400 |         2020 |    52.52 |     1.139 | q3     | 2019-08-20 19:30:00 |
| Tacolneston        |       52.5177 |         1.1386 | United Kingdom | TAC       | 4.496e-06 | 1640993400 |         2022 |    52.52 |     1.139 | max    | 2021-12-31 23:30:00 |

We added a function to plot the data read from ObsPack. The y-axis is
the field `value` and the x-axis is by default `time`. The data
illustrated sorted by color is the field `site_code`, with the default
number of 3 sites. The argument `pal` is to define the color palette,
used by the internally imported function `cptcity::cpt`.

``` r
obs_plot(dt = df[site_name %in% usites], time = "time", yfactor = 1e+09, cex = 0.5)
```

    ## Found the following sites: 
    ## [1] DVV TAC
    ## Plotting the following sites: 
    ## [1] DVV TAC

<figure>
<img src="README_files/figure-gfm/unnamed-chunk-8-1.png"
alt="First two sites in ObsPack" />
<figcaption aria-hidden="true">First two sites in ObsPack</figcaption>
</figure>

Here we can see 2.61 million observations for `tower-insitu`. These
observations are made between 2004 and 2021. The identification of the
altitude and type is critical. The approach used here consists of:

1.  Identify `agl` from the name of the tile.
2.  If `agl` not present, search fill_values used in elevation and
    transform them into NA (not available)
3.  If `agl` is not present, `agl = altitude - elevation`.
4.  If there are some NA in elevation, will result some NA in `agl`
5.  A new column is added named `altitude_final` to store `agl` or `asl`
6.  Another column named `type_altitude` is added to identify `agl` or
    `asl`.
7.  If there is any case NA in `altitude_final`, `type_altitude` is “not
    available”

### Filtering

ObsPack includes global observations and sometimes we need to extract
data for a specific region and periods of time. In this part we include
spatial and temporal parameters to filter data. The year of interest is
2020, but we also included December of 2019 and January of 2021. At this
stage, we can apply the spatial filter by using the coordinates.

# TODO ADD identifier of type of altitude of intake_height

``` r
north <- 80
south <- 10
west <- -170
east <- -50
max_altitude <- 8000
evening <- 14

yy <- 2020
df <- rbind(df[year == yy - 1 & month == 12],
            df[year == yy],
            df[year == yy + 1 & month == 1])


if(any(names(df) %in% "intake_height")) {
  df <- df[intake_height < max_altitude &
             latitude < north &
             latitude > south &
             longitude < east &
             longitude > west]
} else {
  df <- df[altitude_final < max_altitude &
             latitude < north &
             latitude > south &
             longitude < east &
             longitude > west]
}

unique(df[, c("intake_height", "site_elevation", "elevation",
              "dataset_selection_tag",
              "site_name")])
```

    ##     intake_height site_elevation elevation dataset_selection_tag
    ##  1:          17.1         611.43    611.43       allvalid-17magl
    ##  2:          31.7         611.43    611.43       allvalid-32magl
    ##  3:           4.9         611.43    611.43        allvalid-5magl
    ##  4:         122.0         472.00    472.00      allvalid-122magl
    ##  5:          30.0         472.00    472.00       allvalid-30magl
    ##  6:         396.0         472.00    472.00      allvalid-396magl
    ##  7:         304.8         115.20    115.20      allvalid-305magl
    ##  8:          31.0         115.20    115.20       allvalid-31magl
    ##  9:          61.0         115.20    115.20       allvalid-61magl
    ## 10:          30.0           2.00      2.00       allvalid-30magl
    ## 11:         484.0           2.00      2.00      allvalid-483magl
    ## 12:          89.1           2.00      2.00       allvalid-91magl
    ##                                                        site_name
    ##  1: Carbon in Arctic Reservoirs Vulnerability Experiment (CARVE)
    ##  2: Carbon in Arctic Reservoirs Vulnerability Experiment (CARVE)
    ##  3: Carbon in Arctic Reservoirs Vulnerability Experiment (CARVE)
    ##  4:                                        Park Falls, Wisconsin
    ##  5:                                        Park Falls, Wisconsin
    ##  6:                                        Park Falls, Wisconsin
    ##  7:                                 Beech Island, South Carolina
    ##  8:                                 Beech Island, South Carolina
    ##  9:                                 Beech Island, South Carolina
    ## 10:                                     Walnut Grove, California
    ## 11:                                     Walnut Grove, California
    ## 12:                                     Walnut Grove, California

After filtering by space and time, we have 1.12883^{5} million
observations. Towers can have observations at different heights. Here we
need to select one site with the observations registered at the highest
height. The column with the height is named `altitude_final` and the max
altitude was named `max_altitude`.

``` r
if(any(names(df) %in% "intake_height")) {
  dfa <- df[,
            max(intake_height),
            by = site_code] |> unique()
  
  names(dfa)[2] <- "max_altitude"
} else {
  dfa <- df[,
            max(altitude_final),
            by = site_code] |> unique()
  
  names(dfa)[2] <- "max_altitude"
}
dfa
```

    ##    site_code max_altitude
    ## 1:       CRV         31.7
    ## 2:       LEF        396.0
    ## 3:       SCT        304.8
    ## 4:       WGC        484.0

### Key Time

Here we need to start time columns. The function `obs_addtime` adds time
columns `timeUTC`, `timeUTC_start` which shows the start time of each
observation and `timeUTC_end` which shows the end time for each
observation.

``` r
df2 <- obs_addtime(df)
```

    ## Adding timeUTC
    ## Adding timeUTC_start
    ## Adding timeUTC_end
    ## Found time_interval

Then we need a *key_time* to aggregate data. This can be done using UTC,
solar, or local time. The normal approach is using afteroon solar or
local time.

#### Hierarchy of solar or local time

1)  solar time

2)  local time with columns `site_utc2lst`

3)  local time longitude

4)  solar time (default)

Here we select the hours of interest and then aggregate data by year,
month and day of solar time. In this way, we will have one information
per day. however this approach is not appropiate for aircraft which are
aggregated every 10 or 20 seconds. Hence we need to aggregate data by
one time column. Also, this helps to generate the receptor info files
including hour, minute and second. Hence, *we need to add solar or local
time column*.

``` r
df2$solar_time <- obs_addstime(df2)
```

2)  local time with column `site_utc2lst`

Then we need to identify the local time with the function `add_ltime`.
This is important because to identifying observations in the evening in
local time. `add_ltime` uses two methods, first identify the time
difference with utc by identifying the metadata column “site_utc2lst”.
If solar time is not available \#now we need to cut solar time for the
frequency needed. As we will work with

3)  local time longitude

If this information is not available, with the aircrafts for instance,
the local time is calculated with an approximation based on longitude:

$$
lt = UTC + longitude/15 * 60 * 60
$$ Where $lt$ is the local time, $UTC$ the time, $longitude$ the
coordinate. Then, the time is cut every two hours. Now, we identify the
local time to select evening hours.

#### Cut time

Now we have they key column time, we can cut it accordingly.

``` r
df2$solar_time_cut <- cut(x = df2$solar_time,
                   breaks = "1 hour") |>
  as.character()
```

How we can check the solar time and the cut solar time. Please note that
solar_time_cut, the column that it will be used to aggregate data

``` r
df2[, c("solar_time", "solar_time_cut")]
```

    ##                  solar_time      solar_time_cut
    ##      1: 2019-12-02 00:49:35          2019-12-02
    ##      2: 2019-12-02 01:49:35 2019-12-02 01:00:00
    ##      3: 2019-12-02 02:49:35 2019-12-02 02:00:00
    ##      4: 2019-12-02 03:49:35 2019-12-02 03:00:00
    ##      5: 2019-12-02 04:49:35 2019-12-02 04:00:00
    ##     ---                                        
    ## 112879: 2021-01-31 11:10:52 2021-01-31 11:00:00
    ## 112880: 2021-01-31 12:10:52 2021-01-31 12:00:00
    ## 112881: 2021-01-31 13:10:52 2021-01-31 13:00:00
    ## 112882: 2021-01-31 14:10:52 2021-01-31 14:00:00
    ## 112883: 2021-01-31 15:10:52 2021-01-31 15:00:00

How we filter for the required solar time, in this case 14.

``` r
df3 <- df2[hour_st %in% evening]
```

Now there are 4316 observations and before filtering 112883. At this
point we can calculate the averages of several columns by the cut time.
The function `obs_agg` does this aggregation as shown in the following
lines of code. The argument `gby` establish the function used to
aggregate `cols`. I need to aggregate the data by date (year, month,
date), because it is already filtered by the hours of interest. Then, I
would have 1 observation per day.

As standard, let us define `key_time` as `solar_time`. The `obs_agg`
function will aggregate the desired data by that column.

``` r
df3$key_time <- df3$solar_time_cut
```

``` r
df4 <- obs_agg(dt = df3,
               gby = "mean",
               cols = c("value",
                        "latitude",
                        "longitude",
                        # "type_altitude",
                        "year_end",
                        "site_utc2lst"),
               verbose = T,
               byalt = TRUE)
```

    ## Selecting by alt
    ## Identified intake_height
    ## Adding time

Now there are 4316 observations, 108567 less observations. Here we add
the column `max_altitude` to identify the max altitude by site.

``` r
if(any(names(df) %in% "intake_height")) {
  df4[,
      max_altitude := max(intake_height),
      by = site_code]
  df4[,
      c("site_code",
        "intake_height",
        "max_altitude")] |> unique()
} else {
  df4[,
      max_altitude := max(altitude_final),
      by = site_code]
  df4[,
      c("site_code",
        "altitude_final",
        "max_altitude")] |> unique()
}
```

    ##     site_code intake_height max_altitude
    ##  1:       CRV          17.1         31.7
    ##  2:       CRV          31.7         31.7
    ##  3:       CRV           4.9         31.7
    ##  4:       LEF         122.0        396.0
    ##  5:       LEF         396.0        396.0
    ##  6:       LEF          30.0        396.0
    ##  7:       SCT         304.8        304.8
    ##  8:       SCT          31.0        304.8
    ##  9:       SCT          61.0        304.8
    ## 10:       WGC          30.0        484.0
    ## 11:       WGC         484.0        484.0
    ## 12:       WGC          89.1        484.0

### Saving master as text and csvy

Now that we have all the required information, we can save the files.
Here, we name the data.frame as master, because it contains all the
information. This is important because some fields can be used in the
future, and for traceability. For convenience, time variables are
transformed into character before writing into the disk. The separation
is space ” “.

``` r
master <- df3
master$timeUTC <- as.character(master$timeUTC)
fwrite(master, 
       file =  paste0(tempdir(), "/tower_insitu_2020.txt"),
       sep = " ")
```

The format Comma Separated Value with YAML (CSVY)\[^3\] consists in a
typical CSV with a YAML header. The function`obs_write_csvy` includes
the argument `notes` which allows adding custom notes at the header of
the file. Below the notes, `obs_write_csvy` adds the output of the R
function `str`, which provides a vertical summary of the data, known as
structure.

``` r
csvy <- paste0(tempdir(), "/tower_insitu_2020.csvy")
obs_write_csvy(dt = master,
               notes = "tower 2020",
               out = csvy
)
```

To check the YAML header we read the first 38 lines of the files that
were generated. Here we can see the column names, type of data and first
observations. The YAML header is delimited by the characters “—”.

``` r
readLines(csvy)[1:33]
```

### Saving receptors

We need to filter some columns from the master files in a new object
called receptors. This is needed because internally we run HYSPLIT
\[@hy\] using the information from the receptors. In the case of a
tower, we need to select observations with the highest altitude. The
specific columns are selected as shown on the following code. We are
selecting the ending times, because later HYSPLIT is run backwards based
on the time of measurement, between ending and starting times. The
columns about time are formatted to have two characters. For instance,
the month 1, is formatted as “01”. We also need to filter for
`type_altitude` equal 0, representing `agl`observations , or equal to 1,
`asl`.

``` r
if(any(names(df) %in% "intake_height")) {
receptor <- master[intake_height == max_altitude,
                   c("site_code",
                     "year",
                     "month",
                     "day",
                     "hour", 
                     "minute",
                     "second",
                     "latitude",
                     "longitude",
                     "intake_height",
                     "type_altitude",
                     "year_end",
                     "month_end",
                     "day_end",
                     "hour_end",
                     "minute_end",
                     "second_end")]
receptor$intake_height <- round(receptor$intake_height)

} else {
receptor <- master[altitude_final == max_altitude,
                   c("site_code",
                     "year",
                     "month",
                     "day",
                     "hour",
                     "minute",
                     "second",
                     "latitude",
                     "longitude",
                     "altitude_final",
                     "type_altitude",
                     "year_end",
                     "month_end",
                     "day_end",
                     "hour_end",
                     "minute_end",
                     "second_end")]

receptor$altitude_final <- round(receptor$altitude_final)
}




receptor <- obs_format(receptor)

if(nrow(receptor_agl) > 0) {
  fwrite(x = receptor_agl,
         file = paste0(tempdir(), "/receptor_tower_insitu_2020_AGL.txt"),
         sep = " ")}

if(nrow(receptor_asl) > 0) {
  fwrite(x = receptor_asl,
         file = paste0(tempdir(), "/receptor_tower_insitu_2020_ASL.txt"),
         sep = " ")}
```

## Application for other sectors

In this package we are sharing scripts to process other sectors The
scripts are available in the path
`https://github.com/ibarraespinosa/rtorf/tree/main/rscripts`

## Implementation in python:

I’m currently implementing a version in python:

| R                  | description                                     | Python |
|:-------------------|:------------------------------------------------|:-------|
| fex                | File extension                                  |        |
| invfile            | Methods for objects with class ‘invfile’        |        |
| obs_addltime       | local hour (bsed on longitude and time)         |        |
| obs_addstime       | Add solar time into obspack                     |        |
| obs_addtime        | Add times into obspack                          |        |
| obs_agg            | Aggregates ObsPack by time                      |        |
| obs_find_receptors | Compares expected receptors                     |        |
| obs_footname       | Expected footprint name                         |        |
| obs_format         | Formatting data                                 |        |
| obs_freq           | return numeric vector in intervals              |        |
| obs_index          | Summary of the ObsPack files (.txt)             | OK     |
| obs_invfiles       | Generate files to perform inverse modeling      |        |
| obs_list.dt        | list.dt                                         |        |
| obs_meta           | Read obspack metadata                           |        |
| obs_out            | outersect                                       |        |
| obs_plot           | Read obspack metadata                           |        |
| obs_rbind          | rbind obspack                                   |        |
| obs_read           | Read obspack (.txt)                             |        |
| obs_read_csvy      | reads CSVY                                      |        |
| obs_read_nc        | Read obspack (.nc)                              | OK     |
| obs_roundtime      | round seconds from “POSIXct” “POSIXt” classes   |        |
| obs_summary        | Summary of the ObsPack files (.txt)             |        |
| obs_table          | Obspack Table                                   |        |
| obs_trunc          | Trunc numbers with a desired number of decimals |        |
| obs_write_csvy     | Generates YAML and write data.frame             |        |
| plot.invfile       | Methods for objects with class ‘invfile’        |        |
| print.invfile      | Methods for objects with class ‘invfile’        |        |
| sr                 | Extacts n last characters                       |        |
| summary.invfile    | Methods for objects with class ’invfile         |        |
