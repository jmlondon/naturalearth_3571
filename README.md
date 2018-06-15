
<!-- README.md is generated from README.Rmd. Please edit that file -->

# naturalearth\_3571\_tiles

The goal of naturalearth\_3571\_tiles is to provide tile maps for use
within the `leaflet` package that are projected in WGS 84 / North Pole
LAEA Bering Sea (EPSG:3571). The tiles are derived from the Natural
Earth II dataset.

The initial command line calls to `gdal`
are

``` bash
gdal_translate -a_srs "+proj=longlat +ellps=sphere" NE2_HR_LC_SR_W_DR.tif ne.tif
gdalwarp -te -180 0 180 90 ne.tif ne_crop.tif
gdalwarp -dstnodata 0 -ot Byte -wo SOURCE_EXTRA=1000 -t_srs "EPSG:3571" \
-r lanczos -of GTiff  ne_crop.tif naturalearth_3571.tif
~/anaconda3/bin/gdal2tiles.py --profile=raster -z 3-8 \
naturalearth_3571.tif naturalearth_3571
```

``` r
tile_url <- "naturalearth_3571/{z}/{x}/{y}.png"
```

The `leafletCRS` function requires us to provide various parameters all
of which are listed in the `tilemapresource.xml` file.

``` r
crs <- leafletCRS(crsClass = "L.Proj.CRS", code = "EPSG:3571",
                  proj4def = "+proj=laea +lat_0=90 +lon_0=180 +x_0=0 +y_0=0 +ellps=WGS84 +datum=WGS84 +units=m +no_defs",
  resolutions = c(9156.71827398535061,
                  4578.35913699267530,
                  2289.17956849633765,
                  1144.58978424816883,
                  572.29489212408441,
                  286.14744606204221),
  origin = c(-9009964.76123128645122, -9010456.80197188444436),
  bounds = list(c(-9009964.76123128645122, -9010456.80197188444436),
                c(9010456.80197188444436, 9009964.76123128645122))
  )
```

Here we create an example trackline that crosses the 180 longitude
line

``` r
track_line <- 'LINESTRING (-169.4444 62.57549, -169.8999 62.31869, -170.4738 62.16436, -170.6529 61.4344, -171.7103 60.5111, -173.5295 60.78438, -175.2325 61.55942, -176.0107 61.88525, -177.1819 61.43621, -177.3077 61.25797, -177.3493 61.98553, -176.4385 62.26207, -177.3228 62.73083, -179.2457 62.84639, 179.7168 62.67369, 179.8549 62.58532, 179.8978 62.60145, -179.7893 62.64932, 179.9897 62.37538, -179.6393 62.66016, -179.8876 62.60982, -179.68 62.75012, 179.8264 62.78815, -179.789 62.72092, 179.6381 62.48176)'

track_line <- sf::st_as_sfc(track_line) %>% 
  sf::st_set_crs(4326)
```

And, now our call to create the leaflet map

``` r
leaflet(options = leafletOptions(crs = crs)) %>%
  addTiles(urlTemplate = tile_url,
           options = tileOptions(noWrap = TRUE,
                      continuousWorld = FALSE))
```

![](README_files/figure-gfm/leaflet-map-1.png)<!-- -->

## Let’s try and create tiles with `tiler`

`tiler` is a new package that should create tiles in the same manner as
our use of `gdal2tiles.py`.

``` r
devtools::install_github("leonawicz/tiler", ref = "projections")
#> Using GitHub PAT from envvar GITHUB_PAT
#> Skipping install of 'tiler' from a github remote, the SHA1 (5e5681fd) has not changed since last install.
#>   Use `force = TRUE` to force installation
```

``` r
library(tiler)
map <- "naturalearth_3571.tif"
tile_dir <- "tiles"
tile(map, tile_dir, "3-7", profile = "raster")
```
