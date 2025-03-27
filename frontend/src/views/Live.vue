<template>
    <v-container bg-color="light-blue" fluid align="center">
        <v-row style="max-width: 1200px;">
            <v-col cols="9">
                <figure class="highcharts-figure">
                    <div id="container"></div>
                </figure>
            </v-col>
            <v-col cols="3">
                <v-card class="mb-5" max-width="500" color="primaryContainer" subtitle="Temperature">
                    <v-card-item><span class="text-h3 text-onPrimaryContainer">{{ temperature }}</span></v-card-item>
                </v-card>
                <v-card class="mb-5" max-width="500" color="tertiaryContainer" subtitle="Heat Index (Feels like)">
                    <v-card-item><span class="text-h3 text-onTertiaryContainer">{{ heatindex }}</span></v-card-item>
                </v-card>
                <v-card class="mb-5" max-width="500" color="secondaryContainer" subtitle="Humidity">
                    <v-card-item><span class="text-h3 text-onSecondaryContainer">{{ humidity }}</span></v-card-item>
                </v-card>
            </v-col>
        </v-row>
        <v-row style="max-width: 1200px;" justify="start">
            <v-col cols="9">
                <figure class="highcharts-figure">
                    <div id="container1"></div>
                </figure>
            </v-col>
            <v-col cols="3">
                <v-card class="mb-5" max-width="500" color="primaryContainer" subtitle="Pressure">
                    <v-card-item><span class="text-h3 text-onPrimaryContainer">{{ pressure }}</span></v-card-item>
                </v-card>
                <v-card class="mb-5" max-width="500" color="tertiaryContainer" subtitle="Altitude">
                    <v-card-item><span class="text-h3 text-onTertiaryContainer">{{ altitude }}</span></v-card-item>
                </v-card>
                <v-card class="mb-5" max-width="500" color="secondaryContainer" subtitle="Soil Moisture">
                    <v-card-item><span class="text-h3 text-onSecondaryContainer">{{ moisture }}</span></v-card-item>
                </v-card>
            </v-col>
        </v-row>
        <v-row style="max-width: 1200px;" justify="start">
            <v-col cols="9">
                <figure class="highcharts-figure">
                    <div id="container2"></div>
                </figure>
            </v-col>
        </v-row>
        <v-row style="max-width: 1200px;" justify="start">
            <v-col cols="9">
                <figure class="highcharts-figure">
                    <div id="container3"></div>
                </figure>
            </v-col>
        </v-row>
        <v-row style="max-width: 1200px;" justify="start">
            <v-col cols="9">
                <figure class="highcharts-figure">
                    <div id="container4"></div>
                </figure>
            </v-col>
        </v-row>
    </v-container>
</template>

<script setup>
/** JAVASCRIPT HERE */

// IMPORTS
import { ref,reactive,watch ,onMounted,onBeforeUnmount,computed } from "vue";  
import { useRoute ,useRouter } from "vue-router";
import { useMqttStore } from '@/store/mqttStore'; // Import Mqtt Store
import { useAppStore } from "@/store/appStore";
import { storeToRefs } from "pinia";

// Highcharts, Load the exporting module and Initialize exporting module.
import Highcharts from 'highcharts';
import more from 'highcharts/highcharts-more';
import Exporting from 'highcharts/modules/exporting';
 
 
// VARIABLES
const router      = useRouter();
const route       = useRoute();
const Mqtt = useMqttStore();
const AppStore = useAppStore();
const { payload, payloadTopic } = storeToRefs(Mqtt);
const tempHiChart = ref(null); // Chart object
const humidityChart = ref(null); // Chart object
const pressureChart = ref(null); // Chart object
const altitudeChart = ref(null); // Chart object
const moistureChart = ref(null); // Chart object
const points = ref(10); // Specify the quantity of points to be shown on the live graph simultaneously.
const shift = ref(false); // Delete a point from the left side and append a new point to the right side of the graph.

