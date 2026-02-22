<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Culebra Map</title>
    
    <!-- 1. THE BRAIN: These must be in the <head> to load the map software -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" crossorigin="" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" crossorigin=""></script>

    <style>
        /* 2. THE BOX: This forces the map to actually be visible */
        #map { height: 100vh; width: 100%; background: #ddd; }
        body { margin: 0; padding: 0; }
    </style>
</head>
<body>

    <!-- 3. THE CONTAINER: This is where the map will sit -->
    <div id="map"></div>

    <script>
        // 4. THE LOGIC: This script runs the map
        document.addEventListener('DOMContentLoaded', function() {
            // Setup the map view centered on Culebra
            var map = L.map('map').setView([18.31, -65.30], 13);

            // Load the actual map tiles from OpenStreetMap (HTTPS)
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '&copy; OpenStreetMap'
            }).addTo(map);

            // Your Locations
            var locations = [
                { name: "Todo Culebra Plaza Hub", lat: 18.3012, lng: -65.30213, desc: "www.shopculebra.com" },
                { name: "Zaco's Tacos", lat: 18.300, lng: -65.29558, desc: "Zacos Tacos" },
                { name: "Playa Flamenco", lat: 18.3280, lng: -65.3358, desc: "Snorkel with Locals" },
                { name: "Playa Zoni", lat: 18.3197, lng: -65.2757, desc: "Culebra Camping" }
            ];

            // Add the Pins to the map
            locations.forEach(function(p) {
                L.marker([p.lat, p.lg || p.lng]).addTo(map)
                    .bindPopup("<b>" + p.name + "</b><br>" + (p.desc || ""));
            });
            
            // Final nudge for Safari
            setTimeout(function(){ map.invalidateSize(); }, 500);
        });
    </script>
</body>
</html>
