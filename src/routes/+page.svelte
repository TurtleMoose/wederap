<script lang="ts">
    import { onMount } from "svelte";
    import { Input } from "$lib/components/ui/input/index.js";
    import { Switch } from "$lib/components/ui/switch";
    import { Button } from "$lib/components/ui/button/index.js";
    import { Slider } from "$lib/components/ui/slider/index.js";
    import { ArrowUp, ArrowDown } from 'lucide-svelte';
    import * as Card from "$lib/components/ui/card/index.js";
    import * as Chart from "$lib/components/ui/chart/index.js";
    import { scaleUtc, scaleBand } from "d3-scale";
    import { LineChart, BarChart, type ChartContextValue } from "layerchart";
    import { curveNatural } from "d3-shape";
    import { cubicInOut } from "svelte/easing";
    import { ModeWatcher, mode, toggleMode } from "mode-watcher";
    import Sun from "lucide-svelte/icons/sun";
    import Moon from "lucide-svelte/icons/moon";
    import { ScrollArea } from "$lib/components/ui/scroll-area/index.js";
    let activeMode = $mode;

    let celsius = $state(-273.15);
    let hours24 = $state(false);
    let celfeellike = $state(-273.15);
    let currenticon = "failed";
    let isCelsius: boolean = $state(false);
    let isDarkMode: boolean = $state(false);
    let weatherData: any = $state(null);
    let locationOption = $state("")
    let location = "Franklin, WI";
    let next24Hours = $state([]);
    
    const fetchWeatherData = async () => {
        const apiKey = "KG6B4U33X4ZYSJ7W2UGEGCNEZ";
        const url = `https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/${encodeURIComponent(location)}?unitGroup=metric&key=${apiKey}&contentType=json&iconSet=icons2`;

        try {
            const response = await fetch(url);
            if (!response.ok) {
                throw response;
            }
            weatherData = await response.json();
            celsius = weatherData?.currentConditions.temp;
            celfeellike = weatherData?.currentConditions.feelslike;
            currenticon = weatherData?.currentConditions.icon;
            console.log("Weather data:", weatherData);
        } catch (error) {
            console.error("Error:", error);
        }
        document.getElementById("currenticon").src = "icons/"+currenticon+".png";
        
    };

    async function getthestuffagain(){
        if(locationOption!=""){
        location = locationOption}
        let url = new URL(window.location.href);
        window.location.replace(url.origin+url.pathname+"?l="+location)
        // await fetchWeatherData();
    }
    let dayList = $state([])
    onMount(async () => {
        const urlParams = new URLSearchParams(window.location.search);
        if(urlParams.get("l")!=undefined){
            location = urlParams.get("l")
        }else{
            location = "Franklin, WI";
        }

        
        await fetchWeatherData();
        await setNext24();
if(activeMode=="dark"){
            let elems = document.querySelectorAll("img");
            for (var i = 0; i < elems.length; i++) {
                elems[i].style.filter= "invert(100%)";
            }
            isDarkMode = true;
        }
        
    });
async function setNext24(){
    let NOW = new Date().getHours();
        let tnow = []
        for(var i=NOW;i<24;i++){
            tnow.push(weatherData.days[0].hours[i]);
        }
        for(var i=0;i<NOW;i++){
            tnow.push(weatherData.days[1].hours[i]);
        }
        
        for(var i=0;i<24;i++){
            next24Hours[i]= {
                date: new Date(tnow[i].datetimeEpoch*1000), 
                hour: (hours24)?(i+NOW)%24+":00":(i+NOW)%24, 
                temp: round((isCelsius)?tnow[i].temp:(tnow[i].temp*1.8) +32, 0), 
                precip: tnow[i].precipprob,
                icon: tnow[i].icon,
                winddir: tnow[i].winddir
            };
        }

        for(var i=0;i<24;i++){
            dayList.push(next24Hours[i])//{ date: new Date(weatherData?.days[0].hours[i].datetimeEpoch*1000), hour: (hours24)?i+":00":"uh"+i, desktop: i, temp: round((isCelsius)?weatherData?.days[0].hours[i].temp:(weatherData?.days[0].hours[i].temp*1.8) +32, 0), precip: weatherData?.days[0].hours[i].precipprob })
        }
}

    function round(n, d) {
        return Math.floor(n * Math.pow(10, d) + 0.5) / Math.pow(10, d);
    }
    function toggleBG() {
        toggleMode();
        isDarkMode = !isDarkMode;
        // return;
        //deal with later
        if (isDarkMode) {
            let elems = document.querySelectorAll("img");
            for (var i = 0; i < elems.length; i++) {
                elems[i].style.filter= "invert(100%)";
            }
        } else {
            let elems = document.querySelectorAll("img");
            console.log(elems)
            for (var i = 0; i < elems.length; i++) {
                elems[i].style.filter= "invert(0%)";
            }
        }
    }

    const chartData = $state(dayList);

    const chartConfig = {
        views: { label: "Page Views", color: "" },
        desktop: { label: "Temperature", color: "black" },
    } satisfies Chart.ChartConfig;

    let activeChart = $state<keyof typeof chartConfig>("desktop");

    const total = $derived({
        desktop: chartData.reduce((acc, curr) => acc + curr.desktop, 0),
        mobile: chartData.reduce((acc, curr) => acc + curr.mobile, 0),
    });

    const activeSeries = $derived([
        {
        key: activeChart,
        label: chartConfig[activeChart].label,
        color: chartConfig[activeChart].color,
        },
    ]);

  const chartConfig2 = {
    desktop: { label: "Precipitation Chance", color: "var(--chart-1)" },
  } satisfies Chart.ChartConfig;
    let context = $state<ChartContextValue>();