const CreateCharts = async () => {

    // TEMPERATURE CHART
    tempHiChart.value = Highcharts.chart('container', {
        chart: { 
            zoomType: 'x',
            backgroundColor: '#1e1e1e'  // Dark background
        },
        title: { 
            text: 'Temperature Analysis (Live)',
            align: 'left',
            style: { color: '#FFFFFF' }  // White title text
        },
        yAxis: {
            title: { 
                text: '°C',
                style: { color: '#FFFFFF' } // White axis label
            },
            labels: { 
                format: '{value}°C',
                style: { color: '#FFFFFF' }  // White labels
            },
            gridLineColor: '#444444' // Dark gray grid lines
        },
        xAxis: {
            type: 'datetime',
            title: { 
                text: '',
                style: { color: '#FFFFFF' }  // White axis label
            },
            labels: { style: { color: '#FFFFFF' } },  // White text
            gridLineColor: '#444444' // Dark gray grid lines
        },
        tooltip: { 
            shared: true,
            backgroundColor: '#2b2b2b', // Dark tooltip background
            style: { color: '#FFFFFF' } // White tooltip text
        },
        legend: {
            itemStyle: { color: '#FFFFFF' } // White legend text
        },
        series: [
            {
                name: 'Temperature',
                type: 'spline',
                data: [],
                turboThreshold: 0,
                color: '#FF9999', // Muted pink (matches image)
                lineWidth: 2 // Slightly thicker line
            },
            {
                name: 'Heat Index',
                type: 'spline',
                data: [],
                turboThreshold: 0,
                color: '#FFC0CB', // Light pink (matches image)
                lineWidth: 2
            } 
        ],
    });

    // HUMIDITY CHART
    humidityChart.value = Highcharts.chart('container1', {
        chart: { 
            zoomType: 'x',
            backgroundColor: '#1e1e1e'  // Dark background
        },
        title: { 
            text: 'Humidity Analysis (Live)',
            align: 'left',
            style: { color: '#FFFFFF' }  // White title text
        },
        yAxis: {
            title: { 
                text: 'Humidity (%)',
                style: { color: '#FFFFFF' } // White axis label
            },
            labels: { 
                format: '{value}%',
                style: { color: '#FFFFFF' }  // White labels
            },
            gridLineColor: '#444444' // Dark gray grid lines
        },
        xAxis: {
            type: 'datetime',
            title: { 
                text: '',
                style: { color: '#FFFFFF' }  // White axis label
            },
            labels: { style: { color: '#FFFFFF' } },  // White text
            gridLineColor: '#444444' // Dark gray grid lines
        },
        tooltip: { 
            shared: true,
            backgroundColor: '#2b2b2b', // Dark tooltip background
            style: { color: '#FFFFFF' } // White tooltip text
        },
        legend: {
            itemStyle: { color: '#FFFFFF' } // White legend text
        },
        series: [
            {
                name: 'Humidity',
                type: 'spline',
                data: [],
                turboThreshold: 0,
                color: '#00BFFF', // Light blue color for humidity
                lineWidth: 2 // Slightly thicker line
            }
        ],
    });

    // PRESSURE CHART
    pressureChart.value = Highcharts.chart('container2', {
        chart: { 
            zoomType: 'x',
            backgroundColor: '#1e1e1e'  // Dark background
        },
        title: { 
            text: 'Pressure Analysis (Live)',
            align: 'left',
            style: { color: '#FFFFFF' }  // White title text
        },
        yAxis: {
            title: { 
                text: 'Pressure (Pa)',
                style: { color: '#FFFFFF' } // White axis label
            },
            labels: { 
                format: '{value}Pa',
                style: { color: '#FFFFFF' }  // White labels
            },
            gridLineColor: '#444444' // Dark gray grid lines
        },
        xAxis: {
            type: 'datetime',
            title: { 
                text: '',
                style: { color: '#FFFFFF' }  // White axis label
            },
            labels: { style: { color: '#FFFFFF' } },  // White text
            gridLineColor: '#444444' // Dark gray grid lines
        },
        tooltip: { 
            shared: true,
            backgroundColor: '#2b2b2b', // Dark tooltip background
            style: { color: '#FFFFFF' } // White tooltip text
        },
        legend: {
            itemStyle: { color: '#FFFFFF' } // White legend text
        },
        series: [
            {
                name: 'Pressure',
                type: 'spline',
                data: [],
                turboThreshold: 0,
                color: '#00BFFF', // Light blue color for humidity
                lineWidth: 2 // Slightly thicker line
            }
        ],
    });

    // ALTITUDE CHART
    altitudeChart.value = Highcharts.chart('container3', {
        chart: { 
            zoomType: 'x',
            backgroundColor: '#1e1e1e'  // Dark background
        },
        title: { 
            text: 'Altitude Analysis (Live)',
            align: 'left',
            style: { color: '#FFFFFF' }  // White title text
        },
        yAxis: {
            title: { 
                text: 'Altitude (m)',
                style: { color: '#FFFFFF' } // White axis label
            },
            labels: { 
                format: '{value}m',
                style: { color: '#FFFFFF' }  // White labels
            },
            gridLineColor: '#444444' // Dark gray grid lines
        },
        xAxis: {
            type: 'datetime',
            title: { 
                text: '',
                style: { color: '#FFFFFF' }  // White axis label
            },
            labels: { style: { color: '#FFFFFF' } },  // White text
            gridLineColor: '#444444' // Dark gray grid lines
        },
        tooltip: { 
            shared: true,
            backgroundColor: '#2b2b2b', // Dark tooltip background
            style: { color: '#FFFFFF' } // White tooltip text
        },
        legend: {
            itemStyle: { color: '#FFFFFF' } // White legend text
        },
        series: [
            {
                name: 'Altitude',
                type: 'spline',
                data: [],
                turboThreshold: 0,
                color: '#00BFFF', // Light blue color for humidity
                lineWidth: 2 // Slightly thicker line
            }
        ],
    });

    // MOISTURE CHART
    moistureChart.value = Highcharts.chart('container4', {
        chart: { 
            zoomType: 'x',
            backgroundColor: '#1e1e1e'  // Dark background
        },
        title: { 
            text: 'Soil Moisture Analysis (Live)',
            align: 'left',
            style: { color: '#FFFFFF' }  // White title text
        },
        yAxis: {
            title: { 
                text: 'Soil Moisture (%)',
                style: { color: '#FFFFFF' } // White axis label
            },
            labels: { 
                format: '{value}%',
                style: { color: '#FFFFFF' }  // White labels
            },
            gridLineColor: '#444444' // Dark gray grid lines
        },
        xAxis: {
            type: 'datetime',
            title: { 
                text: '',
                style: { color: '#FFFFFF' }  // White axis label
            },
            labels: { style: { color: '#FFFFFF' } },  // White text
            gridLineColor: '#444444' // Dark gray grid lines
        },
        tooltip: { 
            shared: true,
            backgroundColor: '#2b2b2b', // Dark tooltip background
            style: { color: '#FFFFFF' } // White tooltip text
        },
        legend: {
            itemStyle: { color: '#FFFFFF' } // White legend text
        },
        series: [
            {
                name: 'Soil Moisture',
                type: 'spline',
                data: [],
                turboThreshold: 0,
                color: '#00BFFF', // Light blue color for humidity
                lineWidth: 2 // Slightly thicker line
            }
        ],
    });

};

