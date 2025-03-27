<template>
    <v-container fluid class="bg-surface">
      <!-- First Row -->
      <v-row class="p-1" style="max-width: 1200px;">
        <!-- Column 1 -->
        <v-col>
          <v-sheet class="p-2" height="250">
            <p>Enter date range for Analysis</p>
            <v-divider></v-divider>
  
            <v-text-field
              class="m-5"
              label="Start date" 
              type="date"
              density="compact"
              variant="solo-inverted"
              flat
              style="max-width: 300px;"
              v-model="start"
            ></v-text-field>
  
            <v-text-field
              label="End date" 
              type="date"
              density="compact"
              variant="solo-inverted"
              flat
              style="max-width: 300px;"
              v-model="end"
            ></v-text-field>
  
            <v-spacer></v-spacer>
  
            <v-btn class="text-caption" text="Analyze" color="primary" variant="tonal" @click="updateLineCharts();updateCards();">
            </v-btn>
          </v-sheet>
        </v-col>
  
        <!-- Column 2 (Temperature Card) -->
        <v-col cols="3" align="center">
          <v-card title="Temperature" width="250" variant="outlined" color="primary" density="compact" rounded="lg">
            <v-card-item class="mb-n5">
              <v-chip-group class="d-flex flex-row justify-center" color="primaryContainer" variant="flat">
                <v-tooltip text="Min" location="start">
                  <template v-slot:activator="{ on, attrs }">
                    <v-chip v-bind="attrs" v-on="on">{{ temperature.min }}</v-chip>
                  </template>
                </v-tooltip>
                <v-tooltip text="Range" location="top">
                  <template v-slot:activator="{ on, attrs }">
                    <v-chip v-bind="attrs" v-on="on">{{ temperature.range }}</v-chip>
                  </template>
                </v-tooltip>
                <v-tooltip text="Max" location="end">
                  <template v-slot:activator="{ on, attrs }">
                    <v-chip v-bind="attrs" v-on="on">{{ temperature.max }}</v-chip>
                  </template>
                </v-tooltip>
              </v-chip-group>
            </v-card-item>
            <v-card-item align="center">
              <span class="text-h1 text-primary font-weight-bold">{{ temperature.avg }}</span>
            </v-card-item>
          </v-card>
        </v-col>
  
        <!-- Column 3 (Humidity Card) -->
        <v-col cols="3" align="center">
          <v-card title="Humidity" width="250" variant="outlined" color="primary" density="compact" rounded="lg">
            <v-card-item class="mb-n5">
              <v-chip-group class="d-flex flex-row justify-center" color="primaryContainer" variant="flat">
                <v-tooltip text="Min" location="start">
                  <template v-slot:activator="{ on, attrs }">
                    <v-chip v-bind="attrs" v-on="on">{{ humidity.min }}</v-chip>
                  </template>
                </v-tooltip>
                <v-tooltip text="Range" location="top">
                  <template v-slot:activator="{ on, attrs }">
                    <v-chip v-bind="attrs" v-on="on">{{ humidity.range }}</v-chip>
                  </template>
                </v-tooltip>
                <v-tooltip text="Max" location="end">
                  <template v-slot:activator="{ on, attrs }">
                    <v-chip v-bind="attrs" v-on="on">{{ humidity.max }}</v-chip>
                  </template>
                </v-tooltip>
              </v-chip-group>
            </v-card-item>
            <v-card-item align="center">
              <span class="text-h1 text-primary font-weight-bold">{{ humidity.avg }}</span>
            </v-card-item>
          </v-card>
        </v-col>
      </v-row>
  
      <!-- Second Row (Charts) -->
      <v-row style="max-width: 1200px;">
        <v-col cols="12">
          <figure class="highcharts-figure">
            <div id="container"></div>
          </figure>
        </v-col>
        <v-col cols="12">
          <figure class="highcharts-figure">
            <div id="container0"></div>
          </figure>
        </v-col>
      </v-row>
  
      <!-- Third Row (Charts) -->
      <v-row style="max-width: 1200px;">
        <v-col class="border" cols="12">
          <figure class="highcharts-figure">
            <div id="container1"></div>
          </figure>
        </v-col>
        <v-col cols="12">
          <figure class="highcharts-figure">
            <div id="container2"></div>
          </figure>
        </v-col>
        <v-col cols="12">
          <figure class="highcharts-figure">
            <div id="container3"></div>
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
const humidChart = ref(null); // Chart object
const pressChart = ref(null); // Chart object
const altChart = ref(null); // Chart object
const moistChart = ref(null); // Chart object
const points = ref(10); // Specify the quantity of points to be shown on the live graph simultaneously.
const shift = ref(false); // Delete a point from the left side and append a new point to the right side of the graph.
const start   = ref("");
const end    = ref("");
const temperature = reactive({"min":0,"max":0,"avg":0,"range":0});
const humidity = reactive({"min":0,"max":0,"avg":0,"range":0});

