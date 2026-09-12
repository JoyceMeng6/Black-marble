# Black-marble
# I made it so that u would have to get your own token but i can change that if you want

import geopandas as gpd
from blackmarble import BlackMarble, Product
from shapely.geometry import box

# Bounding box
gdf = gpd.GeoDataFrame(
    geometry=[box(-80.54400344781475, 43.46312564957355, -80.54068886266587, 43.47688913023313)],
    crs="EPSG:4326",
)

# 2. Authenticate (assumes BLACKMARBLE_TOKEN is set as an env var)
bm = BlackMarble()

# 3. Download VNP46 data
#    The result is an xarray.Dataset object.
raster_earth_day = bm.raster(
    gdf,
    product_id=Product.VNP46A4,
    date_range="2025-01-15",  # Format is YYYY-MM-DD
)

# Optional: See what you got
print(raster_earth_day)
