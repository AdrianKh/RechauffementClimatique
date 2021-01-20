
<style>
    .center-div {
         margin: 0 auto;
         width: 1000px; 
     }
    
    .box {
        display: grid;
        grid-template-columns: 1fr 1fr 1fr;
        grid-gap: 10px 2em;
    }

    .box :first-child {
        align-self: center;
    }

   
 </style>

<script>
    import { onMount } from "svelte";
    var tooltip;
    var loadedData = d3.json("/data/data_anomalies.json");
    var geoFrance = d3.json("/data/france.json");

    var width = 200;
    var height = 200;
    const visus = ['#RCP26', '#RCP45', '#RCP85']

    const projection = d3
        .geoConicConformal()
        .center([2.454071, 45.279229])
        .scale(1000)
        .translate([width / 2, height / 2]);
    
    var path = d3.geoPath().projection(projection);
    var rcp;
    var month;
    var colorList = [
        "#ffffee",
        "#ffeda0",
        "#fed976",
        "#feb24c",
        "#fd8d3c",
        "#fc4e2a",
        "#e31a1c",
        "#bd0026",
        "#800026",
    ]; //.reverse();
    var list_mois = [
        "Janvier",
        "Février",
        "Mars",
        "Avril",
        "Mai",
        "Juin",
        "Juillet",
        "Août",
        "Septembre",
        "Octobre",
        "Novembre",
        "Décembre",
    ];
    
    var color = d3.scaleQuantize().domain([0, 6]).range(colorList);
    var tab_svg;


    function updateViewMonth(event){
        month = event.target.value;
        draw3France();
    }
    

    function drawFrance(rcp, svg) {
        rcp = rcp.substring(1,5)+"."+rcp.substring(5)
        console.log(rcp)
        loadedData.then((data) => {
            var pointess = data.filter((d) => {
                return d.Contexte === rcp && d.Mois == month;
            });
            
            var pointes = pointess;
            console.log(pointes.length);
            
            var carte = svg.selectAll("rect")
                           .data(pointes)


            carte.enter()
                .append("rect")
                .attr("class", "enter")
                .attr("x", (data) => {
                    return projection([data.Longitude, data.Latitude])[0];
                })
                .attr("y", (data) => {
                    return projection([data.Longitude, data.Latitude])[1] + 50;
                })
                .attr("width", 3)
                .attr("height", 3)
                .attr("fill", (d) => {
                    return color(d.Moy);
                })
                carte
                .attr("class", "update")
                .attr("x", (data) => {
                    return projection([data.Longitude, data.Latitude])[0];
                })
                .attr("y", (data) => {
                    return projection([data.Longitude, data.Latitude])[1] + 50;
                })
                .attr("width", 3)
                .attr("height", 3)
                .attr("fill", (d) => {
                    return color(d.Moy);
                })

                svg.append("text")
                    .attr("class","enter")
                     .attr("x",  0)
                     .attr("y", 40 )
                     .text(rcp)
        });
    }
    
    const svg_init= () => {
        visus.forEach(visu => {
            
            let svg = d3
                .select(visu)
                .append("svg")
                .attr("id", "map")
                .attr("width", width)
                .attr("height", height);
            tab_svg[visu] = svg 
        })  
    }

    const draw3France = () => {
        visus.forEach(scenario => {
            drawFrance(scenario, tab_svg[scenario])
        })
    }
    
    onMount(() => {
        tab_svg = []
        rcp = "RCP2.6";
        month = 12;
        svg_init(visus)
        draw3France();
    });
</script>
<h1>Projections sur les anomalies de températures en France selon différents scénarios</h1>

<div class="box">
    <div id="RCP26"/>
    <div id="RCP45"/>
    <div id="RCP85"/>
</div>
<br><br>
<svg width="192" height="24" >
    <rect fill="rgb(255,255,204)" width="24" height="24" x="0"></rect>
    <text font-weight="bold" y="20" x="4">0 °C</text><rect fill="rgb(255,237,160)" width="24" height="24" x="24"></rect>
    <rect fill="rgb(254,217,118)" width="24" height="24" x="48"></rect><rect fill="rgb(254,178,76)" width="24" height="24" x="72"></rect>
    <rect fill="rgb(253,141,60)" width="24" height="24" x="96"></rect><rect fill="rgb(252,78,42)" width="24" height="24" x="120"></rect>
    <rect fill="rgb(227,26,28)" width="24" height="24" x="144"></rect><rect fill="rgb(177,0,38)" width="24" height="24" x="168"></rect>
    <text font-weight="bold" y="20" x="158" fill="white">+6 °C</text>
</svg> 
<br>




<input
    class="slider"
    id="slider2"
    type="range"
    min="1"
    max="12"
    step="1"
    on:input={updateViewMonth} />

<p>Mois : {list_mois[parseInt(month) - 1]}</p>

<div style="margin: 2%">
    Source des données: <a href="https://www.data.gouv.fr/fr/organizations/meteo-france/">Météo France - data.gouv.fr</a> 
</div>



<a href='#Metropoles' class="btn btn-primary btn-lg active" role="button" aria-pressed="true">Precedent</a>
<a href='#Cercles' class="btn btn-primary btn-lg active" role="button" aria-pressed="true">Suivant</a>
