# LimassolWildfireApp
//Code snippet analyse the Limassol Wildfires in Cyprus, July 2025

// Adding the Limassol boundaries from GEE
var adm1 = ee.FeatureCollection("FAO/GAUL/2015/level1")
  .filter(ee.Filter.eq('ADM0_NAME', 'Cyprus'));

var limassol = adm1.filter(ee.Filter.eq('ADM1_NAME', 'Limassol'));

print('Limassol district:', limassol);
Map.centerObject(limassol, 12);
Map.setCenter(32.8581, 34.7844, 12);  

// Time ranges for July 2025 fire incident
var preFireStart = '2025-07-01';
var preFireEnd = '2025-07-20';
var postFireStart = '2025-07-25';
var postFireEnd = '2025-07-31';

// Cloud mask function for Sentinel-2 SR
function maskS2sr(image) {
  var cloudProb = image.select('MSK_CLDPRB');
  var snowProb = image.select('MSK_SNWPRB');
  var cloud = cloudProb.lt(5);
  var snow = snowProb.lt(5);
  return image.updateMask(cloud).updateMask(snow);
}

// Sentinel-2 SR ImageCollection
var s2 = ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED");

// Pre-fire composite
var preFire = s2
  .filterBounds(limassol)
  .filterDate(preFireStart, preFireEnd)
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 10))
  .map(maskS2sr)
  .median();

// Post-fire composite
var postFire = s2
  .filterBounds(limassol)
  .filterDate(postFireStart, postFireEnd)
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 10))
  .map(maskS2sr)
  .median();

// Visualization: True Color
var vizRGB = {
  bands: ['B4', 'B3', 'B2'],
  min: 0,
  max: 3000,
  gamma: 1.3
};

Map.addLayer(preFire.clip(limassol), vizRGB, 'Pre-Fire (Limassol)');
Map.addLayer(postFire.clip(limassol), vizRGB, 'Post-Fire (Limassol)');

// Compute NBR pre- and post-fire ---
function computeNBR(image) {
  return image.normalizedDifference(['B8', 'B12']).rename('NBR');
}

var nbrPre = computeNBR(preFire);
var nbrPost = computeNBR(postFire);

// ΔNBR (difference) ---
var dNBR = nbrPre.subtract(nbrPost).rename('dNBR');

// Visualization of dNBR ---
var dnbrVis = {
  min: 0,
  max: 0.5,
  palette: ['white', 'yellow', 'orange', 'red', 'darkred']
};

// Mask non-burn areas (threshold 0.1) for transparency outside fire scar
var dNBRmasked = dNBR.updateMask(dNBR.gt(0.1));
Map.addLayer(dNBRmasked.clip(limassol), dnbrVis, 'ΔNBR Burn Severity (masked)');

// Threshold to extract burned areas (moderate-to-high severity) ---
var burnThreshold = 0.2;
var burnedArea = dNBR.gt(burnThreshold).selfMask();
// Map.addLayer(burnedArea.clip(limassol), {palette: ['red']}, 'Burned Area Mask');

// Step 3: Estimate burned area in km²
var pixelArea = ee.Image.pixelArea();
var burnedPixelArea = pixelArea.updateMask(burnedArea);

var stats = burnedPixelArea.reduceRegion({
  reducer: ee.Reducer.sum(),
  geometry: limassol.geometry(),
  scale: 20, // Sentinel-2 resolution
  maxPixels: 1e10
});

var areaSqMeters = ee.Number(stats.get('area'));
var areaSqKm = areaSqMeters.divide(1e6);
print('Estimated Burned Area in Limassol (km²):', areaSqKm);

// Legend for burn severity
var legend = ui.Panel({
  style: {
    position: 'bottom-left',
    padding: '8px 15px',
    backgroundColor: 'white'
  }
});

var legendTitle = ui.Label({
  value: 'Burn Severity (ΔNBR)',
  style: {fontWeight: 'bold', fontSize: '16px', margin: '0 0 6px 0'}
});
legend.add(legendTitle);

function makeRow(color, name) {
  var colorBox = ui.Label({
    style: {
      backgroundColor: color,
      padding: '8px',
      margin: '0 0 4px 0',
      border: '1px solid black',
      width: '20px',
      height: '20px'
    }
  });

  var description = ui.Label({
    value: name,
    style: {margin: '0 0 4px 6px'}
  });

  return ui.Panel({
    widgets: [colorBox, description],
    layout: ui.Panel.Layout.Flow('horizontal')
  });
}

legend.add(makeRow('yellow', 'Low severity'));
legend.add(makeRow('orange', 'Moderate severity'));
legend.add(makeRow('red', 'High severity'));
legend.add(makeRow('darkred', 'Very high severity'));
Map.add(legend);

// Severity thresholds
var lowThresh = 0.1;
var moderateThresh = 0.2;
var highThresh = 0.3;
var veryHighThresh = 0.4;

// Severity masks
var low = dNBR.gte(lowThresh).and(dNBR.lt(moderateThresh));
var moderate = dNBR.gte(moderateThresh).and(dNBR.lt(highThresh));
var high = dNBR.gte(highThresh).and(dNBR.lt(veryHighThresh));
var veryHigh = dNBR.gte(veryHighThresh);

function computeArea(mask) {
  var areaImage = ee.Image.pixelArea().updateMask(mask);
  var area = areaImage.reduceRegion({
    reducer: ee.Reducer.sum(),
    geometry: limassol.geometry(),
    scale: 20,
    maxPixels: 1e10
  }).get('area');
  
  var areaNumber = ee.Algorithms.If(area, ee.Number(area).divide(1e6), 0);
  return areaNumber;
}


// Calculate area for each severity class
var lowArea = computeArea(low);
var moderateArea = computeArea(moderate);
var highArea = computeArea(high);
var veryHighArea = computeArea(veryHigh);

// Print areas
print('Low severity area (km²):', lowArea);
print('Moderate severity area (km²):', moderateArea);
print('High severity area (km²):', highArea);
print('Very high severity area (km²):', veryHighArea);

// Create Pie Chart for burn severity proportions
var chart = ui.Chart.array.values({
  array: ee.Array([lowArea, moderateArea, highArea, veryHighArea]),
  axis: 0,
  xLabels: ['Low', 'Moderate', 'High', 'Very High']
})
.setChartType('PieChart')
.setOptions({
  title: 'Burn Severity Proportions',
  slices: {
    0: {color: 'yellow'},
    1: {color: 'orange'},
    2: {color: 'red'},
    3: {color: 'darkred'}
  },
  legend: 'none',
  pieSliceText: 'percentage',
  pieSliceTextStyle: {color: 'black'},
  backgroundColor: {fill: 'white'},
  titleTextStyle: {color: 'black', fontSize: 14},
  width: 300,
  height: 200
});

// Create a panel for the pie chart and place it at bottom right
var chartPanel = ui.Panel({
  widgets: [chart],
  style: {
    position: 'bottom-right',
    padding: '8px',
    backgroundColor: 'white'
  }
});
Map.add(chartPanel);