const updateLineCharts = async ()=>{
    if(!!start.value && !!end.value){
        // Convert output from Textfield components to 10 digit timestamps
        let startDate = new Date(start.value).getTime() / 1000;
        let endDate = new Date(end.value).getTime() / 1000;
        // Fetch data from backend
        const data = await AppStore.getAllInRange(startDate,endDate);
        // Create arrays for each plot
        let temperature = [];
        let heatindex = [];
        let humidity = [];
        let pressure = [];
        let altitude = [];
        let moisture = [];
        // Iterate through data variable and transform object to format recognized by highcharts
        data.forEach(row => {
            temperature.push({"x": row.timestamp * 1000, "y": parseFloat(row.temperature.toFixed(2)) });
            heatindex.push({ "x": row.timestamp * 1000,"y": parseFloat(row.heatindex.toFixed(2)) });
            humidity.push({ "x": row.timestamp * 1000,"y": parseFloat(row.humidity.toFixed(2)) });
            pressure.push({"x": row.timestamp * 1000, "y": parseFloat(row.pressure.toFixed(2)) });
            altitude.push({ "x": row.timestamp * 1000,"y": parseFloat(row.altitude.toFixed(2)) });
            moisture.push({ "x": row.timestamp * 1000,"y": parseFloat(row.moisture.toFixed(2)) });
        });
        // Add data to Temperature and Heat Index chart
        tempHiChart.value.series[0].setData(temperature);
        tempHiChart.value.series[1].setData(heatindex);
        humidityChart.value.series[0].setData(humidity);
        pressureChart.value.series[0].setData(pressure);
        altitudeChart.value.series[0].setData(altitude);
        moistureChart.value.series[0].setData(moisture);
    }
}

