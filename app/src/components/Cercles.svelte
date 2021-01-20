<style>
    svg {
      margin-left: 50px
    }

    p {
        text-indent: 30px;
    }

    #svg {
      margin: 30px
    }

   .center-div {
        margin: 0 auto;
        width: 1000px; 
    }

    .footer {
        position: fixed;
        left: 0;
        bottom: 0;
        height:10%;
        width: 100%;
        background: rgba(0, 0, 0, 0.5);
        color: white;
        text-align: center;
    }
    
</style>
<script>
    import { onMount } from "svelte";
    var width = 1250;
    var height = 140;
    var tab_svg;
    var maxRayon = 30;
    var minRayon = 18;
    var ecartCircle = 40;
    let maxtemp = 3;
    let mintemp = 0;
    var scenarios = ['#RCP26', '#RCP45', '#RCP60', '#RCP85']

    var titles = {"#RCP26": "Faible emission de gaz à effet de serre", "#RCP45": "Emission moyenne faible de gaz à effet de serre", "#RCP60": "Emission moyenne forte de gaz à effet de serre", "#RCP85": "Forte emission de gaz à effet de serre"}
    var scale = d3
        .scaleLinear()
        .domain([mintemp, maxtemp])
        .range([minRayon, maxRayon]);
    var colorList = [
        "#d7191c",
        "#fdae61",
        "#ffffbf",
        "#abd9e9",
        "#2c7bb6",
    ].reverse();
    var mois = [
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
    var color = d3
        .scaleQuantize()
        .domain([mintemp, maxtemp])
        .range(colorList.slice(1));
    var rcp = 26;
    var year = "2020_2039";

    function GeneratePositions(d, rayon, ecart, pos) {
        // console.log(data);
        var data = d;
        var posX = rayon + ecart;
        var posY = pos;
        for (let i = 0; i < 12; i++) {
            data[i].x = posX;
            data[i].y = posY;
            data[i].mois = mois[i];
            posX = posX + ecart + 2 * rayon;
        }
        return data;
    }

    function csvToJson(data) {
        var json = [];
        for (let i = 0; i < 12; i++) {
            json[i] = { temp: d3[i]["Monthly Temperature - (Celsius)"] };
        }
    }

    function createCircles(data, rayon, ecart, pos, svg, title, drawMonth = false) {
        let fd = data.slice(24, 36);
        console.log(svg);
        var firstData = GeneratePositions(fd, rayon, ecart, pos);
        var circles = svg.selectAll("circle").data(firstData);

        circles
            .attr("class", "update")
            .attr("r", function (d) {
                return scale(d["Monthly Temperature - (Celsius)"]);
            })
            .attr("cx", (d) => {
                return d["x"];
            })
            .attr("cy", (d) => {
                return d["y"];
            })
            .attr("fill", (d) => {
                return color(d["Monthly Temperature - (Celsius)"]);
            })
            .attr("class", (d) => {
                return d[" Statistics"];
            })

        circles
            .enter()
            .append("circle")
            .attr("class", "enter")
            .attr("r", function (d) {
                return scale(d["Monthly Temperature - (Celsius)"]);
            })
            .attr("cx", (d) => {
                return d["x"];
            })
            .attr("cy", (d) => {
                return d["y"];
            })
            .attr("fill", (d) => {
                return color(d["Monthly Temperature - (Celsius)"]);
            })
            .attr("class", (d) => {
                return d[" Statistics"];
            })
            .attr("stroke","black")

        
        if (drawMonth){
        svg.selectAll("text")
            .data(firstData)
            .enter()
            .append("text")
            .attr("x", (d) => {
                return d["x"] - maxRayon;
            })
            .attr("y", (d) => {
                return d["y"] + maxRayon + 40;
            })
            .text((d) => {
                return d["mois"];
            });
            
        }
            svg.append("text")
           .attr("x", 0)
           .attr("y", 20)
           .text(titles[title])
    }

    function iterateScenario(rcp){
        scenarios.forEach((vis)=>{
            d3.csv("/data/tas_"+year+"_manom_rcp"+vis.slice(4,6)+"_FRA.csv").then((data) => {
                if(vis != "#RCP85")
                    createCircles(data, maxRayon, ecartCircle, 60,tab_svg[vis], vis);
                else createCircles(data, maxRayon, ecartCircle, 60,tab_svg[vis], vis, true);
            });
        })
    }

    function changeRcp(event) {
        rcp = event.target.value;
        iterateScenario(rcp)
    }

    function changeYear(event) {
        year = event.target.value;
        iterateScenario(rcp)
    }

    function initSvg(){
        scenarios.forEach((scenario) =>{
            let svg = d3
            .select(scenario)
            .append("svg")
            .attr("id", "map")
            .attr("width", width)
            .attr("height", height);
            tab_svg[scenario]=svg
        })
    }

    onMount(() => {
        
        tab_svg = []
        initSvg();
        iterateScenario("26")

    });
</script>
<h1>Projection sur les anomalies de températures entre {year.substring(0,4)} et {year.substring(5,9)} en France</h1>

<div id="cont" class="row">
    <div class="col-4">
        <p>Selectionnez la période:</p>
        <select name="pets" id="pet-select" on:input={changeYear}>
            <option value="2020_2039">2020-2040</option>
            <option value="2040_2059">2040-2060</option>
            <option value="2060_2079">2060-2080</option>
            <option value="2080_2099">2080-2100</option>
        </select>
    </div>
</div>
<div id="RCP26" />
<div id="RCP45" />
<div id="RCP60" />
<div id="RCP85" />
<div class="footer">
    <svg width="192" height="24" id="svg" >
        <rect fill="#2c7bb6" width="24" height="24" x="0"></rect>
        <rect fill="#abd9e9" width="24" height="24" x="24"></rect>
        <text font-weight="bold" y="20" x="4">0°C</text>
        <rect fill="#ffffbf" width="24" height="24" x="48"></rect>
        <rect fill="#fdae61" width="24" height="24" x="72"></rect>
        <rect fill="#d7191c" width="24" height="24" x="96"></rect>
        <text font-weight="bold" y="20" x="82" fill="black">+6°C</text>
    </svg>
</div>
<a href='#France' class="btn btn-primary btn-lg active" role="button" aria-pressed="true">Precedent</a>
<br>
<br>
<br>
<br>

