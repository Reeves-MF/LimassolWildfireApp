Limassol Wildfires Analysis – July 2025 (Google Earth Engine)
This Earth Engine script analyzes the July 2025 wildfires in Limassol, Cyprus using Sentinel-2 satellite imagery. It performs the following tasks:

Data Preparation: Loads administrative boundaries and filters Sentinel-2 surface reflectance data before and after the fire event.

Burn Severity Mapping: Calculates the Normalized Burn Ratio (NBR) and its difference (ΔNBR) to detect burn severity using standard thresholds.

Visualization: Displays true-color pre- and post-fire imagery and burn severity using a color-coded ΔNBR heatmap.

Burned Area Estimation: Computes the total area burned in square kilometers.

Severity Classification: Categorizes the burned area into low, moderate, high, and very high severity levels.

Interactive Legend and Pie Chart: Adds a legend and a pie chart to visualize the proportions of each burn severity class interactively.

The script is optimized for deployment on Earth Engine Apps and designed to support rapid fire assessment, disaster response, and environmental monitoring workflows.


Check here for the deployed app: https://fokengreeves.users.earthengine.app/view/limassol-wildfire-cyprus 
