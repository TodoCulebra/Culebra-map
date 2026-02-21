<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Culebra Map</title>
    <!-- Load the map brain -->
    <link rel="stylesheet" href="https://unpkg.com" />
    <script src="https://unpkg.com"></script>
    <style>
        #map { height: 100vh; width: 100%; margin: 0; }
        body { margin: 0; padding: 0; }
    </style>
</head>
<body>
    <div id="map"></div>
    <script>
        var map = L.map('map').setView([18.31, -65.30], 13);
        L.tileLayer('https://tile.openstreetmap.org{z}/{x}/{y}.png').addTo(map);

        var locations = [
            { name: "Todo Culebra Plaza Hub", lat: 18.3012, lng: -65.30213 },
            { name: "Zaco's Tacos", lat: 18.300, lng: -65.2955823 },
            { name: "Playa Flamenco", lat: 18.3280301, lng: -65.3358813 },
            { name: "Playa Zoni", lat: 18.3197313, lng: -65.2757021 }
        ];

        locations.forEach(function(p) {
            L.marker([p.lat, p.lng]).addTo(map).bindPopup(p.name);
        });
    </script>
</body>
</html>
