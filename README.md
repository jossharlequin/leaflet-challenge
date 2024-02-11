# leaflet-challenge

// Create the map
let myMap = L.map("map", {
    center: [61.2176, -149.8997],
    zoom: 2
});

// Define a markerSize function
function markerSize(magnitude) {
    return Math.sqrt(magnitude) * 5;
}

// Add tile layer (replace with your preferred tile layer)
L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png").addTo(myMap);

// Load earthquake data
let earthquakeDataUrl = "https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/significant_month.geojson";

// Fetch earthquake data using AJAX (https://www.freecodecamp.org/news/how-to-use-fetch-api/)
fetch(earthquakeDataUrl)
    .then(response => response.json())
    .then(earthquakeData => {
        // Define arrays to hold the created earthquake markers.
        let earthquakeMarkers = [];

        // Loop through earthquakeData and create the earthquake markers.
        earthquakeData.features.forEach(feature => {
            const magnitude = feature.properties.mag;
            const depth = feature.geometry.coordinates[2];

            const circleMarker = L.circleMarker([feature.geometry.coordinates[1], feature.geometry.coordinates[0]], {
                radius: markerSize(magnitude),
                fillColor: getColor(depth),
                color: "white",
                weight: 1,
                opacity: 0.75,
                fillOpacity: 0.8,
            });

            circleMarker.bindPopup(function () {
                let popupContent = "<b>Location:</b> " + feature.properties.place +
                    "<br><b>Magnitude:</b> " + magnitude +
                    "<br><b>Depth:</b> " + depth;
            
                // Additional messages based on magnitude and depth (https://www.sitepoint.com/community/t/javascript-if-else-and-prompt-and-alert-messages/365320)
                if (magnitude > 9) {
                    popupContent += "<br>This is Yuge!";
                } else if (magnitude > 7) {
                    popupContent += "<br>This is big!";
                } else if (magnitude > 5) {
                    popupContent += "<br>This is mid.";
                } else if (magnitude > 3) {
                    popupContent += "<br>This is fine.";
                } else {
                    popupContent += "<br>Was there an earthquake?";
                }
            
                if (depth > 50) {
                    // Include multi-line strings for lyrics
                    popupContent += "<br>We could've had it all!";
                    popupContent += "<br>Rolling in the Deep.";
                } else {
                    popupContent += "<br>In the sha-ha, sha-hallow;";
                    popupContent += "<br>In the sha-ha, sha-la-la-la-low;";
                    popupContent += "<br>In the sha-ha, sha-hallow;";
                    popupContent += "<br>We're far from the shallow now.";
                }
            
                return popupContent;
            });

            earthquakeMarkers.push(circleMarker);
        });

        // Add the earthquake markers to a layer group
        let earthquakeLayer = L.layerGroup(earthquakeMarkers);

        // Add the layer group to the map
        earthquakeLayer.addTo(myMap);

        // Define a getColor function based on earthquake depth
function getColor(depth) {
    return depth > 90 ? "#800026" :
        depth > 70 ? "#BD0026" :
        depth > 50 ? "#E31A1C" :
        depth > 30 ? "#FC4E2A" :
        depth > 10 ? "#FD8D3C" :
        "#FEB24C";
}

// Create a legend (https://gis.stackexchange.com/questions/193161/add-legend-to-leaflet-map)
let legend = L.control({ position: 'bottomright' });

legend.onAdd = function (map) {
    let div = L.DomUtil.create('div', 'info legend');
    let depths = [0, 10, 30, 50, 70, 90];

    // Add title to the legend
    div.innerHTML = '<h4>Depths</h4>';

    // Loop through the depths and create a colored square for each depth
    for (let i = 0; i < depths.length; i++) {
        div.innerHTML +=
    '<i style="background:' + getColor(depths[i] + 1) + '; width: 10px; height: 10px; display: inline-block;"></i> ' +
    depths[i] + (depths[i + 1] ? '&ndash;' + depths[i + 1] + '<br>' : '+');
    }

    // Add a white background and padding to the legend
    div.style.backgroundColor = 'white';
    div.style.padding = '8px';

    // Make sure to return the div
    return div;
};

// Add the legend to the map
legend.addTo(myMap);
})
.catch(error => {
    console.error('Error during fetch operation:', error);
});