// COMPUTED PROPERTIES
const temperature = computed(()=>{
    if(!!payload.value){
        return `${payload.value.temperature.toFixed(2)} °C`;
    }
});

const heatindex = computed(()=>{
    if(!!payload.value){
        return `${payload.value.heatindex.toFixed(2)} °C`;
    }
});

const humidity = computed(()=>{
    if(!!payload.value){
        return `${payload.value.humidity.toFixed(2)} %`;
    }
});

const pressure = computed(()=>{
    if(!!payload.value){
        return `${(payload.value.pressure/1000).toFixed(2)} kPa`;
    }
});

const altitude = computed(()=>{
    if(!!payload.value){
        return `${payload.value.altitude.toFixed(2)} m`;
    }
});

const moisture = computed(()=>{
    if(!!payload.value){
        return `${payload.value.moisture.toFixed(2)} %`;
    }
});

// FUNCTIONS
onMounted(()=>{
    // THIS FUNCTION IS CALLED AFTER THIS COMPONENT HAS BEEN MOUNTED
    CreateCharts();

    Mqtt.connect(); // Connect to Broker located on the backend

    setTimeout( ()=>{
        // Subscribe to each topic
        // Mqtt.subscribe("topic1");
        Mqtt.subscribe("620162206");
    },3000);
});


onBeforeUnmount(()=>{
    // THIS FUNCTION IS CALLED RIGHT BEFORE THIS COMPONENT IS UNMOUNTED
    // unsubscribe from all topics
    Mqtt.unsubcribeAll();
});

//Watchers
watch(payload,(data)=> {
    if(points.value > 0){ points.value -- ; }
    else{ shift.value = true; }
    tempHiChart.value.series[0].addPoint({y:parseFloat(data.temperature.toFixed(2)) ,x: data.timestamp * 1000 },true, shift.value);
    tempHiChart.value.series[1].addPoint({y:parseFloat(data.heatindex.toFixed(2)) ,x: data.timestamp * 1000 },true, shift.value);
    humidityChart.value.series[0].addPoint({y:parseFloat(data.humidity.toFixed(2)) ,x: data.timestamp * 1000 },true, shift.value);
    pressureChart.value.series[0].addPoint({y:parseFloat(data.pressure.toFixed(2)) ,x: data.timestamp * 1000 },true, shift.value);
    altitudeChart.value.series[0].addPoint({y:parseFloat(data.altitude.toFixed(2)) ,x: data.timestamp * 1000 },true, shift.value);
    moistureChart.value.series[0].addPoint({y:parseFloat(data.moisture.toFixed(2)) ,x: data.timestamp * 1000 },true, shift.value);
})


</script>


<style scoped>
/** CSS STYLE HERE */
Figure {
border: 2px solid black;

}
</style>
  