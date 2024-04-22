    <!DOCTYPE html>
    <html>

    <head>
        
    <meta charset='utf-8' />
    <title></title>
    <meta name='viewport' content='initial-scale=1,maximum-scale=1,user-scalable=no' />
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.5.0/jquery.min.js"></script>
    <script src='https://api.mapbox.com/mapbox-gl-js/v2.0.1/mapbox-gl.js'></script>
    <link href='https://api.mapbox.com/mapbox-gl-js/v2.0.1/mapbox-gl.css' rel='stylesheet' />
    <script src='https://npmcdn.com/csv2geojson@latest/csv2geojson.js'></script>
    <script src='https://npmcdn.com/@turf/turf/turf.min.js'></script>
    <style>

         /* Adding Title and Description */
        body {
        margin: 0;
        padding: 0;
        }

        #description {
            position: absolute;
            top: 20px;
            left: 20px;
            width: 400px; /* Set the width to 400 pixels */
            height: 150px; /* Set the height to 200 pixels */
            background-color: #fff;
            padding: 10px;
            border-radius: 5px;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
            font-size: 12px; /* Set the font size to 16 pixels */
            color: #333; /* Set the text color to dark gray */
            font-family: Arial, sans-serif; /* Specify the font family */
            z-index: 1000; /* Set a high z-index value */
        }

        #map {
        position: absolute;
        top: 0;
        bottom: 0;
        width: 100%;
        }

        /* Popup styling */

        .mapboxgl-popup {
        padding-bottom: 5px;
        }

        .mapboxgl-popup-close-button {
        display: none;
        }

   .mapboxgl-popup-content {
    font: 400 15px/22px 'Roboto', 'Roboto Light', Sans-serif;
    padding: 0;
    width: 250px;
    max-height: 500px; /* Set maximum height for the popup */
    overflow-y: auto; /* Enable vertical scrolling */
    background-color: #fff; /* Set background color to white */
    border-radius: 5px; /* Add some border-radius for a softer look */
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* Add a subtle box shadow */
    }

    .mapboxgl-popup-content h3 {
    text-align: center;
    color: #333; /* Adjust text color to improve readability */
    margin: 0;
    padding: 15px;
    font-weight: 700;
    border-bottom: 1px solid #ccc; /* Add a bottom border for separation */
    }

    .mapboxgl-popup-content h4 {
    margin: 0;
    padding: 10px 15px; /* Adjust padding for better spacing */
    font-weight: 400;
    }

        .mapboxgl-container {
        cursor: pointer;
        }

        .mapboxgl-popup-anchor-top>.mapboxgl-popup-content {
        margin-top: 3px;
        }

        .mapboxgl-popup-anchor-top>.mapboxgl-popup-tip {
        border-bottom-color: rgb(61, 59, 59);
        }
    </style>
    </head>

    <body>    
        <div id="description">
             <h4>  Gaza Heritage Monitor

            </h3>

            We are visualizing the Gaza Damage Map by comparing satellite data with site photos to make it available as an interactive map for researchers and activists to visualize the damage done to Palestine buildings and people. The map helps to understand how satellite imagery could be biased in understanding the site.
            Data Point Sources: OPenStreetMap, United Nations Satellite Centre
        </div>
        <div id="map"></div>
    <script>

        var transformRequest = (url, resourceType) => {
        var isMapboxRequest =
            url.slice(8, 22) === "api.mapbox.com" ||
            url.slice(10, 26) === "tiles.mapbox.com";
        return {
            url: isMapboxRequest
            ? url.replace("?", "?pluginName=sheetMapper&")
            : url
        };
        };
        //YOUR TURN: add your Mapbox token
        
        mapboxgl.accessToken = 'pk.eyJ1Ijoib21nb3Jhc2hpIiwiYSI6ImNsczZpazA4MzFqbGIya3Awbzd3MWVqbTgifQ.JJn41ktf6QNQHglzu39Z1A'; //Mapbox token 
        var map = new mapboxgl.Map({
        container: 'map', // container id
        style: 'mapbox://styles/mapbox/satellite-v9', // YOUR TURN: choose a style: https://docs.mapbox.com/api/maps/#styles
        center: [34.3612727864078, 31.423879915023853], // starting position [lng, lat]
        zoom: 11,// starting zoom
        
        transformRequest: transformRequest
        });

        $(document).ready(function () {
        $.ajax({
            type: "GET",
            //YOUR TURN: Replace with csv export link
            url: 'https://docs.google.com/spreadsheets/d/1S5UsCUU3DHrik2O3mZ24oQSEyD1Ll5Ic/gviz/tq?tqx=out:csv&sheet=Sheet1',
            dataType: "text",
            success: function (csvData) { makeGeoJSON(csvData); }
        });

        //Padding, Title, Context, Rotation of map orientation... (Telling a story/narrative on the page//

        function makeGeoJSON(csvData) {
            csv2geojson.csv2geojson(csvData, {
            latfield: 'Latitude',
            lonfield: 'Longitude',
            delimiter: ','
            }, function (err, data) {
            map.on('load', function () {0

                //Add the the layer to the map
                map.addLayer({
                'id': 'csvData',
                'type': 'circle',
                'source': {
                    'type': 'geojson',
                    'data': data
                },
                'paint': {
                    'circle-radius': 2,
                    'circle-color': "red",
                }
                });
                // When a click event occurs on a feature in the csvData layer, open a popup at the
                // location of the feature, with description HTML from its properties.
                map.on('click', 'csvData', function (e) {
                var coordinates = e.features[0].geometry.coordinates.slice();

                //set popup text
                //You can adjust the values of the popup to match the headers of your CSV.
                // For example: e.features[0].properties.Name is retrieving information from the field Name in the original CSV.
                var description =
                    `<h3>` + e.features[0].properties.Name + `</h3>` + `<h4>` +
                    `Address: ` + `<b>` + e.features[0].properties.Address + `</b>` + `</h4>` + `<h4>` +
                    `<b>` + `Damage Status: ` + `</b>` + e.features[0].properties.DamageStatus + `</h4>` + `<br>` +
                    `Significance: ` + `<h3>` + e.features[0].properties.BuildingType + `</h3>` + `<br>` +
                    `<div class="image-container" style="text-align: center;">` +
                    `<div class="image-label">Before</div>` +
                    `<img src="` + e.features[0].properties.Before + `" style="max-width: 150px;" class="popup-image" alt="Before Image">` +
                    `</div>` +
                    `<div class="image-container" style="text-align: center;">` +
                    `<div class="image-label">After</div>` +
                    `<img src="` + e.features[0].properties.After + `" style="max-width: 150px;" class="popup-image" alt="After Image">` +
                    `</div>`;
                // Adding Additional Feature Layer info (columns)
                // Ensure that if the map is zoomed out such that multiple
                // copies of the feature are visible, the popup appears
                // over the copy being pointed to.
                while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
                    coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
                }

                //add Popup to map

                new mapboxgl.Popup()
                    .setLngLat(coordinates)
                    .setHTML(description)
                    .addTo(map);
                });

                // Change the cursor to a pointer when the mouse is over the places layer.
                map.on('mouseenter', 'csvData', function () {
                map.getCanvas().style.cursor = 'pointer';
                });

                // Change it back to a pointer when it leaves.
                map.on('mouseleave', 'csvData', function () {
                map.getCanvas().style.cursor = '';
                });

                var bbox = turf.bbox(data);
                map.fitBounds(bbox, { padding: 50 });

            });

            });
        };
        });




    </script>

    </body>

    </html>
