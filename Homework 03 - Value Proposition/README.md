## [A day in the life of Master's student (BADS-7105)](https://sukitpom.github.io/BADS-7105/Homework%2003%20-%20Value%20proposition/index.html)


# Data prepare
Prepare data in Excel file.

Source file: 
https://github.com/sukitpom/BADS7105/tree/master/Homework%2003%20-%20Value%20Proposition/data

![Image](https://github.com/sukitpom/BADS7105/blob/master/Homework%2003%20-%20Value%20Proposition/img/prepare_data.jpg)

# Summary data
Summary data and create the value proposition.

![Image](https://github.com/sukitpom/BADS7105/blob/master/Homework%2003%20-%20Value%20Proposition/img/Value_proposition.jpg)

# Create visulization
Generate visualize by d3.js

Reference: https://flowingdata.com/2015/12/15/a-day-in-the-life-of-americans/

Add script in html file:

    <script src="js/d3-3-5-5.min.js"></script>
    <script>
        var margin = {top: 0, right: 0, bottom: 0, left: 0},
        width = 1500 - margin.left - margin.right,
        height = 1000 - margin.top - margin.bottom;
        ...
    </script>

Create svg for chart object:

    var svg = d3.select("#chart").append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

Create node object:

    var node_radius = 10,
        padding = 1,
        cluster_padding = 5,
        num_nodes = 33;
    var nodes = d3.range(0, num_nodes).map(function(o, i) {
        return {
            id: "node" + i,
            x: foci.cenpos.x + Math.random(),   //x: foci.toppos.x + Math.random(),
            y: foci.cenpos.y + Math.random(),   //y: foci.toppos.y + Math.random(),
            radius: node_radius,
            choice: "cenpos",  //start cluster
        }
    });
    
Function draw circle:

    var circle = svg.selectAll("circle")
        .data(nodes)
        .enter().append("circle")
        .attr("id", function(d) { return d.id; })
        .attr("class", "node")
        .style("fill", function(d) { return foci[d.choice].color; });  

Create label activities:

    var center_act = "Sleeping",
        center_pt = { "x": 750, "y": 365 };
        var act_codes = [
        {"index": "0", "short": "Sleeping", "desc": "Sleeping"},
        {"index": "1", "short": "Gaming", "desc": "Play games"},
        {"index": "2", "short": "Relaxing", "desc": "Socializing, Relaxing, and Leisure"},
        {"index": "3", "short": "Sports", "desc": "Sports, Exercise, and Recreation"},
        {"index": "4", "short": "Commuting", "desc": "Commuting"},
        {"index": "5", "short": "Personal Care", "desc": "Personal Care"},
        {"index": "6", "short": "Eating & Drinking", "desc": "Eating and Drinking"},
        {"index": "7", "short": "Education", "desc": "Education"},
        {"index": "8", "short": "Work", "desc": "Work and Work-Related Activities"},
        {"index": "9", "short": "Housework", "desc": "Household Activities"},
        {"index": "10", "short": "Meeting", "desc": "Meeting of work."},
        {"index": "11", "short": "Family", "desc": "Family activities"},
        {"index": "12", "short": "Shopping", "desc": "Consumer Purchases"},
        ];

Create force layout on circle:

    var foci = {
        "cenpos": { x: 750, y: 450, color: "#C0C0C0" },  //sleeping
        "rightbottompos1": { x: 1040, y: 610, color: "#FF3399" },  //gamin
        "rightbottompos2": { x: 950, y: 740, color: "#FF5050" }, //relaxing                        
        "bottompos": { x: 750, y: 795, color: "#FF9933" }, //sports  
        "bottomleftpos2": { x: 580, y: 740, color: "#33CCCC" },   //commuting
        "bottomleftpos1": { x: 460, y: 610, color: "#00FF99" },   //personal care                      
        "leftpos": { x: 410, y: 450, color: "#33CC33" }, //eating
        "lefttoppos2": { x: 460, y: 290, color: "#99FF33" },  //education 
        "lefttoppos1": { x: 580, y: 165, color: "#FFFF00" }, //work
        "toppos": { x: 750, y: 100, color: "#0099FF" },  //housework
        "toprightpos1": { x: 950, y: 165, color: "#3366FF" },  //meeting
        "toprightpos2": { x: 1040, y: 290, color: "#9966FF" },   //family
        "rightpos": { x: 1090, y: 450, color: "#FF00FF" },  //shopping
    };


Create function for change position of node:

    function tick(e) {
        circle
          .each(gravity(0.051 * e.alpha))
          .each(collide(.5))
          .style("fill", function(d) { return foci[d.choice].color; })
          .attr("cx", 100)  
          .attr("cx", function(d) { return d.x; })   //เปลี่ยนตำแหน่ง x
          .attr("cy", function(d) { return d.y; });
    }

Create function timer for delay loop to move node:

    var timeout;
    function timer() {           
        console.log(timeout + '=' + document.getElementById("time_number").innerHTML.substring(0, 5))
        var k;
        for (k = 0; k < 721; k++){  //max 721
            if (act_list[k].time_index==timeout) {
                //console.log(timeout + '=' + document.getElementById("time_number").innerHTML.substring(0, 5) + ',' + act_list[k].node_index + ',' + act_list[k].foci_index + ',' + act_list[k].desc);
                var foci_index = act_list[k].foci_index;
                var choice = d3.keys(foci)[foci_index];
                nodes[act_list[k].node_index].cx = foci[choice].x;
                nodes[act_list[k].node_index].cy = foci[choice].y;
                nodes[act_list[k].node_index].choice = choice;   
                document.getElementById("activities").innerHTML= "ID:" + act_list[k].node_index + '=' + act_list[k].desc;                 
                force.resume();
            };
        }
            countnodes();
            // Update percentages
            label.selectAll("tspan.actpct")                
                .text(function(d) {
                    //console.log(d.index);
                    return readablePercent(act_counts[d.index]);
                });
        if (timeout==355) {
            //timeout = 0;   //set speed  default = 400  //delay for loop
            clearTimeout;
        }
        else {
            timeout = setTimeout(timer, 240);   //set speed  default = 400  //delay for loop
        };
    }

 Caculated percentage of node on activities in the same time:
 
    function countnodes (){
      var j;
      for (j = 0; j < 13; j++) {
          act_counts[j] = 0;
          nodes.forEach((nd) => {
              if (nd["choice"]==d3.keys(foci)[j]) {
                  act_counts[j] = act_counts[j]+1;
              };
          });
      };
    };

    function readablePercent(n) {
        
        //var pct = 100 * n / 1000;
        var pct = n / num_nodes *100;
        //console.log(n);
        if (pct < 1 && pct > 0) {
            pct = "<1%";
        } else {
            pct = Math.round(pct) + "%";
        }
        //console.log(pct);
        return pct;

    }

      
Create array from raw data for generate visualize:

    var act_list = [
        {"time_index":1,"node_index":0,"foci_index":0,"desc":"นอน"},
        {"time_index":1,"node_index":2,"foci_index":11,"desc":"เล่นกับแมว"},
        {"time_index":1,"node_index":3,"foci_index":0,"desc":"นอน"},
        {"time_index":1,"node_index":9,"foci_index":2,"desc":"ดูโคนันเดอะซีรี่ย์"},
        {"time_index":1,"node_index":10,"foci_index":2,"desc":"ตื่นนอน"},
        {"time_index":1,"node_index":13,"foci_index":0,"desc":"เข้านอน"},
        {"time_index":1,"node_index":14,"foci_index":8,"desc":"เคลียร์งานนิดหน่อย"},
        {"time_index":1,"node_index":16,"foci_index":7,"desc":"อ่านบทความ"},
        {"time_index":1,"node_index":17,"foci_index":0,"desc":"นอนหลับ"},
        {"time_index":1,"node_index":18,"foci_index":1,"desc":"เล่นเกมส์กับเพื่อน"},
        {"time_index":1,"node_index":19,"foci_index":0,"desc":"นอน"},
        {"time_index":1,"node_index":21,"foci_index":0,"desc":"นอน"},
        {"time_index":1,"node_index":24,"foci_index":2,"desc":"สวดมนต์"},
        {"time_index":1,"node_index":24,"foci_index":0,"desc":"เข้านอน"},
        ...
      ]
  
