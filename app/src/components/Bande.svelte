<style>
    svg {
      margin: 100px
    }

   .center-div {
        margin: 0 auto;
        width: 1000px; 
    }

    #cont {
        text-align: left;
        margin-left: 4px;
        margin-top: 10px;
    }
</style>

<script>
    import { onMount } from "svelte";
    import { attr } from "svelte/internal";

    var tooltip;
    var svg;
    var loadedData = d3.json("/data/data_temp.json");
    let selectedYear = 6;
    let selectCity = 69;
    let maxtemp = 40;
    let mintemp = -20;
    var colorList = [
        "#a50026",
        "#d73027",
        "#f46d43",
        "#fdae61",
        "#fee090",
        "#ffffbf",
        "#e0f3f8",
        "#abd9e9",
        "#74add1",
        "#4575b4",
        "#313695",
    ].reverse();
    var periode = {"0":1930,"1":1940,"2":1950,"3":1960,"4":1970,"5":1980,"6":1990,"7":2000};
    var color = d3
        .scaleQuantize()
        .domain([mintemp, maxtemp])
        .range(colorList.slice(1));

    function calculateIntervalFloor(value, minVal, maxVal, nbPart) {
        var size = maxVal - minVal;
        var sizePart = size / nbPart;
        var floorInterval = minVal;
        while (floorInterval <= value) {
            floorInterval = floorInterval + sizePart;
        }
        return floorInterval - sizePart;
    }

    function drawYearValues(data, year, codeVille, sv, vTemp, pos, depart) {
        let sizeBar = 1.2;
        var filteredData = data.filter(function (entery) {
            return entery.annee == year && entery.IdVille == codeVille;
        });
        for (let i = 0; i < filteredData.length; i++) {
            // if (
            //     filteredData[i][vTemp] != "null" &&
            //     filteredData[i][vTemp] > -800
            // )
                Coloring_Algorithm(
                    parseInt(filteredData[i][vTemp]) * 0.1,
                    depart + i * sizeBar,
                    pos,
                    sizeBar,
                    30,
                    sv
                );
        }
    }

    function Coloring_Algorithm(value, xPos, yPos, w, h, sv) {
        var size = maxtemp - mintemp;
        var sizePart = size / (colorList.length - 1);
        var firstColor = color(value);
        var secondColor = colorList[colorList.indexOf(firstColor) - 1];
        var heightTop =
            (Math.abs(
                value -
                    calculateIntervalFloor(
                        value,
                        mintemp,
                        maxtemp,
                        colorList.length - 1
                    )
            ) *
                h) /
            sizePart;
        // valeurs manquantes
        if (value === null || value < -800) {
            sv.append("rect")
                .attr("x", xPos)
                .attr("y", yPos)
                .attr("class", "R")
                .attr("width", w)
                .attr("height", h)
                .attr("fill", "#ccc")
                .on("mousemove", function (event) {
                    // on recupere la position de la souris
                    // var mousePosition = d3.mouse(this);
                    // on affiche le toolip
                    tooltip
                        .style("visibility", "visible")
                        .style("position", "absolute")
                        // on positionne le tooltip en fonction
                        // de la position de la souris
                        .style("left", event.x + 5 + "px")
                        .style("top", event.y - 25 + "px")
                        // on recupere le nom de l'etat
                        .html("Valeur manquante");
                })
                .on("mouseout", function () {
                    tooltip.style("visibility", "hidden");
                });
        } else {
            sv.append("rect")
                .attr("x", xPos)
                .attr("y", yPos + h - heightTop)
                .attr("width", w)
                .attr("class", "R")
                .attr("height", heightTop)
                .attr("fill", firstColor)
                .on("mousemove", function (event) {
                    // on recupere la position de la souris
                    // var mousePosition = d3.mouse(this);
                    // on affiche le toolip
                    tooltip
                        .style("visibility", "visible")
                        .style("position", "absolute")
                        // on positionne le tooltip en fonction
                        // de la position de la souris
                        .style("left", event.x + 5 + "px")
                        .style("top", event.y - 25 + "px")
                        // on recupere le nom de l'etat
                        .html(Math.round(value * 100) / 100 + " °C");
                })
                .on("mouseout", function () {
                    tooltip.style("visibility", "hidden");
                });

            sv.append("rect")
                .attr("x", xPos)
                .attr("y", yPos)
                .attr("class", "R")
                .attr("width", w)
                .attr("height", h - heightTop)
                .attr("fill", secondColor)
                .on("mousemove", function (event) {
                    // on recupere la position de la souris
                    // var mousePosition = d3.mouse(this);
                    // on affiche le toolip
                    tooltip
                        .style("visibility", "visible")
                        .style("position", "absolute")
                        // on positionne le tooltip en fonction
                        // de la position de la souris
                        .style("left", event.x + 5 + "px")
                        .style("top", event.y - 25 + "px")
                        // on recupere le nom de l'etat
                        .html(Math.round(value * 100) / 100 + " °C");
                })
                .on("mouseout", function () {
                    tooltip.style("visibility", "hidden");
                });
        }
        return [firstColor, secondColor, heightTop];
    }

    function changeCity(event) {
        selectCity = event.target.value;
        reDraw();
    }

    let updateMultiValue = (event) => {
        selectedYear = event.target.value;
        reDraw();
    };

    function reDraw(annees) {
        svg.selectAll(".R").remove();
        svg.selectAll("text").remove();
        var j = 1;
        var years = {"0":1930,"1":1940,"2":1950,"3":1960,"4":1970,"5":1980,"6":1990,"7":2000};
        console.log(parseInt(selectedYear));
        for(let i = years[parseInt(selectedYear)]; i<= (years[parseInt(selectedYear)] + 9); i++){
        loadedData.then((data) => {
            let ville = selectCity;
            let posDepart = 20 * j * 2;
            j++;
            drawYearValues(
                data,
                i,
                ville,
                svg,
                "Tmin",
                posDepart + 10,
                50
            );

            svg.append("text")
            .attr("x", 0)
            .attr("y", posDepart +22)
            .attr("font-size", 20)
            .attr("dy", ".35em")
            .text(i);
            
            drawYearValues(
                data,
                i,
                ville,
                svg,
                "Tmax",
                posDepart + 10,
                530
            );

            
        });
        
    }
    drawScale();
    }

    function drawScale() {
        var diff = 10;
        if (mintemp < 0) {
            diff = -1 * mintemp;
        }
        for (let i = mintemp; i < maxtemp; ) {
            Coloring_Algorithm(i, diff * 4 + i * 4, 470, 4, 20, svg);
            i = i + 0.1;
        }
        for (
            let i = mintemp;
            i <= maxtemp;
            i += (maxtemp - mintemp) / (colorList.length - 1)
        ) {
            svg.append("text")
                .attr("x", diff * 4 + i * 4)
                .attr("y", 500)
                .attr("font-size", 10)
                .attr("dy", ".35em")
                .text(i);
        }
        svg.append("text")
            .attr("x", 0)
            .attr("y", 460)
            .attr("font-size", 10)
            .attr("dy", ".35em")
            .text("Température en °C");

        svg.append("text")
            .attr("x", 50)
            .attr("y", diff)
            .attr("font-size", 10)
            .attr("dy", ".35em")
            .text("Température minimale par jour");

        svg.append("text")
            .attr("x", 530)
            .attr("y", diff)
            .attr("font-size", 10)
            .attr("dy", ".35em")
            .text("Température maximale par jour");
    }

    onMount(() => {
        var width = 1200;
        var height = 530;
        
        tooltip = d3.select("#visu1").append("div");

        svg = d3
            .select("#visu1")
            .append("svg")
            .attr("id", "map")
            .attr("width", width)
            .attr("height", height);

        reDraw(periode);
    });
    //