const CreateCharts = async () => {

    // TEMPERATURE CHART
    tempHiChart.value = Highcharts.chart('container', {
        chart: { zoomType: 'x' },
        title: { text: 'Air Temperature and Heat Index Analysis', align: 'left' },
        subtitle: { text: 'The heat index, also known as the "apparent temperature," is a measure that combines air temperature and relative humidity to assess how hot it feels to the human body. The relationship between heat index and air temperature is influenced by humidity levels. As humidity increases, the heat index also rises, making the perceived temperature higher than the actual air temperature.' },
        yAxis: {
            title: { text: 'Air Temperature & Heat Index' , style:{color:'#000000'}},
            labels: { format: '{value} °C' }
        },
        xAxis: {
            type: 'datetime',
            title: { text: 'Time', style:{color:'#000000'} },
        },
        tooltip: { shared:true, 
            pointFormat: 'Humidity: {point.x} % <br/> Temperature: {point.y} °C'
        },
        series: [
            {
                name: 'Temperature',
                type: 'spline',
                data: [],
                turboThreshold: 0,
                color: Highcharts.getOptions().colors[0]
            },
            {
                name: 'Heat Index',
                type: 'spline',
                data: [],
                turboThreshold: 0,
                color: Highcharts.getOptions().colors[1]
            } 
        ],
    });

    humidChart.value = Highcharts.chart('container0', {
        chart: { zoomType: 'x' },
        title: { text: 'Humidity Analysis', align: 'left' },
        subtitle: { text: 'This chart shows the humidity levels over time, which impacts the perceived temperature.' },
        yAxis: {
            title: { text: 'Humidity', style: { color: '#000000' } },
            labels: { format: '{value} %' }
        },
        xAxis: {
            type: 'datetime',
            title: { text: 'Time', style: { color: '#000000' } },
        },
        tooltip: {
            shared: true,
            pointFormat: 'Humidity: {point.x} % <br/> Temperature: {point.y} °C'
        },
        series: [
            {
                name: 'Humidity',
                type: 'spline',
                data: [], // Replace with actual dynamic data
                turboThreshold: 0,
                color: Highcharts.getOptions().colors[0]
            }
        ],
    });

    pressChart.value = Highcharts.chart('container1', {
        chart: { zoomType: 'x' },
        title: { text: 'Pressure Analysis', align: 'left' },
        subtitle: { text: 'This chart shows the pressure levels over time.' },
        yAxis: {
            title: { text: 'Pressure', style: { color: '#000000' } },
            labels: { format: '{value} Pa' }
        },
        xAxis: {
            type: 'datetime',
            title: { text: 'Time', style: { color: '#000000' } },
        },
        tooltip: {
            shared: true,
            pointFormat: 'Pressure: {point.x} Pa <br/> Temperature: {point.y} °C'
        },
        series: [
            {
                name: 'Pressure',
                type: 'spline',
                data: [], // Replace with actual dynamic data
                turboThreshold: 0,
                color: Highcharts.getOptions().colors[0]
            }
        ],
    });

    altChart.value = Highcharts.chart('container2', {
        chart: { zoomType: 'x' },
        title: { text: 'Altittude Analysis', align: 'left' },
        subtitle: { text: 'This chart shows the altitude levels over time.' },
        yAxis: {
            title: { text: 'Altitude', style: { color: '#000000' } },
            labels: { format: '{value} m' }
        },
        xAxis: {
            type: 'datetime',
            title: { text: 'Time', style: { color: '#000000' } },
        },
        tooltip: {
            shared: true,
            pointFormat: 'Altitude: {point.x} % <br/> Temperature: {point.y} °C'
        },
        series: [
            {
                name: 'Altitude',
                type: 'spline',
                data: [], // Replace with actual dynamic data
                turboThreshold: 0,
                color: Highcharts.getOptions().colors[0]
            }
        ],
    });

    moistChart.value = Highcharts.chart('container3', {
        chart: { zoomType: 'x' },
        title: { text: 'Moisture Analysis', align: 'left' },
        subtitle: { text: 'This chart shows the moisture levels over time.' },
        yAxis: {
            title: { text: 'Moisture', style: { color: '#000000' } },
            labels: { format: '{value} %' }
        },
        xAxis: {
            type: 'datetime',
            title: { text: 'Time', style: { color: '#000000' } },
        },
        tooltip: {
            shared: true,
            pointFormat: 'Moisture: {point.x} % <br/> Temperature: {point.y} °C'
        },
        series: [
            {
                name: 'Moisture',
                type: 'spline',
                data: [], // Replace with actual dynamic data
                turboThreshold: 0,
                color: Highcharts.getOptions().colors[0]
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
        humidChart.value.series[0].setData(humidity);
        pressChart.value.series[0].setData(pressure);
        altChart.value.series[0].setData(altitude);
        moistChart.value.series[0].setData(moisture);
    }
}

const updateCards = async () => {
    // Retrieve Min, Max, Avg, Spread/Range
    if(!!start.value && !!end.value){
        // 1. Convert start and end dates collected fron TextFields to 10 digit timestamps
        let startDate = new Date(start.value).getTime() / 1000;
        let endDate = new Date(end.value).getTime() / 1000;
        // 2. Fetch data from backend by calling the API functions
        const temp = await AppStore.getTemperatureMMAR(startDate,endDate);
        const humid = await AppStore.getHumidityMMAR(startDate,endDate);

        temperature.max = temp[0].max.toFixed(1);
        //3. complete for min, avg and range
        temperature.min = temp[0].min.toFixed(1);
        temperature.avg = temp[0].avg.toFixed(1);
        temperature.range = temp[0].range.toFixed(1);
        //4. complete max, min, avg and range for the humidity variable
        humidity.max = humid[0].max.toFixed(1);
        //3. complete for min, avg and range
        humidity.min = humid[0].min.toFixed(1);
        humidity.avg = humid[0].avg.toFixed(1);
        humidity.range = humid[0].range.toFixed(1);
    }
}

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

/*watch(payload,(data)=> {
    if(points.value > 0){ points.value -- ; }
    else{ shift.value = true; }
    tempHiChart.value.series[0].addPoint({y:parseFloat(data.temperature.toFixed(2)) ,x: data.timestamp * 1000 },true, shift.value);
    tempHiChart.value.series[1].addPoint({y:parseFloat(data.heatindex.toFixed(2)) ,x: data.timestamp * 1000 },true, shift.value);
    humidChart.value.series[0].addPoint({y:parseFloat(data.humidity.toFixed(2)) ,x: data.timestamp * 1000 },true, shift.value);
})*/


</script>


<style scoped>
/** CSS STYLE HERE */

</style>
