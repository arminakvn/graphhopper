<template>


        <map-module
                v-bind:features="items"
                v-bind:anchors="anchors"
                v-bind:waypoints="waypoints"

        ></map-module>

    <!--</div>-->
</template>

<script>


    import * as d3 from 'd3'

    import MapModule from './MapModule.vue'


    export default {
        name: 'MapView',
        components:{
          MapModule
        },
        methods:{
            // updateMap: function () {
            //     const self  = this;
            //     // console.log(self.items[0].path)
            //     let lineFeat = {'type': 'Feature',
            //         'properties': {
            //             'color': '#F7455D',
            //             'width': '1'
            //         },
            //         'geometry':{
            //             'type': 'LineString',
            //             'coordinates': []
            //         }}
            //
            //     let dataLayer = {
            //         'id': 'lines',
            //         'type': 'line',
            //         'source': {
            //             'type': 'geojson',
            //             'data': {
            //
            //                 'type': 'FeatureCollection',
            //                 'features': [{
            //
            //                 }]
            //             }
            //
            //         }
            //     }
            //
            // }
        },
        computed:{

        },
        data: function(){
            return {
                    items:[],
                    anchors:[],
                    dataitems:[],
                    waypoints:[],
                    currentRoute:null
            }
        },
        watch:{

                '$route.params.route':{
                        handler(){
                                this.currentRoute = this.$route.params.route;
                                this.$emit('updateRoute',this.currentRoute)
                        }
                }
        },
        mounted() {
                // console.log($route.params.route)

                let routeChoice = "home to mapc";
                let num_of_alterntves = 1;
                let encoderFlg = "mapcrider2";




                ////////
                        //encoderFlg
                let mapdata =new Map();
                let routesDataMap = new Map();

                // mapdata.set("mapcrider",[["darkblue","lighblue"]])
                mapdata.set("encoderFlg",encoderFlg)
                const self = this;
                let lineScale = d3.scalePow()
                        .range([1,8]);
                console.log("self in MapView", self)
                let files = [];
                routesDataMap.set("mapc to neu", "point=42.355278%2C-71.0616&point=42.337207%2C-71.089543");
                routesDataMap.set("neu to mapc", "point=42.337207%2C-71.089543&point=42.355278%2C-71.0616");
                routesDataMap.set("mapc to home", "point=42.355097%2C-71.061361&point=42.318246%2C-71.105052")
                routesDataMap.set("home to mapc", "point=42.318246%2C-71.105052&point=42.355097%2C-71.061361")
                routesDataMap.set("home to neu", "point=42.318246%2C-71.105052&point=42.337043%2C-71.089667")
                // if (num_of_alterntves>1)

                for (let [key, value] of mapdata) {
                                if (num_of_alterntves<2) {
                                        files.push("http://0.0.0.0:8989/route?"+routesDataMap.get(routeChoice)+"&locale=en-US&vehicle=" + value + "&weighting=fastest&points_encoded=false&elevation=true&use_miles=true&details=weight_value&details=street_name&details=edge_id&details=average_speed&details=facilities_overal")

                                } else {
                                files.push(
                                        "http://0.0.0.0:8989/route?"+routesDataMap.get(routeChoice)+"&locale=en-US&vehicle=" + value + "&weighting=fastest&points_encoded=false&algorithm=alternative_route&alternative_route.max_paths=" + num_of_alterntves + "&alternative_route.max_share_factor=0.3&elevation=true&use_miles=true&details=weight_value&details=street_name&details=edge_id&details=average_speed"
                                )
                                // files.push(
                                //         "http://0.0.0.0:8989/route?point=42.338875%2C-71.088759&point=42.355695%2C-71.060661&locale=en-US&vehicle="+key+"&weighting=fastest&points_encoded=false&elevation=true&use_miles=true&details=weight_value&details=street_name&details=edge_id&details=average_speed"
                                // )
                        }
                }

                console.log(files.length)
                Promise.all(
                        files.map(
                                url => d3.json(url)
                        )
                ).then(function(d) {
                        let dateArr = [];
                        let anchorArr = [];
                        let waypointArr = [];
                        let dataitems = [];
                        let colors = [
                                ["darkred","red"],
                                ["darkred","lightred"],
                                ["darkgreen","lightgreen"]
                        ];
                        let weight_range = [];
                        for (let jo = 0; jo < files.length; jo++) {
                                for (let j = 0; j < num_of_alterntves; j++) {
                                        dataitems.push(d[jo]["paths"][j])

                                        let total_weight =d[jo]["paths"][j]["weight"];
                                        let total_distance =d[jo]["paths"][j]["distance"];
                                        let starts_from_lon = d[jo]["paths"][j]["points"]["coordinates"][0][0]
                                        let starts_from_lat = d[jo]["paths"][j]["points"]["coordinates"][0][1]
                                        //anchorArr[joz]
                                        waypointArr.push(

                                                {
                                                        'type': 'Feature',
                                                        'properties': {
                                                                // 'total_weight': "total weight: " + Math.round(total_weight),
                                                                // 'total_distance':total_distance,
                                                                // // 'color': color(i),
                                                                // // 'width': lineScale(weight_val),
                                                                'opacity': 0.7,
                                                                // 'title': aver_spd

                                                        },
                                                        'geometry': d[jo]["paths"][j]["snapped_waypoints"]
                                                }
                                        )
                                        anchorArr.push(
                                                {
                                                        'type': 'Feature',
                                                        'properties': {
                                                                'total_weight': "total weight: " + Math.round(total_weight),
                                                                'total_distance':total_distance,
                                                                // 'color': color(i),
                                                                // 'width': lineScale(weight_val),
                                                                'opacity': 0.7,
                                                                // 'title': aver_spd

                                                        },
                                                        'geometry': d[jo]["paths"][j]["points"]
                                                }



                                        )
                                        d[jo]["paths"][j]["details"]["weight_value"].forEach(function (event) {
                                                weight_range.push(event[2])
                                        })
                                }
                        }
                        self.items = dataitems;
                        let dataDomain = d3.extent(weight_range);
                        lineScale.domain(dataDomain);
                        console.log("extent",dataDomain);
                        for (let joz = 0; joz < files.length; joz++) {
                                let color = d3.scaleLinear()
                                        .domain([0, files.length])
                                        .range(colors[joz])
                                        .interpolate(d3.interpolateHcl);
                                for (let i = 0; i < num_of_alterntves; i++) {
                                        d[joz]["paths"][i]["details"]["weight_value"].forEach(function(event){
                                                weight_range.push(event[2])
                                        })
                                        console.log(d[joz]["paths"][i]["details"]["average_speed"]);console.log(d[joz]["paths"][i]["details"]["weight_value"]);
                                        d[joz]["paths"][i]["details"]["weight_value"].forEach(function(event){
                                                let event_from_id = event[0];
                                                let event_to_id = event[1];
                                                let aver_spd = "segment ave speed: "
                                                // find the average_speed interval and get the value of that
                                                // for each interval pair from the average_speeds details check the events from and to id
                                                d[joz]["paths"][i]["details"]["average_speed"].forEach(function (a_s_event) {
                                                        let from_id = a_s_event[0]
                                                        let to_id = a_s_event[1]
                                                        if (event_from_id >= from_id && event_from_id <= to_id){
                                                                if (event_to_id >=from_id && event_to_id <= to_id){
                                                                        aver_spd = aver_spd + a_s_event[2]
                                                                        // console.log("event_from_id ", event_from_id,"event_to_id", event_to_id,"from_id",from_id,"to_id",to_id)

                                                                }
                                                        }
                                                })

                                                let fromlon = d[joz]["paths"][i]["points"]["coordinates"][event[0]][0]
                                                let fromlat = d[joz]["paths"][i]["points"]["coordinates"][event[0]][1]
                                                let tolon = d[joz]["paths"][i]["points"]["coordinates"][event[1]][0]
                                                let tolat = d[joz]["paths"][i]["points"]["coordinates"][event[1]][1]
                                                let weight_val = event[2]
                                                // ancG["geometry"]["coordinates"].push([[fromlon,fromlat],[tolon,tolat]])
                                                dateArr.push(
                                                        {
                                                                'type': 'Feature',
                                                                'properties': {
                                                                        'color': color(i),
                                                                        'width': lineScale(weight_val),
                                                                        'opacity': 0.7,
                                                                        'title': "weight: " + Math.round(weight_val)

                                                                },
                                                                'geometry':{
                                                                        'type': 'LineString',
                                                                        'coordinates': [[fromlon,fromlat],[tolon,tolat]]
                                                                }}
                                                                )
                                        })
                                }
                        }
                        self.items = dateArr;
                        self.dataitems = dataitems;
                        self.anchors = anchorArr;
                        self.waypoints = waypointArr;
                });
        }
    }
</script>
