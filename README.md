# Grafana-Dashboads
Working with Grafana, a widely used data visualization tool, to create visualizations and reports in the logistics context. Technical skills and creativity are applied to provide effective visual solutions that help to understand and analyze logistics data efficiently. The main objective is to use Grafana to create interactive dashboards and custom graphs that display relevant logistics information, such as transportation performance, shipment tracking, delivery times, and other key performance indicators (KPIs) related to the supply chain.
Close collaboration with the company's development team, who will provide real and representative datasets for participants to explore and visualize. This way, a better understanding of the logistics process and the challenges associated with supply chain management will be achieved.

- [Notebooks](#notebooks)
- [Instance](#instance)
- [Solution](#solution)


## Notebooks
1. [GeoMap.ipynb](#geomapipynb)
2. [Timeline.ipynb](#timelineipynb)
3. [Transform JSON and Wait Times.ipynb](#transform-json-and-wait-timesipynb)

### Transform JSON and Wait Times.ipynb

The objective of this notebook is to create a JSON (e.g., solutionComplete-20-50-2-0.json) from the JSON instance and solution, adding the necessary information to properly represent the visualizations and execute adjacent scripts.
- An array 'desplazamientos' is added, containing information on each vehicle's movement, such as initial position, final position, accumulated cost, etc.
- An array 'mixedTrayectories' is added, containing the movements of all vehicles together.
- A series of scripts to obtain the vehicles' waiting times are added. After running them all one by one, a CSV named '7conflict_rows3' followed by the instance name (e.g., 20-50-2-0) is obtained, where groups of vehicle movements that are in conflict for being in the same position at the same time can be observed. Manual review is necessary since not all of them are real conflicts. Once reviewed, they must be manually added to the timeline CSV in Grafana, adjusting times if a vehicle needs to remain on hold according to priority.

### GeoMap.ipynb

The objective of this notebook is to represent the GeoMap element in the Grafana dashboard, by generating coordinates of the positions and other scripts to add data to the visualization.
- The first script generates the coordinates of the positions.
- Scripts 2 and 3 generate the trajectories of the vehicles, resulting in as many CSV files as vehicles in the instance, which need to be manually added to the GeoMap element of the Solution dashboard.
- Scripts 4, 5, and 6 create CSV files with the coordinates of the jobs, and must be executed one by one. Script 6.1 creates a JSON-structured file representing the GeoMap's csvContent, to be copied into the JSON file of the Instance dashboard in Grafana, avoiding the need to add jobs one by one, which can be a costly process if there are many jobs. Script 6.2 creates another JSON file adding the routes of the jobs to the GeoMap element in Grafana, reading the previously added csvContent, since to complete a job it must start at an initial position, move to the position where it ends, and finish the job at this last position.

### Timeline.ipynb
Executing the script in this notebook will create a CSV with the structure ready to be added as a data source in the State timeline element in Grafana, representing the states of each vehicle.

## Instance
1. [Instance Information](#instance-information)
2. [Histogram of distances](#histogram-of-distances)
3. [Histogram of distances (Grouped by 5)](#histogram-of-distances-grouped-by-5)
4. [Vehicles Priority](#vehicles-priority)
5. [Vehicles Time](#vehicles-time)
6. [Vehicles Information](#vehicles-information)
7. [Jobs GeoMap](#jobs-geomap)

### Instance Information

These are numerical data representing information about the instance being worked on. The 'yesoreyeram-infinity-datasource' Datasource is used with the backend parser and the instance URL with the IP assigned by Docker to the local server (e.g., http://172.17.0.2:7000/instances/instance-60-200-10-0.json). In the datasource options, only the variable to be represented needs to be written.

### Histogram of distances

Represents a histogram of distances showing a balance in the distances between positions. The 'distances' array from the JSON is selected with the 'value' column.

### Histogram of distances (Grouped by 5)

Represents the same as Histogram of distances but grouped in groups of 5.

### Vehicles Priority

Shows the priority of each vehicle with a bar graph. It influences the waiting times of vehicles in case of conflicts in positions. The 'id' and 'priority' fields from the 'vehicles' array are read.

### Vehicles Time

Shows the time it takes for each vehicle to start with a bar graph. This will be the initial time at the beginning of each vehicle's route. The 'id' and 'time' fields from the 'vehicles' array are read.
### Vehicles Information

Displays a table with the fields 'id', 'priority', and 'initialPos' of each element of 'vehicles'.
### Jobs GeoMap

Represents the initial and final positions of the jobs, where each union between one position and another represents a job. The 'grafana-testdata-datasource' Datasource is used for this.

First, a 'CSV Content' query is added, in which the CSV file with the coordinates of the jobs is pasted. To generate the CSV containing the initial and final positions of each job, along with their coordinates, the first step is to generate a CSV with the instance positions' coordinates using script 1 from the 'GeoMap.ipynb' notebook. Once the coordinates are obtained, script 4 is used to obtain a CSV with the positions of all jobs, resulting in the file 'jobs-coordinates-1.csv'; and script 5 is used to add the jobs' position coordinates to the previous file, modifying the 'jobs-coordinates-1.csv' file. Once this file contains the positions of the jobs along with the 'latitude' and 'longitude' coordinate fields, it will be the one pasted into the query to display the jobs.

After this, the initial and final positions of the jobs should be displayed, but not the union between them. To represent the union, it is necessary to add a query for each job, for example:

```bash

node,latitude,longitude
126,0.96224,1.05886
50,0.20882,0.54142
```

Now it is necessary to add a 'route' type 'Map layer' in the options of the 'Geomap' element, pointing with the 'Data' option to the job query to be represented. However, this is a very costly task. To speed up this process, scripts 6.1 and 6.2 from the 'GeoMap.ipynb' notebook are used. Script 6.1 reads the 'jobs-coordinates-1.csv' file and adds a block like the one shown below for each job, to a JSON file (e.g., 'instance-60-200-10-0-geomap2.json').

```bash

{
 "csvContent": csv_content,
 "datasource": {
    "type": "grafana-testdata-datasource",
    "uid": "edksd0i6rrdhcd"
 },
 "refId": ref_id,
 "scenarioId": "csv_content"
}  
```

Script 6.2 also reads the 'jobs-coordinates-1.csv' file and adds a block like the one shown below for each job, to a JSON file (e.g., 'instance-60-200-10-0-geomap3.json').

```bash

{
        "config": {
            "arrow": 0,
            "style": {
                "color": {
                    "fixed": "light-orange"
                },
                "lineWidth": 2,
                "opacity": 1,
                "rotation": {
                    "fixed": 0,
                    "max": 360,
                    "min": -360,
                    "mode": "mod"
                },
                "size": {
                    "fixed": 5,
                    "max": 15,
                    "min": 2
                },
                "symbol": {
                    "fixed": "img/icons/marker/circle.svg",
                    "mode": "fixed"
                },
                "symbolAlign": {
                    "horizontal": "center",
                    "vertical": "center"
                },
                "textConfig": {
                    "fontSize": 12,
                    "offsetX": 0,
                    "offsetY": 0,
                    "textAlign": "center",
                    "textBaseline": "middle"
                }
            }
        },
        "filterData": {
            "id": "byRefId",
            "options": ref_id
        },
        "location": {
            "mode": "auto"
        },
        "name": f"Layer {ref_id_index - 1}",
        "opacity": 0.7,
        "tooltip": True,
        "type": "route"
    }  
```

Once we have the two JSON files, one to add the queries and another to add the 'Map layer' of each query, we only need to add each JSON file just after each block in the Grafana dashboard JSON. When saved, the elements will automatically be added to the Geomap component. If there's an error saving the dashboard JSON after modifying it, just change the version at the end of the JSON to the previous version.

## Solution
1. [Solution Information](#solution-information)
2. [Accumulated Cost by id](#accumulated-cost-by-id)
3. [Vehicle % Status](#vehicle-%-status)
4. [Job Quantity by Vehicle](#job-quantity-by-vehicle)
5. [Vehicle Trajectories GeoMap](#vehicle-trajectories-geomap)
6. [Single Vehicle Trajectories GeoMap](#single-vehicle-trajectories-geomap)
7. [Vehicles Timeline](#vehicles-timeline)

### Solution Information

This includes numerical data representing information about the instance being worked on. The 'yesoreyeram-infinity-datasource' Datasource is used with the backend parser and the instance URL with the IP assigned by Docker to the local server (e.g., http://172.17.0.2:7000/instances/instance-60-200-10-0.json). Variables such as the number of jobs, the number of possible positions for the vehicles, and the number of vehicles are displayed.

### Accumulated Cost by id

Shows the evolution of the accumulated cost of each vehicle using the Trend element in Grafana. Information such as which vehicle finishes earlier or later and the number of movements each vehicle makes can be seen. A JSON solutionComplete file is needed, which can be obtained from the 'Transform JSON and Wait Times.ipynb' notebook. A query is added for each vehicle in the instance, and a color is assigned to each in the options. For example, for vehicle 1, use the URL http://172.17.0.2:7000/solutionsComplete/solutionComplete-60-200-10-0.json, selecting routes[0].trayectoria.desplazamientos and the columns 'accumulatedCost' and 'id'. After adding all the queries, a data transformation is necessary. The type is 'Join by field', selecting 'OUTER (TIME SERIES)' and 'id (base field name)'. Also, in the Trend element options, select the 'id' field on the X-axis.

### Vehicle % Status
This stacked bar chart shows the status of vehicles by the percentage of time they spend moving without working, moving while working, and being stopped while starting or finishing a job. A query is used with the URL http://172.17.0.2:7000/solutionsComplete/solutionComplete-60-200-10-0.json, and with routes[0].vehicleInfo for each vehicle. The fields selected are 'vehicleId', 'relMoveEmpty', 'relMoveWorking', and 'relStoppedTime'. To show the percentages of each category in the same color, it is necessary to adjust value ranges or exact values as needed according to the JSON values. Additionally, a 'Join by field' data transformation is required with 'OUTER (TIME SERIES)' mode and referring to the 'vehicleId (base field name)' field. In the element options, select the 'vehicleId' field on the X-axis.

### Job Quantity by Vehicle
This bar chart shows the number of jobs each vehicle performs. A query is used with the URL http://172.17.0.2:7000/solutionsComplete/solutionComplete-60-200-10-0.json, and with routes[0].vehicleInfo for each vehicle. The fields selected are 'vehicleId' and 'route_length'. Additionally, a 'Join by field' data transformation with 'OUTER (TIME SERIES)' mode is required, referring to the 'vehicleId (base field name)' field.

### Vehicle Trajectories GeoMap
Represents the movements of vehicles throughout their routes, showing positions based on coordinates obtained from the 'GeoMap.ipynb' notebook. Once the coordinates CSV file is generated, it is necessary to determine the positions each vehicle passes through. This is achieved by running script 2 from the notebook, which obtains the positions; and script 3, which adds the coordinates according to the obtained coordinates file. After running these, CSV files for each vehicle in the instance are obtained, each with the positions and coordinates they pass through. Now, a CSV Content query needs to be added for each CSV file of each vehicle. Another query is needed after merging all vehicle CSVs, which is used to represent the positions numerically by adding a layer of markers in the options. For the other queries, a route layer is added for each, selecting the color of each vehicle.

### Single Vehicle Trajectories GeoMap
Represents the movements of a single vehicle individually. This is achieved using a variable created in Grafana named $vehicle. With scripts 2 and 3 from the 'GeoMap.ipynb' notebook, CSV files with each vehicle's routes are generated. Therefore, a query needs to be added in the GeoMap element with the infinity datasource, reading the URL http://172.17.0.2:7000/geomap/geomap-60-200-10-0/geomap-$vehicle.csv. A 'route' layer and a 'markers' layer are also needed.

### Vehicles Timeline
This component represents the most complex part of the dashboard, showing the evolution of each vehicle's actions over time. The script from the 'Timeline.ipynb' notebook is executed, resulting in a CSV file like timeline-20-50-2-0.csv. This file contains a 'timestamps' field in YY-MM-DD hh:mm
format and a 'Vehicle n' field for each vehicle in the instance. The 'timestamps' field contains the moment a vehicle starts an action, and the 'Vehicle n' field contains the action performed by the respective vehicle. Only the field corresponding to each vehicle should be completed. For correct visualization of the timeline, a 'Convert file type' data transformation is needed, selecting the 'timestamps' field as 'Time'. To represent colors, mappings of action values are also necessary in the State timeline component options in Grafana.

[@mpadillagarcia](https://github.com/mpadillagarcia)
