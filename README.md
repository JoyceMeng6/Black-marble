import geopandas as gpd
from blackmarble import BlackMarble, Product
import matplotlib.pyplot as plt
import osmnx as ox

# 1. Load a region from a GeoJSON
gdf = gpd.read_file(r"C:\Users\joyce\Documents\VS code\Black marble\Municipal_Boundary_-8906112526470909522.geojson")

# 2. Authenticate
bm = BlackMarble(
output_directory="blackmarble_data", token=""
check_all_tiles_exist=True,
drop_values_by_quality_flag=[255],
output_skip_if_exists=True
)

# 3. Download VNP46 data
# Download the graph FIRST, before the loop
place_name = "Waterloo, Ontario, Canada"
G = ox.graph_from_place(place_name, network_type="all")
G = ox.project_graph(G, to_crs="EPSG:4326") #reprojects graph to WGS84 (EPSG:4326) for compatibility with BlackMarble data

ax.set_title(
    "NASA Black Marble: Nighttime Lights Radiance",
    fontsize=16,
)

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

fig, ax = plt.subplots(figsize=(10, 5))

####################################################################################
'''
# Raster stack of monthly data
r_monthly = bm.raster(
    gdf,
    product_id="VNP46A3",
    date_range=pd.date_range("2024-06-15", "2025-05-15", freq="MS").date.tolist(),
)
'''

# Plot NTL radiance
raster["NearNadir_Composite_Snow_Free"].sum(dim=["x", "y"]).plot.line(
    ax=ax, color="purple", marker="o"
)

ax.set_title("Monthly Aggregate Nighttime Lights Radiance", fontsize=18, weight="bold")
ax.set_xlabel("Month", fontsize=12)
ax.set_ylabel("Radiance (nW/cm²/sr)", fontsize=12)
plt.ylim(bottom=0)
ax.text(
    0,
    -0.2,
    "Source: NASA Black Marble VNP46A3",
    ha="left",
    va="center",
    transform=ax.transAxes,
    fontsize=10,
    color="black",
    weight="normal",
)

fig.tight_layout()
plt.show()
