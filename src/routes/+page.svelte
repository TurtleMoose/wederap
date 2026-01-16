<script lang="ts">
    import { onMount } from "svelte";
    import { Input } from "$lib/components/ui/input/index.js";
    import { Switch } from "$lib/components/ui/switch";
    import { Button } from "$lib/components/ui/button/index.js";
    import { Slider } from "$lib/components/ui/slider/index.js";
    import { ArrowUp, ArrowDown } from 'lucide-svelte';
    import * as Card from "$lib/components/ui/card/index.js";
    import * as Chart from "$lib/components/ui/chart/index.js";
    import { scaleUtc } from "d3-scale";
    import { LineChart } from "layerchart";
    import { curveNatural } from "d3-shape";

    let celsius = $state(-273.15);
    let hours24 = $state(false);
    let celfeellike = $state(-273.15);
    let currenticon = "failed";
    let isCelsius: boolean = $state(false);
    let isDarkMode: boolean = $state(true);
    let weatherData: any = $state(null);
    let locationOption = $state("")
    let location = "Franklin, WI";
    const fetchWeatherData = async () => {
        const apiKey = "TTZ6BN7P2NTXWS6L9RDH96SWC";
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
        if(locationOption!="")
        location = locationOption
        await fetchWeatherData();
    }
    let dayList = $state([])
    onMount(async () => {
        toggleBG();
        await fetchWeatherData();
        //weatherData?.days[0].hours[i]
        
        for(var i=0;i<24;i++){
            dayList.push({ date: new Date(weatherData?.days[0].hours[i].datetimeEpoch*1000), desktop: 222*i, mobile: 150 })
            console.log(weatherData?.days[0].hours[i].datetimeEpoch)
            console.log(dayList[i])
        }
    });
    function round(n, d) {
        return Math.floor(n * Math.pow(10, d) + 0.5) / Math.pow(10, d);
    }
    function toggleBG() {
        const lightColor = "#222222"
        const darkColor = "#DDDDDD"
        return;
        //deal with later
        if (isDarkMode) {
            let elems = document.querySelectorAll("[data-type]");
            for (var i = 0; i < elems.length; i++) {
                if (elems[i].getAttribute("data-type") == "bg") {
                    elems[i].style.backgroundColor = elems[i].getAttribute("data-darkColor");
                } else if (elems[i].getAttribute("data-type") == "text") {
                    elems[i].style.color = elems[i].getAttribute("data-darkColor");
                } else if (elems[i].getAttribute("data-type") == "img") {
                    elems[i].style.filter = "invert(1)";
                }
            }
        } else {
            let elems = document.querySelectorAll("[data-type]");
            for (var i = 0; i < elems.length; i++) {
                if (elems[i].getAttribute("data-type")?.includes("bg")) {
                    elems[i].style.backgroundColor = elems[i].getAttribute("data-lightColor");
                } else if (elems[i].getAttribute("data-type")?.includes("text")) {
                    elems[i].style.color = elems[i].getAttribute("data-lightColor");
                } else if (elems[i].getAttribute("data-type")?.includes("img")) {
                    elems[i].style.filter = "invert(0)";
                }
            }
        }
    }

    const chartData = dayList;

    const chartConfig = {
        views: { label: "Page Views", color: "" },
        desktop: { label: "Desktop", color: "var(--chart-1)" },
        mobile: { label: "Mobile", color: "var(--chart-2)" },
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

</script>

<title>The wederap: all the weder all the tiem</title>
<div data-darkColor="#222222" data-lightColor="#DDDDDD" data-type="bg" style="left:0;top:0;width:100%;height:100%;position:fixed;z-index:-100;background-color:white;"></div>
<div class="flex gap-4 flex-row items-center">
    <div class="flex-1 "></div>
    <h1 data-darkColor="#DDDDDD" data-lightColor="#222222" data-type="text" class="text-3xl font-bold flex-1 justify-center flex">the wederap</h1>
    <div class="flex-1">
        <div class="pt-2.5 flex-1 pr-4 flex justify-end font-bold" data-darkColor="#DDDDDD" data-lightColor="#222222" data-type="text">
            F
            <Switch data-darkColor="" data-lightColor="" data-type="switch" id="celsius" bind:checked={isCelsius}/>
            C
               
            12
            <Switch data-darkColor="" data-lightColor="" data-type="switch" id="hours" bind:checked={hours24}/>
            24
        </div>
        <!-- <div class="pt-2.5 pl-3 flex-1">
            ☀️
            <Switch data-darkColor="" data-lightColor="" data-type="switch" id="dark" bind:checked={isDarkMode} onCheckedChange={toggleBG}/>
            🌙
        </div> -->
    </div>
</div>
<div class="flex gap-4 p-4 pt-2">
    <div class="flex-1">
        <div class="flex">
            <Input placeholder="Location" class="w-[100%] h-10" bind:value={locationOption}/>
            <Button class="h-10 ml-3" data-darkColor="#DDDDDD" data-lightColor="#222222" onclick={getthestuffagain} data-type="bg">Go</Button>
        </div>
        <div class="flex">
            <div class="flex-1 text-center">{weatherData?.resolvedAddress}</div>
        </div>
        <div class="flex">
            <div class="flex-1"></div>
            <img src="icons/failed.png" id="currenticon" class="w-60" data-type="img" alt="current-icon"/>
            <div class="flex-1"></div>
            <h2 data-darkColor="#DDDDDD" data-lightColor="#222222" data-type="text" id="currentTemp" class="text-[230px] font-bold">
                {round(isCelsius?celsius:celsius*1.8+32, isCelsius?1:0)}<span class="text-6xl">{isCelsius ? "C" : "F"}</span>
            </h2>
            <div class="flex-1"></div>
        </div>
        <div class="flex justify-center">
            <div class="text-3xl items-center"><span class="flex justify-center "><ArrowUp class="w-8 h-8 mt-0.75"/>{round(isCelsius?weatherData?.days[0].tempmax:weatherData?.days[0].tempmax*1.8+32, isCelsius?1:0)} <ArrowDown class="w-8 h-8 mt-0.75"/>{round(isCelsius?weatherData?.days[0].tempmin:weatherData?.days[0].tempmin*1.8+32, isCelsius?1:0)}</span></div>
            <div class="text-3xl ml-[10%] text-center">Feels Like {round(isCelsius?celfeellike:celfeellike*1.8+32, isCelsius?1:0,)}<span class="text-2xl">{isCelsius ? "C" : "F"}</span></div>
        </div>

        <div style="" class="">
            <!-- <Card.Root>
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
            </Card.Root> -->
            <Card.Root>
                <Card.Header class="flex flex-col items-stretch space-y-0 p-0 sm:flex-row">
                    <div class="flex flex-1 flex-col justify-center gap-0 px-6 py-0 sm:py-0">
                    <Card.Title>Line Chart - Interactive</Card.Title>
                    <Card.Description>Showing total visitors for the last 3 months</Card.Description>
                    </div>
                </Card.Header>
                <Card.Content class="px-2 sm:p-6">
                    <Chart.Container config={chartConfig} class="aspect-auto h-[250px] w-full">
                    <LineChart
                        data={chartData}
                        x="date"
                        xScale={scaleUtc()}
                        axis="x"
                        series={activeSeries}
                        props={{
                        spline: { curve: curveNatural, motion: "tween", strokeWidth: 2 },
                        xAxis: {
                            format: (v: Date) => {
                                let ret="";
                                if(hours24){ret = v.getHours();ret+=":00"}else{ret = ((v.getHours()==0||v.getHours()==12)?12:v.getHours()%12).toString(); ret+=(v.getHours()>=12)?"PM":"AM"}
                            return ret// ((v.getHours()==0||v.getHours()==12)?12:v.getHours()%12).toString()
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
                </Card.Content>
                </Card.Root>
        </div>
    </div>
    <div class="flex-1">
        <Card.Root class="h-[91vh]">
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
        <Card.Root class="h-[91vh] overflow-scroll">
            <Card.Header>
                <Card.Title><div class="max-w-xs text-2xl font-bold" data-darkColor="#DDDDDD" data-lightColor="#222222" data-type="text">WEATHER ALERTS</div></Card.Title>
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
            <div class="max-w-xs font-bold" data-darkColor="#DDDDDD" data-lightColor="#222222" data-type="text">
                {weatherData?.alerts.length == 0 ? "No Alerts :)" : ""}
            </div>
            </Card.Content>
        </Card.Root>
    </div>
</div>

<style>
    /* img{
        filter: invert(1);
    } */
</style>
