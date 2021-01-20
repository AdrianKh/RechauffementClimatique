<style>
    svg {
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
    import { onMount } from 'svelte';
    
    var svg 
    var tab_svg
    var width 
    var height
    var lyon_geo =  d3.json("/data/lyon_geojson.json")
    var paris_geo = d3.json("/data/paris_geojson.json")
    var bordeaux_geo = d3.json("/data/bordeaux_geojson.json")
    var toulouse_geo = d3.json("/data/toulouse_geojson.json")
    var marseille_geo = d3.json("/data/marseille_geojson.json")
    var temperatures = d3.json("/data/data_temp.json")
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
    function mean_temps(data){
        var sumMax = 0
        var sumMin = 0
        var dividerMax= 0
        var dividerMin= 0
       
        data.forEach(elem => {
            if(elem.QTmax != '9.0'){
                sumMax+= elem.Tmax
                dividerMin+=1
            }
            if (elem.QTmin != '9'){
                sumMin+= elem.Tmin
                dividerMax+=1
            }
        })
        return { "meanMin":  sumMin/dividerMin,
                 "meanMax":  sumMax/dividerMax }
    }

    function min_max(data){
        var vMax = -100000
        var vMin = 1000000
       
        data.forEach(elem => {
            if(elem.QTmax != '9.0'){
                if(vMax < elem.Tmax) vMax = elem.Tmax
                if(vMin > elem.Tmax) vMin = elem.Tmax
                
            }
        })
        return { "min":  vMin,
                 "max":  vMax }
    }
    //init_color("01","1920","2005")

    const process_data = (annee, mois, ville, sv) => {
        return temperatures.then((temp) => {
            var filtered = temp.filter(val => {
                return val.annee > annee-10 &&  val.annee <= annee && val.mois === mois && val.IdVille == ville
            })
            var means = mean_temps(filtered)
            var geo;
            if (ville == "75") geo = paris_geo
            else if (ville == "69") geo = lyon_geo
            else if (ville == "13") geo = marseille_geo
            else if (ville == "31") geo = toulouse_geo
            else if (ville == "33") geo = bordeaux_geo 
            var maxmin = min_max(filtered);
            return geo.then(data => {
                for (var i = 0; i< data.features.length; i++){
                    data.features[i].properties.Tmin= means.meanMin
                    data.features[i].properties.Tmax= means.meanMax
                }
                draw_colors(data,sv,ville, maxmin, annee)
                return data
            })
        })
    }

    const draw_colors = (data, sv, ville,maxmin, annee) => {

        var color = d3.scaleQuantize();

        var projection 
        if (ville === "75") projection =  d3.geoConicConformal().center([2.3488, 48.8534]).scale(30000).translate([width / 2, height / 2]);
        else if (ville === "69") projection =  d3.geoConicConformal().center([4.85, 45.75]).scale(50000).translate([width / 2, height / 2]);
        else if (ville === "31") projection =  d3.geoConicConformal().center([1.4437,43.6043]).scale(15000).translate([width / 2, height / 2]);
        else if (ville === "13") projection = d3.geoConicConformal().center([5.4, 43.3]).scale(20000).translate([width / 2, height / 2]);
        else if (ville === "33") projection = d3.geoConicConformal().center([-0.57918, 44.837789]).scale(16000).translate([width / 2, height / 2]);

        var path = d3.geoPath().projection(projection);

        const carte =  sv.selectAll("path")
                          .data(data.features)
        
        color.domain([50,maxmin.max])
             .range(['#67001f','#b2182b','#d6604d','#f4a582','#fddbc7','#f7f7f7','#d1e5f0','#92c5de','#4393c3','#2166ac','#053061'].reverse()); 

        carte.enter()
             .append("path")
             .attr("d", path)
             .style("fill", function(elem){
                return color(elem.properties.Tmax)
             })
             .attr("stroke","black")
             .attr("stroke-width", "0.2px")
             
        
        carte.style("fill", function(elem){
                return color(elem.properties.Tmax)
              })
              .attr("stroke","black")
              .attr("stroke-width", "0.2px")
        
        sv.append("text")
          .attr("x",  0)
          .attr("y", height )
          .text(() => {return annee-10 + "-" +annee});
    }

    let year;
    let month;

    var years = [1930,1940,1950,1960,1970,1980,1990,2000,2008]
    var visus = ['#l69','#p75', '#m13', "#b33"]

    const updateViewMonth = (event) => { 
        month = event.target.value
        if (parseInt(month) < 10){ 
            month = "0"+month
        } 
        var y=0;
        visus.forEach(visu => {
            tab_svg[visu].forEach(element => {
                process_data(years[y],month,visu.substring(2),element)
                y++;
            });
            y=0;
        })
        
    }

    const svg_init= (visus) => {
        visus.forEach(visu => {
            tab_svg[visu] = []
            for(let i=0; i<years.length;i++){
                let svg = d3
                .select(visu)
                .append("svg")
                .attr("id","map")
                .attr("width", width)
                .attr("height", height)
                .style("margin-left", "30px")
                tab_svg[visu].push(svg)
            }  
        })  
    }

    onMount(() => {

        width = 80;
        height = 120;
        tab_svg = []
        
        svg_init(visus)
        
        // process_data(year,month).then((data) => {
        //     draw_colors(data,svg,ville)
        // })
        
        updateViewMonth({"target" : {"value": "12"}})

    })    
    </script>
    <div class="center-div">
    <h2>Evolution des températures historiques par métropoles françaises (cartes)</h2>
        <h3>Pré-raitements:</h3>
        <p>Dans cette visualisation nous avons récupérer les données des grandes métropoles françaises séparemment (Lyon, Paris, Marseille, Bordeaux) que nous avons prétraité pour faire une union. Nous avons réalisé ce prétraitement en python </p>
        <p> Lorsqu'une donnée de température est absente, on l'ignore dans le calcul de la moyenne. </p>
        
    </div>
    <div id="visu">
        <!-- <Inner on:message={handleMessage}/> -->

        <input
            class="slider"
            id="slider2"
            type="range"
            min="1"
            max="12"
            step="1"
            on:input={updateViewMonth}
        />

        <p>Mois : {list_mois[parseInt(month)-1]}</p>
    </div>
    <div id="l69">
        <p>Lyon</p>
    </div>
    <hr>
    <div id="p75">
        <p>Paris</p>
    </div>
    <hr>
    <div id="m13">
        <p>Marseille</p>
    </div>
    <hr>
    <div id="b33">
        <p>Bordeaux</p>
    </div>
    <hr>

<div class="footer">
<svg width="264" height="24" >
    <rect fill="#053061" width="24" height="24" x="0"></rect>
    <rect fill="#2166ac" width="24" height="24" x="24"></rect>
    <text font-weight="bold" y="20" x="4" fill="white">-20°C</text>
    <rect fill="#4393c3" width="24" height="24" x="48"></rect>
    <rect fill="#92c5de" width="24" height="24" x="72"></rect>
    <rect fill="#d1e5f0" width="24" height="24" x="96"></rect>
    <rect fill="#f7f7f7" width="24" height="24" x="120"></rect>
    <rect fill="#fddbc7" width="24" height="24" x="144"></rect>
    <rect fill="#f4a582" width="24" height="24" x="168"></rect>
    <rect fill="#d6604d" width="24" height="24" x="192"></rect>
    <rect fill="#b2182b" width="24" height="24" x="216"></rect>
    <rect fill="#67001f" width="24" height="24" x="240"></rect>
    <text font-weight="bold" y="20" x="218" fill="white">+40°C</text>
</svg> 
</div>
<br>
<div style="margin: 2%">
    Source des données: <a href="https://www.meteo-paris.com/">Météo Paris</a> 
</div>

<a href='#Bande' class="btn btn-primary btn-lg active" role="button" aria-pressed="true">Precedent</a>
<a href="#France" class="btn btn-primary btn-lg active" role="button" aria-pressed="true">Suivant</a>
<br>
<br>
<br>
<br>