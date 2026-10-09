import geopandas as gpd
from blackmarble import BlackMarble, Product
import matplotlib.pyplot as plt
import osmnx as ox

# Bounding box
#gdf = gpd.GeoDataFrame(
#    geometry=[box(-80.54400344781475, 43.46312564957355, -80.54068886266587, 43.47688913023313)],
#    crs="EPSG:4326",
#)

# In this example ,we load a region from a GeoJSON.
gdf = gpd.read_file(r"C:\Users\joyce\Documents\VS code\Black marble\Municipal_Boundary_-8906112526470909522.geojson")

# 2. Authenticate (assumes BLACKMARBLE_TOKEN is set as an env var)
bm = BlackMarble(output_directory="blackmarble_data", token="eyJ0eXAiOiJKV1QiLCJvcmlnaW4iOiJFYXJ0aGRhdGEgTG9naW4iLCJzaWciOiJlZGxqd3RwdWJrZXlfb3BzIiwiYWxnIjoiUlMyNTYifQ.eyJ0eXBlIjoiVXNlciIsInVpZCI6ImpveWNlbWVuZyIsImV4cCI6MTc5NDM1ODc0MSwiaWF0IjoxNzg5MTc0NzQxLCJpc3MiOiJodHRwczovL3Vycy5lYXJ0aGRhdGEubmFzYS5nb3YiLCJpZGVudGl0eV9wcm92aWRlciI6ImVkbF9vcHMiLCJhY3IiOiJlZGwiLCJhc3N1cmFuY2VfbGV2ZWwiOjN9.06XNEMz3lrSf8W3nelDASDUQZzs5coXGUkLkcaAqoMMEkD8jPZj4v8hhg-g3JUspYC83InirQfqJzPeIUUli_94tIwmvVJAQYA2k9V6JANCuW20UL79mcVRivXOw9bg1sc_jWtlrnAPH3n1qYWGbcBcTqwW4WDQ5jJDrPFxmyZ3v47PWw0jKrbNTqw4Y-H0e-pebfJBv8bAAi6LYku3pSml3kaaBZRqlLD15Uso-JLdN7VdlH1AsMn4RdJvpwNtaTd7tcsuGmzZJtsV6eF9Mo1p6dV0jKj8Ks4iDEhPg2lB3WiXbeQIGef8UvrgBJ-DO7BJfSl3stln40SfmW1YpLw")

# 3. Download VNP46 data

# Download the graph FIRST, before the loop
place_name = "Waterloo, Ontario, Canada"
G = ox.graph_from_place(place_name, network_type="all")
G = ox.project_graph(G, to_crs="EPSG:4326") #reprojects graph to WGS84 (EPSG:4326) for compatibility with BlackMarble data

dates = [
    "2024-06-15", "2024-07-15", "2024-08-15",
    "2024-09-15", "2024-10-15", "2024-11-15", "2024-12-15",
    "2025-01-15", "2025-02-15", "2025-03-15", "2025-04-15", "2025-05-15",
]

fig, axes = plt.subplots(1, len(dates), figsize=(6 * len(dates), 6)) # 1 row, len(dates) columns

# Loop through the dates and plot the raster
for ax, d in zip(axes, dates):
    raster = bm.raster(gdf, product_id=Product.VNP46A3, date_range=d)
    raster["NearNadir_Composite_Snow_Free"].plot(
        ax=ax, vmin=0, vmax=100, cmap="magma"
    )

    # Draw streets on this panel
    ox.plot_graph(
        G,
        ax=ax,
        edge_color="cyan",
        edge_linewidth=0.1,
        node_size=0,
        bgcolor="none",
        show=False,
        close=False,
    )
    ax.set_title(d)

plt.savefig("compare.png", dpi=150, bbox_inches="tight")