</script>

<div class="center-div">
    <h2>Evolution des températures historiques par métropoles françaises (bandes)</h2>
    <h3>Pré-raitements:</h3>
    <p>Dans cette visualisation nous avons récupérer les données des grandes métropoles françaises séparemment (Lyon, Paris, Marseille, Bordeaux) que nous avons prétraité pour faire une union. Nous avons réalisé ce prétraitement en python </p>
    <p> Lorsqu'une donnée de température est absente, on l'ignore dans le calcul de la moyenne. </p>
</div>

<div id="cont" class="row">
    <div class="col-4">
        <p>Selectionnez une ville:</p>
        <select name="pets" id="pet-select" on:input={changeCity}>
            <option value="69">Lyon</option>
            <option value="75">Paris</option>
            <option value="31">Toulouse</option>
            <option value="13">Marseille</option>
            <option value="33">Bordeaux</option>
        </select>
    </div>
    <div class="col-4">
        <input
            class="slider"
            id="slider"
            type="range"
            min=0
            max=7
            value={selectedYear}
            step=1
            on:input={updateMultiValue} />
        <h2 class="date" id="date">{periode[selectedYear]}-{periode[selectedYear]+9}</h2>
    </div>
</div>

<div id="visu1" class="center-div"  />
<div style="margin: 2%">
    Référence: <a href="https://www.researchgate.net/publication/4187821_Two-Tone_Pseudo_Coloring_Compact_Visualization_for_One-Dimensional_Data">Two-Tone Pseudo Coloring</a>
    <br>
    Source des données: <a href="https://www.meteo-paris.com/">Météo Paris</a> 
</div>
<a href='#/' class="btn btn-primary btn-lg active" role="button" aria-pressed="true">Precedent</a>
<a href='#Metropoles' class="btn btn-primary btn-lg active" role="button" aria-pressed="true">Suivant</a>

