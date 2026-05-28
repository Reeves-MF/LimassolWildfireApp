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
<img width="770" height="791" alt="image" src="https://github.com/user-attachments/assets/a2f1e8d1-e91a-47a8-bfce-cbe0c9479242" />
<img width="763" height="797" alt="image" src="https://github.com/user-attachments/assets/aeb45d8c-48f9-4c89-b5fd-bf78260d10d7" />
<img width="1904" height="813" alt="image" src="https://github.com/user-attachments/assets/5cb5f7c4-9c1f-40f3-8676-4d8b533eb48e" />



