# CartoDB integration for Cesium

This plugin helps integrate [CartoDB](http://cartodb.com) maps and data layers into [Cesium](http://cesiumjs.org).

## Server-side layer integration

CartoDB's server-side layers include basemaps and data layers generated in the server. For this, the pluging adds a new imagery provider to Cesium, `CartoDBImageryProvider`.

### Installation

You can use `.../src/cesium-cartodb.js` as is or run `grunt build` to get a minified version `.../build/cesium-cartodb.min.js`. You must include either after including Cesium in your html.

```html
<script src="<path_to>/Cesium.js"></script>
<script src="<path_to>/cesium-cartodb.min.js"></script>
```

### Usage

After this, you can create new imagery objects in Cesium and use them as needed. `CartoDBImageryProvider` is derived from [Cesium's OpenStreetMap imagery provider](http://cesiumjs.org/Cesium/Build/Documentation/OpenStreetMapImageryProvider.html), and can be initialized and used just the same.

#### Basemap layer

Just add a `CartoDBImageryProvider` instance to a viewer in order to use CartoDB's basemaps in Cesium.

```js
var basemapProvider = new Cesium.CartoDBImageryProvider({
    url: '<MAP_TEMPLATE_URL>', // e.g.: http://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png,
    credit: 'Basemap courtesy of CartoDB'
});
var viewer = new Cesium.Viewer('cesiumContainer', {
    imageryProvider: basemapProvider,
    baseLayerPicker: false,
    fullscreenButton: false,
    homeButton: false,
    timeline: false,
    navigationHelpButton: false,
    animation: false,
    scene3DOnly: true,
    geocoder: false
});
```

#### Data layer

In order to have a data layer from tiles generated in CartoDB, use cartodb.core.js's getTiles to inject the tiles in Cesium.

```js
var layerData = {
    user_name: '<USER_NAME',
    sublayers: [{
        sql: "<SQL_QUERY>",
        cartocss: '<CARTOCSS>'
    }]
};
cartodb.Tiles.getTiles(layerData, function (tiles, err) {
    if (tiles == null) {
        console.log("error: ", err.errors.join('\n'));
        return;
    }
    viewer.scene.imageryLayers.addImageryProvider(new Cesium.CartoDBImageryProvider({
        url: tiles.tiles[0],
        credit: 'Data courtesy of CartoDB'
    }));
});
```

## Client-side data layer

You can use CartoDB's SQL API to use data in CartoDB with Cesium renderers. You only need to make user data is in geoJSON format.

```js
var dataSource = new Cesium.GeoJsonDataSource();
viewer.dataSources.add(dataSource);

var sql = new cartodb.SQL({user: '<USER_NAME>', format: 'geoJSON'});
sql.execute("SQL_QUERY")
        .done(function (data) {
            dataSource.load(data)
                    .then(function () {
                    })
        })
        .error(function (errors) {
            // errors contains a list of errors
            console.log("errors:" + errors);
        });
```

## Examples

You can run a local HTTP server by running `grunt` in the root folder and then point your browser to http://127.0.0.1:8000/examples/ to see the examples.
