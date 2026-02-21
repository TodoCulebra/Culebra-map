<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Culebra Map</title>
    
    <!-- 1. The Map's Brain (Loaded FIRST in the head) -->
    <link rel="stylesheet" href="https://unpkg.com" />
    <script src="https://unpkg.com"></script>

    <style>
        /* 2. Fix: Ensures map is NEVER 0px tall */
        html, body, #map { height: 100%; width: 100%; margin: 0; padding: 0; background: #eee; }
    </style>
</head>
<body>

    <div id="map"></div>

    <script>
        // 3. Wait for everything to load before starting
        window.onload = function() {
            var map = L.map('map').setView([18.31, -65.30], 13);

            // 4. Load Map Images
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '&copy; OpenStreetMap'
            }).addTo(map);

            // 5. Your Pins
            var locations = [
                { n: "Todo Culebra Plaza Hub", lt: 18.3012, lg: -65.30213 },
                { n: "Zaco's Tacos", lt: 18.300, lg: -65.29558 },
                { n: "Playa Flamenco", lt: 18.3280, lg: -65.3358 },
                { n: "Playa Zoni", lt: 18.3197, lg: -65.2757 }
            ];

            locations.forEach(function(p) {
                L.marker([p.lt, p.lg]).addTo(map).bindPopup(p.n);
            });

            // 6. THE CRITICAL FIX: Forces the map to "wake up" and redraw itself
            setTimeout(function() {
                map.invalidateSize();
            }, 400);
        };
    </script>
</body>
</html>
