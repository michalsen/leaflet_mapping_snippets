// PHP
```bash
function mapgroove_enqueue_style()
{
    wp_enqueue_style('leafletCSS', plugins_url() . "/mapgroove/assets/css/leaflet.css");
    wp_enqueue_style('leafletCSS');
    wp_enqueue_style('mapgrooveCSS', plugins_url() . "/mapgroove/assets/css/mapgroove.css");
    wp_enqueue_style('mapgrooveCSS');
}

function mapgroove_enqueue_script()
{
    wp_enqueue_script('jquery');
    wp_enqueue_script('jquery-ui-sortable');
    wp_enqueue_script('leaflet.js', plugins_url() . "/mapgroove/assets/js/leaflet.js", false);
    wp_enqueue_script('leaflet.js');
    wp_enqueue_script('mapgrooveJS', plugins_url() . "/mapgroove/assets/js/mapgroove.js");
    wp_enqueue_script('mapgrooveJS');
}

add_action('wp_enqueue_scripts', 'mapgroove_enqueue_style');
add_action('wp_enqueue_scripts', 'mapgroove_enqueue_script');
```
// JS
```bash
function loadMap() {
    LeafIcon = L.Icon.extend({
        options: {
            iconSize:    [25, 25],
            // iconAnchor:  [10, 0],
            // popupAnchor: [0, 0]
        }
    });

    blueIcon = new LeafIcon({
        iconUrl: '/wp-content/plugins/mapgroove/includes/icons/mapicon.png',
    })

    map = L.map('mapid').setView([39.8097343, -98.5556199], 5);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: 'A Straight North Map',
    }).addTo(map);
    // map.scrollWheelZoom.disable();
    // map.dragging.disable();
}
```

```bash
jQuery.ajax({
        type: "POST",
        url: '/wp-content/plugins/mapgroove/listings.php',
        data: {'location_-_state':state,
            'location_-_region':region,
            'services':services,
            'markets':markets,
            'companies':companies,
            'careers':careers
        },
        success: function(data) {
            // console.log(data);
            var latlngs = [];
            var couple = [];

            $.each(JSON.parse(data), function (idx, obj) {
                // console.log(idx);
                // console.log(obj['lat']);
                latlngs.push(
                    [obj['lat'], obj['lng']]
                );
            });

            /**
             *
             * Distance Evaluation
             */
            var locations = [],
                latLongPair,
                longestDistances,
                savecompare = [],
                savecomparisons = [];

            $.each(latlngs, function (idx, obj) {
                locations.push(obj);
            });
            if (locations.length > 1) {
                longestDistances = locations.map(function (outerLatLong) {
                    var comparisons = locations.map(function (innerLatLong) {
                        var first = outerLatLong.toString();
                        var second = innerLatLong.toString();
                        latLongPair = first + ',' + second;
                        checkPair = latLongPair.split(',');
                        var checkdistance = distanceto(checkPair[0], checkPair[1], checkPair[2], checkPair[3], latLongPair);
                        savecompare.push(checkdistance);
                        return checkdistance;
                    });
                    savecomparisons.push(comparisons[1][0]);
                    var check = Math.max.apply(null, savecomparisons);
                    return check;
                });
                var longestDistance = Math.max.apply(null, longestDistances);

                var loop = savecompare.length;
                for (var i = 0; i < loop; i++) {
                    if (savecompare[i][0] == longestDistance) {
                        checkPair = savecompare[i][1].split(',');
                        var point1 = new L.LatLng(checkPair[0], checkPair[1]);
                        var point2 = new L.LatLng(checkPair[2], checkPair[3]);
                        var bounds = new L.LatLngBounds(point1, point2);
                        break;
                    }
                }
            }
            /**
             *
             * END Distance Evaluation
             */

            $(".leaflet-marker-icon").remove();
            $(".leaflet-popup").remove();
             $.each(JSON.parse(data), function (idx, obj) {
                  var marker = L.marker([obj['lat'], obj['lng']], {icon: blueIcon})
                    .addTo(map)
                    .bindPopup(obj['popup_text']);
                    set_latlng = [obj['lat'], obj['lng']];
            });

            if (bounds) {
                map.fitBounds(bounds, {padding: [100, 100]});
            }
            else  {
                map.setView(set_latlng, 6);
            }
        },
        error: function(){ },
        complete: function(){ }
    });

    $('.leaflet-control-attribution').hide();
});

```

```bash
jQuery(document).ready(function($) {

    // MAP RESET
    $('#map_reset').click(function(e) {
        $("select").each(function() { this.selectedIndex = 0 });
        location.reload();
    });

    // MAP STATE/REGION SWITCH
    $( "#selectstate" ).click(function() {
        $( "#selectregion" ).val(0);
    });
    $( "#selectregion" ).click(function() {
        $( "#selectstate" ).val(0);
    });

    // Submit API Key
    $('#api_button').click(function(e) {
        var pressed = $('#api').val();
        api(pressed);
    });

```