</script>


<ModeWatcher />

<title>The wederap: all the weder all the tiem</title>
<div class="fixed h-full w-full z-[-100] light"></div>
<div class="flex gap-4 flex-row items-center">
    <div class="flex-1 mt-2 mb-0 pl-2">
        <Button onclick={toggleBG} variant="outline" size="icon">
  <Sun
    class="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0"
  />
  <Moon
    class="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100"
  />
  <span class="sr-only">Toggle theme</span>
</Button>
    </div>
    <h1 class="dark text-3xl font-bold flex-1 justify-center flex">the wederap</h1>
    <div class="flex-1">
        <div class="pt-2.5 flex-1 pr-4 flex justify-end font-bold" >
            F
            <Switch id="celsius" bind:checked={isCelsius}/>
            C
               
            12
            <Switch id="hours" bind:checked={hours24}/>
            24
        </div>
        <!-- <div class="pt-2.5 pl-3 flex-1">
            ☀️
            <Switch data-darkColor="" data-lightColor="" data-type="switch" id="dark" bind:checked={isDarkMode} onCheckedChange={toggleBG}/>
            🌙
        </div> -->
    </div>
</div>
<div class="flex flex-col md:flex-row gap-4 p-4">
    <div class="flex-1">
        <div class="flex">
            <Input placeholder="Location" class="w-[100%] h-10" bind:value={locationOption}/>
            <Button class="h-10 ml-3" data-darkColor="#DDDDDD" data-lightColor="#222222" onclick={getthestuffagain} data-type="bg">Go</Button>
        </div>
        <div class="flex">
            <div class="flex-1 text-center">{weatherData?.resolvedAddress}</div>
        </div>
        <div class="flex align-middle items-center">
            <!-- <div class="flex-1"></div> -->
            <img src="icons/failed.png" id="currenticon" class="h-10" data-type="img" alt="current-icon"/>
            <div class="flex-1"></div>
            <h2 data-darkColor="#DDDDDD" data-lightColor="#222222" data-type="text" id="currentTemp" class="text-[min(60vw,14rem)] sm:text-[min(60vw,10rem)] text-center font-bold">
                {round(isCelsius?celsius:celsius*1.8+32, isCelsius?1:0)}<span class="text-6xl">{isCelsius ? "C" : "F"}</span>
            </h2>
            <div class="flex-1"></div>
        </div>
        <div class="flex justify-center">
            <div class="text-3xl items-center"><span class="flex justify-center "><ArrowUp class="w-8 h-8 mt-0.75"/>{round(isCelsius?weatherData?.days[0].tempmax:weatherData?.days[0].tempmax*1.8+32, isCelsius?1:0)} <ArrowDown class="w-8 h-8 mt-0.75"/>{round(isCelsius?weatherData?.days[0].tempmin:weatherData?.days[0].tempmin*1.8+32, isCelsius?1:0)}</span></div>
            <div class="text-3xl ml-[10%] text-center">Feels Like {round(isCelsius?celfeellike:celfeellike*1.8+32, isCelsius?1:0,)}<span class="text-2xl">{isCelsius ? "C" : "F"}</span></div>
        </div>

        <div style="" class="w-[100%]">
            <Card.Root class="w-[100%]">
                <Card.Header class="flex flex-col items-stretch space-y-0 p-0 sm:flex-row">
              
                    <Card.Title class="flex flex-1 flex-col justify-center gap-0 px-6 py-0 sm:py-0">Next 24 Hours</Card.Title>
           
                </Card.Header>
                <Card.Content class="px-2 sm:p-6 w-[100%]">
                    <!-- TODO: what the hell, fix this -->
                    <ScrollArea class="w-142 rounded-md border p-5 whitespace-nowrap" orientation="horizontal">
                        <div class="flex w-max space-x-4 m-1 p-1">
                            {#each next24Hours as hour}
                                <div class="">
                                    <img src={"icons/"+hour.icon+".png"} alt="I" class="h-10 select-none"/>
                                    <img src={"wind/wind-"+Math.floor((hour.winddir+23)/45)%8+".png"} alt="W" class="h-10 mt-3 select-none"/>
                                </div>
                            {/each}
                     </div>
                    <br>
                    <Chart.Container config={chartConfig} class="aspect-auto h-[75px] mr-7.5 ml-7.5">
                    <LineChart
                        data={chartData}
                        x="date"
                        xScale={scaleUtc()}
                        axis="x"
                        series={[{key: "temp",label: chartConfig2[activeChart].label,color: chartConfig2[activeChart].color}]}
                        props={{
                        spline: { curve: curveNatural, motion: "tween", strokeWidth: 2 },
                        xAxis: {
                            format: (v: Date) => {
                                let ret="";
                                if(hours24){ret = v.getHours();ret+=":00"}else{ret = ((v.getHours()==0||v.getHours()==12)?12:v.getHours()%12).toString(); ret+=(v.getHours()>=12)?"PM":"AM"}
                            return ret;// ((v.getHours()==0||v.getHours()==12)?12:v.getHours()%12).toString()
                            },
                        },
                        highlight: { points: { r: 4 } },
                        }}
                    >
                        {#snippet tooltip()}
                        <Chart.Tooltip hideLabel />
                        {/snippet}
                    </LineChart>
                    </Chart.Container>
                    <br>
                    <Chart.Container config={chartConfig2} class="aspect-auto h-[75px] w-full">
                    <BarChart
                        bind:context
                        data={chartData}
                        xScale={scaleBand().padding(0)}
                        x="date"
                        axis="x"
                        series={[{ key: "precip", label: "Precip. Chance", color: chartConfig2.desktop.color }]}
                        props={{
                        bars: {
                            stroke: "none",
                            rounded: "all",
                            radius: 8,
                            // use the height of the chart to animate the bars
                            initialY: context?.height,
                            initialHeight: 0,
                            motion: {
                            x: { type: "tween", duration: 500, easing: cubicInOut },
                            width: { type: "tween", duration: 500, easing: cubicInOut },
                            height: { type: "tween", duration: 500, easing: cubicInOut },
                            y: { type: "tween", duration: 500, easing: cubicInOut },
                            },
                        },
                        highlight: { area: { fill: "none" } },
                        xAxis: { format: (v: Date) => {
                            let ret=""; 
                            if(hours24){ret = v.getHours().toString();ret+=":00";}else{ret = ((v.getHours()==0||v.getHours()==12)?12:v.getHours()%12).toString(); ret+=(v.getHours()>=12)?"PM":"AM"; } return ret;},
                        }}}
                    >
                        {#snippet tooltip()}
                        <Chart.Tooltip hideLabel />
                        {/snippet}
                    </BarChart>
                    </Chart.Container>
                       
                    </ScrollArea>
                </Card.Content>
                </Card.Root>
                
        </div>
    </div>
    <div class="flex-1">
        <Card.Root class="md:h-[91vh]">
            <Card.Header>
                <Card.Title>a</Card.Title>
                <Card.Description>b</Card.Description>
            </Card.Header>
            <Card.Content>
                <p>c</p>
            </Card.Content>
            <Card.Footer>
                <p>d</p>
            </Card.Footer>
        </Card.Root>
    </div>
    <div class="flex-1">
        <Card.Root class="md:h-[91vh] wrap-break-words overflow-y-scroll ">
            <Card.Header>
                <Card.Title><div class="max-w-xs text-2xl font-bold">WEATHER ALERTS</div></Card.Title>
                <Card.Description>For {weatherData?.resolvedAddress}</Card.Description>
            </Card.Header>
            <Card.Content>
            {#each weatherData?.alerts as alert}
                <Card.Root>
                    <Card.Header>
                        <Card.Title>{alert.event}</Card.Title>
                        <Card.Description>{alert.headline}</Card.Description
                        >
                    </Card.Header>
                    <Card.Content>
                        <p>{alert.description}</p>
                    </Card.Content>
                    <Card.Footer>
                        <p>Ends: {new Date(alert.endsEpoch * 1000)}</p>
                    </Card.Footer>
                </Card.Root>
            {/each}
            <div class="max-w-xs font-bold" >
                {weatherData?.alerts.length == 0 ? "No Alerts :)" : ""}
            </div>
            </Card.Content>
        </Card.Root>
    </div>
</div>