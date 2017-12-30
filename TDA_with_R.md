---
title: "Using Topology for Data Analysis"
output:
  html_document:
    keep_md: true
  html_notebook: default
---

<br/>
<br/>

When researching data we want to find features that help us understand the information. We look for insight in areas like Machine Learning or other fields in Mathematics and Artificial Intelligence. 

I want to present here a tool initially coming from Mathematics that can be used for exploratory data analysis and give some geometric insight before applying more sophisticated algorithms.

The tool I want to describe is __Persistent Homology__, member of a set of algorithms known as *Topological Data Analysis* __[1,2]__. In this post I will describe the basic methodology when facing a common data analysis scenario: *clustering*.

 <br/>
 
#### SOME IDEAS FROM TOPOLOGY

<br/>

A *space* is a set of data with no structure. The first step is to give some structure that can help us understand the data and also make it more interesting. If we define a notion of how close are all the points we are giving structure to this space. This notion is a neighborhood and it tells us if two points are close. With this notion we already have important information: we now know if our data is connected.

The neighborhoods can be whatever we want and the data points can be numbers or words or other type of data. These concepts and ideas are the subject of study of Topology. For us, Topology is the study of the shape of data.

We need to give some definitions, but all are very intuitive. From our point space or dataset, we define the following notion: a simplex. It is easy to visualize what we mean.

<img  src="simplices.png" width="450" height="400" ></img>

So, a 0-simplex is a point. Every point in our data is a 0-simplex. If we have a "line" joining two points that is a 1-simplex, and so on. Of course, a 4-simplex and higher analogues are difficult for us to visualize. We can immediately see what connectedness is. In the image, we have four connected components, a 0-simplex, a 1-simplx, a 2-simplex and a 3-simplex. If we join them with, for example lines we will connect the dataset into one single component. Like this:

<img  src="simplex_con.png" width="450" height="400" ></img>

The next notion is the neighborhood. We'll use euclidean distance to say when our points are close, we'll use circles as neighborhoods. This distance depends on a parameter, the radius of the circle. If we change these parameter we change the size of the neighborhood.

<img  src="circ.png" width="450" height="400" ></img>

Persistence is an algorithm that changes this parameter from zero to a very large value, one that covers the entire set. With this maximal radius we enclose all our dataset. The algorithm __[2]__ can be put as follows:

1. We construct a neighborhood for each point and set the parameter to zero.

2. Increment the value of this parameter and if two neighborhoods intersect, draw a line between the points. These will form a 1-simplex. After that an n-simplex will form at each step until we fill all the space with lines.

3. Describe in some way the holes of our data has as we increase the parameter. Keep track when they emerge and when they disappear. If the holes and voids persist as we move the parameter, we can say that we found an important feature of a our data.

<img  src="simplex_st.png" width="450" height="400" ></img>

The "some way" part is called Homology and is a field in Mathematics specialized in detecting the structure of space. The reader can refer to the bibliography for these concepts __[1]__.

This algorithm can be shown to detect holes and voids in datasets. *An achievement we can mention is that Persistent Homology was used for detecting a new subtype of breast cancer using it to detect clusters in images __[3]__.*

<br/>

#### VEHICLE ACCIDENTS DATASET

<br/>

The dataset is available in __[4]__. It's about car accidents and has some specifications. We subset only the data we need for this exersice, but a lotmore can be done. 
We use an ID of the accident, the spatial coordinates and some categorical data: Local highway authority and Road Type. That's all we need to start. 

First, the whole data looks like this:


```
## 'data.frame':	140056 obs. of  32 variables:
##  $ Accident_Index                             : Factor w/ 140056 levels "201501BS70001",..: 1 2 3 4 5 6 7 8 9 10 ...
##  $ Location_Easting_OSGR                      : int  525130 526530 524610 524420 524630 525480 526890 527590 524170 525010 ...
##  $ Location_Northing_OSGR                     : int  180050 178560 181080 181080 179040 179530 178940 178660 180930 181200 ...
##  $ Longitude                                  : num  -0.198 -0.179 -0.206 -0.208 -0.206 ...
##  $ Latitude                                   : num  51.5 51.5 51.5 51.5 51.5 ...
##  $ Police_Force                               : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ Accident_Severity                          : int  3 3 3 3 2 3 3 3 3 3 ...
##  $ Number_of_Vehicles                         : int  1 1 1 1 2 2 2 2 2 2 ...
##  $ Number_of_Casualties                       : int  1 1 1 1 1 1 1 1 1 2 ...
##  $ Date                                       : Factor w/ 365 levels "01/01/2015","01/02/2015",..: 133 133 133 145 97 169 169 205 181 229 ...
##  $ Day_of_Week                                : int  2 2 2 3 6 5 5 1 6 3 ...
##  $ Time                                       : Factor w/ 1440 levels "","00:01","00:02",..: 1126 471 1089 461 451 597 550 931 381 945 ...
##  $ Local_Authority_.District.                 : int  12 12 12 12 12 12 12 12 12 12 ...
##  $ Local_Authority_.Highway.                  : Factor w/ 207 levels "E06000001","E06000002",..: 112 112 112 112 112 112 112 112 112 112 ...
##  $ X1st_Road_Class                            : int  5 6 4 4 3 3 3 6 5 6 ...
##  $ X1st_Road_Number                           : int  0 0 415 450 315 315 3218 0 0 0 ...
##  $ Road_Type                                  : int  6 6 6 6 6 6 6 6 6 6 ...
##  $ Speed_limit                                : int  30 30 30 30 30 30 30 30 30 30 ...
##  $ Junction_Detail                            : int  3 3 2 6 6 3 6 0 3 3 ...
##  $ Junction_Control                           : int  4 4 4 4 2 4 2 -1 4 4 ...
##  $ X2nd_Road_Class                            : int  6 3 6 6 3 5 3 -1 6 6 ...
##  $ X2nd_Road_Number                           : int  0 3218 0 0 3220 0 3218 0 0 0 ...
##  $ Pedestrian_Crossing.Human_Control          : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ Pedestrian_Crossing.Physical_Facilities    : int  0 0 1 0 5 4 5 0 1 0 ...
##  $ Light_Conditions                           : int  4 1 4 1 1 1 1 1 1 1 ...
##  $ Weather_Conditions                         : int  1 1 2 1 2 1 8 1 1 8 ...
##  $ Road_Surface_Conditions                    : int  1 1 2 2 2 2 2 1 1 1 ...
##  $ Special_Conditions_at_Site                 : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ Carriageway_Hazards                        : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ Urban_or_Rural_Area                        : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ Did_Police_Officer_Attend_Scene_of_Accident: int  1 1 1 2 2 1 1 1 1 1 ...
##  $ LSOA_of_Accident_Location                  : Factor w/ 28738 levels "","E01000001",..: 2380 2375 2387 2425 2369 2371 2376 2413 2430 2386 ...
```

We start by subseting the data frame with only the variables we are interested. But with a summarise to see it grouped by the highway local authority.


```r
Acc <- Accidents %>% 
  filter(!is.na(Road_Type)) %>% 
  select(Accident_Index,Longitude,Latitude,Local_Authority_.Highway.,Road_Type) %>%
  group_by(Local_Authority_.Highway.) %>% 
  summarise(n())
Acc
```

```
## # A tibble: 207 x 2
##    Local_Authority_.Highway. `n()`
##                       <fctr> <int>
##  1                 E06000001   136
##  2                 E06000002   304
##  3                 E06000003   190
##  4                 E06000004   311
##  5                 E06000005   207
##  6                 E06000006   224
##  7                 E06000007   505
##  8                 E06000008   398
##  9                 E06000009   373
## 10                 E06000010   748
## # ... with 197 more rows
```

In the last output we select any of this subsets and take a look to the road type:


```r
Acc_test <- Accidents %>% 
  filter(!is.na(Road_Type)) %>% 
  filter(Local_Authority_.Highway.== "E06000020") %>%
  select(Accident_Index,Longitude,Latitude,Local_Authority_.Highway.,Road_Type) %>% 
  group_by(Road_Type) %>% 
  summarise(n())
Acc_test
```

```
## # A tibble: 6 x 2
##   Road_Type `n()`
##       <int> <int>
## 1         1     8
## 2         2     1
## 3         3    16
## 4         6   193
## 5         7     4
## 6         9     5
```

We can see that most accidents under these local auhority were under conditions of the road of type 6 and so on... we can also check weather conditions, road conditions, etc. For our purposes we want to visualize spatial data according to this categories:


```r
Acc_test <- Accidents %>% 
  filter(!is.na(Road_Type)) %>% 
  filter(Local_Authority_.Highway.== "E06000020") %>%
  select(Accident_Index,Longitude,Latitude,Local_Authority_.Highway.,Road_Type) 
Acc_test %>% 
  ggvis(x= ~Longitude, y=~Latitude, fill = ~Road_Type) %>% layer_points()
```

<!--html_preserve--><div id="plot_id611710364-container" class="ggvis-output-container">
<div id="plot_id611710364" class="ggvis-output"></div>
<div class="plot-gear-icon">
<nav class="ggvis-control">
<a class="ggvis-dropdown-toggle" title="Controls" onclick="return false;"></a>
<ul class="ggvis-dropdown">
<li>
Renderer: 
<a id="plot_id611710364_renderer_svg" class="ggvis-renderer-button" onclick="return false;" data-plot-id="plot_id611710364" data-renderer="svg">SVG</a>
 | 
<a id="plot_id611710364_renderer_canvas" class="ggvis-renderer-button" onclick="return false;" data-plot-id="plot_id611710364" data-renderer="canvas">Canvas</a>
</li>
<li>
<a id="plot_id611710364_download" class="ggvis-download" data-plot-id="plot_id611710364">Download</a>
</li>
</ul>
</nav>
</div>
</div>
<script type="text/javascript">
var plot_id611710364_spec = {
  "data": [
    {
      "name": ".0",
      "format": {
        "type": "csv",
        "parse": {
          "Longitude": "number",
          "Latitude": "number",
          "Road_Type": "number"
        }
      },
      "values": "\"Longitude\",\"Latitude\",\"Road_Type\"\n-2.48099,52.704674,6\n-2.443791,52.687928,6\n-2.469732,52.640299,6\n-2.426487,52.718799,6\n-2.486844,52.633595,6\n-2.403643,52.749156,6\n-2.448643,52.68212,1\n-2.505663,52.695851,6\n-2.371042,52.770712,6\n-2.428396,52.653733,1\n-2.549015,52.759288,6\n-2.457689,52.648904,6\n-2.518744,52.702124,9\n-2.460816,52.658089,1\n-2.449257,52.681363,1\n-2.516475,52.703006,6\n-2.459891,52.658757,6\n-2.464567,52.688414,6\n-2.506197,52.695948,6\n-2.411908,52.679835,3\n-2.475724,52.636886,6\n-2.488732,52.690205,3\n-2.444554,52.696043,6\n-2.511592,52.667634,6\n-2.368631,52.746825,6\n-2.387973,52.787899,6\n-2.479848,52.677422,6\n-2.46626,52.661214,6\n-2.438613,52.715905,6\n-2.490556,52.691923,3\n-2.482805,52.675981,3\n-2.510924,52.699092,6\n-2.466466,52.681934,6\n-2.437558,52.650437,6\n-2.47294,52.688318,3\n-2.451836,52.634606,9\n-2.421946,52.686929,6\n-2.469737,52.71103,1\n-2.470579,52.717769,6\n-2.517421,52.697886,6\n-2.443705,52.637145,1\n-2.526641,52.689395,3\n-2.368881,52.771078,6\n-2.597382,52.691038,3\n-2.469765,52.71094,3\n-2.447042,52.689003,6\n-2.390417,52.776043,6\n-2.495206,52.696273,6\n-2.470268,52.633806,6\n-2.527572,52.717529,6\n-2.465476,52.669514,6\n-2.473434,52.707959,6\n-2.481494,52.660371,6\n-2.428693,52.658416,6\n-2.456576,52.638534,6\n-2.473336,52.644222,6\n-2.44523,52.714451,6\n-2.461365,52.714389,6\n-2.440029,52.638867,6\n-2.498079,52.701655,6\n-2.491578,52.702581,6\n-2.385607,52.785318,6\n-2.492228,52.707702,6\n-2.452708,52.678913,6\n-2.493833,52.771163,6\n-2.447635,52.689091,6\n-2.361391,52.761572,6\n-2.432134,52.707397,6\n-2.480848,52.695793,6\n-2.476737,52.725835,6\n-2.427926,52.685837,6\n-2.456729,52.639073,6\n-2.52944,52.76175,6\n-2.476995,52.66812,6\n-2.474463,52.707325,6\n-2.472974,52.624356,9\n-2.460378,52.676906,6\n-2.50497,52.696052,6\n-2.44069,52.689926,6\n-2.415545,52.675004,6\n-2.445902,52.678759,6\n-2.399913,52.744638,6\n-2.45785,52.675657,6\n-2.472403,52.708502,6\n-2.472522,52.705805,6\n-2.583861,52.731109,6\n-2.439028,52.716002,6\n-2.472547,52.63964,6\n-2.394396,52.789784,6\n-2.381891,52.767261,6\n-2.496735,52.648441,6\n-2.457021,52.652916,6\n-2.453569,52.70498,6\n-2.455643,52.676744,6\n-2.458265,52.686983,6\n-2.429003,52.629648,6\n-2.477433,52.626855,6\n-2.454164,52.67675,6\n-2.344179,52.740048,6\n-2.571817,52.693411,6\n-2.426136,52.669123,7\n-2.379978,52.768885,2\n-2.435286,52.667202,6\n-2.431675,52.630896,6\n-2.427868,52.725212,6\n-2.50749,52.696491,6\n-2.43481,52.67889,6\n-2.459034,52.70388,6\n-2.484947,52.71852,6\n-2.3711,52.770622,6\n-2.461158,52.680768,6\n-2.528967,52.710421,6\n-2.454468,52.691762,6\n-2.440029,52.638867,6\n-2.447324,52.716511,6\n-2.459925,52.704146,6\n-2.441481,52.680304,3\n-2.579153,52.755764,6\n-2.511782,52.709148,6\n-2.543538,52.787127,6\n-2.399176,52.745,6\n-2.445889,52.6339,6\n-2.470156,52.69224,6\n-2.449459,52.693848,6\n-2.434801,52.707747,6\n-2.479737,52.676739,6\n-2.448529,52.675333,6\n-2.491727,52.70267,6\n-2.470121,52.633897,6\n-2.46628,52.66169,6\n-2.439832,52.678153,3\n-2.464935,52.688215,3\n-2.524619,52.718171,6\n-2.496956,52.706964,6\n-2.512376,52.709325,6\n-2.47152,52.708955,6\n-2.478868,52.663618,6\n-2.531226,52.677059,6\n-2.459696,52.668278,6\n-2.361911,52.77074,1\n-2.455684,52.68061,6\n-2.466286,52.676164,6\n-2.481093,52.704602,6\n-2.543223,52.690399,6\n-2.375937,52.764314,6\n-2.481147,52.696061,6\n-2.439403,52.679593,7\n-2.435155,52.683833,3\n-2.431542,52.7074,6\n-2.549118,52.759233,6\n-2.438616,52.674921,6\n-2.446881,52.687655,7\n-2.491238,52.659162,6\n-2.457752,52.70865,6\n-2.438033,52.705308,6\n-2.515163,52.69466,6\n-2.509192,52.702866,6\n-2.474271,52.716945,1\n-2.437819,52.698747,6\n-2.466116,52.660163,6\n-2.475691,52.629739,6\n-2.514345,52.700597,6\n-2.495242,52.699419,6\n-2.442171,52.719317,6\n-2.437313,52.707289,6\n-2.437329,52.679331,6\n-2.484868,52.671324,6\n-2.493576,52.69601,6\n-2.480958,52.705771,6\n-2.488474,52.689649,3\n-2.419976,52.720566,6\n-2.441129,52.718781,6\n-2.498354,52.699766,6\n-2.450031,52.648718,9\n-2.471422,52.686122,6\n-2.444195,52.627164,6\n-2.443954,52.719939,6\n-2.539651,52.736623,6\n-2.513265,52.709411,6\n-2.457239,52.716203,6\n-2.467194,52.678048,6\n-2.422789,52.644054,6\n-2.455495,52.676745,6\n-2.493994,52.719921,6\n-2.458022,52.635472,6\n-2.362212,52.771188,6\n-2.496287,52.700224,6\n-2.426411,52.666965,7\n-2.425326,52.677037,6\n-2.470579,52.717769,6\n-2.501257,52.759355,6\n-2.533244,52.672735,6\n-2.49118,52.693503,6\n-2.453579,52.691585,6\n-2.492374,52.720647,6\n-2.523698,52.703073,6\n-2.491307,52.691615,6\n-2.529606,52.702148,6\n-2.488474,52.689649,3\n-2.475613,52.704264,6\n-2.454308,52.67639,6\n-2.450063,52.695015,6\n-2.432637,52.653547,3\n-2.409626,52.737862,6\n-2.465019,52.668258,6\n-2.44806,52.716148,6\n-2.47049,52.709499,6\n-2.512886,52.702311,6\n-2.4314,52.708029,6\n-2.496984,52.696446,6\n-2.51076,52.659709,6\n-2.490206,52.672561,6\n-2.53482,52.692865,6\n-2.546653,52.76356,6\n-2.432915,52.726543,6\n-2.457462,52.638531,6\n-2.433624,52.723394,9\n-2.55414,52.757951,6\n-2.457718,52.648868,6\n-2.496914,52.716313,6\n-2.447201,52.646661,6\n-2.466519,52.711852,6\n-2.482848,52.675917,6\n-2.441184,52.680215,3\n-2.518817,52.703274,6\n-2.456248,52.635389,6\n-2.519942,52.698325,6"
    },
    {
      "name": "scale/fill",
      "format": {
        "type": "csv",
        "parse": {
          "domain": "number"
        }
      },
      "values": "\"domain\"\n1\n9"
    },
    {
      "name": "scale/x",
      "format": {
        "type": "csv",
        "parse": {
          "domain": "number"
        }
      },
      "values": "\"domain\"\n-2.61004215\n-2.33151885"
    },
    {
      "name": "scale/y",
      "format": {
        "type": "csv",
        "parse": {
          "domain": "number"
        }
      },
      "values": "\"domain\"\n52.6160846\n52.7980554"
    }
  ],
  "scales": [
    {
      "name": "fill",
      "domain": {
        "data": "scale/fill",
        "field": "data.domain"
      },
      "zero": false,
      "nice": false,
      "clamp": false,
      "range": ["#132B43", "#56B1F7"]
    },
    {
      "name": "x",
      "domain": {
        "data": "scale/x",
        "field": "data.domain"
      },
      "zero": false,
      "nice": false,
      "clamp": false,
      "range": "width"
    },
    {
      "name": "y",
      "domain": {
        "data": "scale/y",
        "field": "data.domain"
      },
      "zero": false,
      "nice": false,
      "clamp": false,
      "range": "height"
    }
  ],
  "marks": [
    {
      "type": "symbol",
      "properties": {
        "update": {
          "size": {
            "value": 50
          },
          "x": {
            "scale": "x",
            "field": "data.Longitude"
          },
          "y": {
            "scale": "y",
            "field": "data.Latitude"
          },
          "fill": {
            "scale": "fill",
            "field": "data.Road_Type"
          }
        },
        "ggvis": {
          "data": {
            "value": ".0"
          }
        }
      },
      "from": {
        "data": ".0"
      }
    }
  ],
  "legends": [
    {
      "orient": "right",
      "fill": "fill",
      "title": "Road_Type"
    }
  ],
  "axes": [
    {
      "type": "x",
      "scale": "x",
      "orient": "bottom",
      "layer": "back",
      "grid": true,
      "title": "Longitude"
    },
    {
      "type": "y",
      "scale": "y",
      "orient": "left",
      "layer": "back",
      "grid": true,
      "title": "Latitude"
    }
  ],
  "padding": null,
  "ggvis_opts": {
    "keep_aspect": false,
    "resizable": true,
    "padding": {},
    "duration": 250,
    "renderer": "svg",
    "hover_duration": 0,
    "width": 672,
    "height": 480
  },
  "handlers": null
};
ggvis.getPlot("plot_id611710364").parseSpec(plot_id611710364_spec);
</script><!--/html_preserve-->

We can see that some clusters form with respect to spatial position. Before we procede with the usual machinery in Machine Learning, we apply the topological method introduced before, Persistence Homology, to this subset of the data.

<br/>

#### PERSISTENT HOMOLOGY

<br/>

We use the [TDA package](https://CRAN.R-project.org/package=TDA).

It's a great package and has many features, but we'll only use a couple to show that topological data analysis is a building block of machine learning and can give geometric insight *a priori* over our data.

We construct the parameter and use *gridDiag* function:


```r
#You can find how to construct this example in TDA package documentation
#library(TDA)

Acc_vector <- cbind("V1" = Acc_test$Longitude, "V2" = Acc_test$Latitude)

xlimit <- c(-2.65, -2.30) 
ylimit <- c(52.5, 52.8)

by <- 0.004

Diag <- gridDiag(X = Acc_vector, FUN = distFct, lim = cbind(xlimit, ylimit), by = by, sublevel = FALSE, library = "Dionysus", printProgress = FALSE, maxdimension = 0)

#To plot the barcode just use the info in the list "diagram":
plot(Diag[["diagram"]],barcode=TRUE)
```

![](TDA_with_R_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

__Comments__:

1. You can find how to construct this example in __TDA__ package documentation.

2. First we need a vector with the spatial data of our accidents: create Acc_vector.

3. We can find the limits of the interval from the plot of by inspection on the dataset with a quick summary.

4. The next parameter *by* is the step size of the grid in which we are looking (we can give a vector to find the best of the steps).

5. *gridDiag* computes the persistence diagram or barcode of the births and deaths of the connected components we are interested.

6. *maxdimension=0* because we only want the $0$-th Homology. *distFct* is the radius of the neighborhood as the infimum of the squared distance.

7. The __Barcode__ is a visualization of the birth and death of connected components. We can see that they form as the radius of the neighborhoodsincrease but also die as the simplex gets high dimensional.

<br/>

#### PASS TO MACHINE LEARNING

<br/>

From the barcode we can expect that 6 to 8 clusters can form and persist. Now we pass this information to a Machine Learning algorithm like the K-MEANS algorithm to find this clusters and the positions of the accidents:


```r
set.seed(10)
AccCluster <- kmeans(Acc_test[, 2:3], 8, nstart = 10)

AccCluster$cluster <- as.factor(AccCluster$cluster)
Acc_test %>% ggvis(x = ~Longitude, y = ~Latitude, fill = ~AccCluster$cluster) %>% 
  layer_points()
```

<!--html_preserve--><div id="plot_id875244194-container" class="ggvis-output-container">
<div id="plot_id875244194" class="ggvis-output"></div>
<div class="plot-gear-icon">
<nav class="ggvis-control">
<a class="ggvis-dropdown-toggle" title="Controls" onclick="return false;"></a>
<ul class="ggvis-dropdown">
<li>
Renderer: 
<a id="plot_id875244194_renderer_svg" class="ggvis-renderer-button" onclick="return false;" data-plot-id="plot_id875244194" data-renderer="svg">SVG</a>
 | 
<a id="plot_id875244194_renderer_canvas" class="ggvis-renderer-button" onclick="return false;" data-plot-id="plot_id875244194" data-renderer="canvas">Canvas</a>
</li>
<li>
<a id="plot_id875244194_download" class="ggvis-download" data-plot-id="plot_id875244194">Download</a>
</li>
</ul>
</nav>
</div>
</div>
<script type="text/javascript">
var plot_id875244194_spec = {
  "data": [
    {
      "name": ".0",
      "format": {
        "type": "csv",
        "parse": {
          "Longitude": "number",
          "Latitude": "number"
        }
      },
      "values": "\"Longitude\",\"Latitude\",\"AccCluster$cluster\"\n-2.48099,52.704674,\"6\"\n-2.443791,52.687928,\"3\"\n-2.469732,52.640299,\"8\"\n-2.426487,52.718799,\"2\"\n-2.486844,52.633595,\"8\"\n-2.403643,52.749156,\"5\"\n-2.448643,52.68212,\"3\"\n-2.505663,52.695851,\"1\"\n-2.371042,52.770712,\"5\"\n-2.428396,52.653733,\"3\"\n-2.549015,52.759288,\"7\"\n-2.457689,52.648904,\"8\"\n-2.518744,52.702124,\"1\"\n-2.460816,52.658089,\"4\"\n-2.449257,52.681363,\"3\"\n-2.516475,52.703006,\"1\"\n-2.459891,52.658757,\"4\"\n-2.464567,52.688414,\"4\"\n-2.506197,52.695948,\"1\"\n-2.411908,52.679835,\"3\"\n-2.475724,52.636886,\"8\"\n-2.488732,52.690205,\"6\"\n-2.444554,52.696043,\"3\"\n-2.511592,52.667634,\"1\"\n-2.368631,52.746825,\"5\"\n-2.387973,52.787899,\"5\"\n-2.479848,52.677422,\"4\"\n-2.46626,52.661214,\"4\"\n-2.438613,52.715905,\"2\"\n-2.490556,52.691923,\"6\"\n-2.482805,52.675981,\"4\"\n-2.510924,52.699092,\"1\"\n-2.466466,52.681934,\"4\"\n-2.437558,52.650437,\"8\"\n-2.47294,52.688318,\"4\"\n-2.451836,52.634606,\"8\"\n-2.421946,52.686929,\"3\"\n-2.469737,52.71103,\"6\"\n-2.470579,52.717769,\"6\"\n-2.517421,52.697886,\"1\"\n-2.443705,52.637145,\"8\"\n-2.526641,52.689395,\"1\"\n-2.368881,52.771078,\"5\"\n-2.597382,52.691038,\"1\"\n-2.469765,52.71094,\"6\"\n-2.447042,52.689003,\"3\"\n-2.390417,52.776043,\"5\"\n-2.495206,52.696273,\"6\"\n-2.470268,52.633806,\"8\"\n-2.527572,52.717529,\"1\"\n-2.465476,52.669514,\"4\"\n-2.473434,52.707959,\"6\"\n-2.481494,52.660371,\"4\"\n-2.428693,52.658416,\"3\"\n-2.456576,52.638534,\"8\"\n-2.473336,52.644222,\"8\"\n-2.44523,52.714451,\"2\"\n-2.461365,52.714389,\"2\"\n-2.440029,52.638867,\"8\"\n-2.498079,52.701655,\"6\"\n-2.491578,52.702581,\"6\"\n-2.385607,52.785318,\"5\"\n-2.492228,52.707702,\"6\"\n-2.452708,52.678913,\"3\"\n-2.493833,52.771163,\"7\"\n-2.447635,52.689091,\"3\"\n-2.361391,52.761572,\"5\"\n-2.432134,52.707397,\"2\"\n-2.480848,52.695793,\"6\"\n-2.476737,52.725835,\"6\"\n-2.427926,52.685837,\"3\"\n-2.456729,52.639073,\"8\"\n-2.52944,52.76175,\"7\"\n-2.476995,52.66812,\"4\"\n-2.474463,52.707325,\"6\"\n-2.472974,52.624356,\"8\"\n-2.460378,52.676906,\"4\"\n-2.50497,52.696052,\"1\"\n-2.44069,52.689926,\"3\"\n-2.415545,52.675004,\"3\"\n-2.445902,52.678759,\"3\"\n-2.399913,52.744638,\"5\"\n-2.45785,52.675657,\"4\"\n-2.472403,52.708502,\"6\"\n-2.472522,52.705805,\"6\"\n-2.583861,52.731109,\"7\"\n-2.439028,52.716002,\"2\"\n-2.472547,52.63964,\"8\"\n-2.394396,52.789784,\"5\"\n-2.381891,52.767261,\"5\"\n-2.496735,52.648441,\"4\"\n-2.457021,52.652916,\"8\"\n-2.453569,52.70498,\"2\"\n-2.455643,52.676744,\"3\"\n-2.458265,52.686983,\"3\"\n-2.429003,52.629648,\"8\"\n-2.477433,52.626855,\"8\"\n-2.454164,52.67675,\"3\"\n-2.344179,52.740048,\"5\"\n-2.571817,52.693411,\"1\"\n-2.426136,52.669123,\"3\"\n-2.379978,52.768885,\"5\"\n-2.435286,52.667202,\"3\"\n-2.431675,52.630896,\"8\"\n-2.427868,52.725212,\"2\"\n-2.50749,52.696491,\"1\"\n-2.43481,52.67889,\"3\"\n-2.459034,52.70388,\"2\"\n-2.484947,52.71852,\"6\"\n-2.3711,52.770622,\"5\"\n-2.461158,52.680768,\"4\"\n-2.528967,52.710421,\"1\"\n-2.454468,52.691762,\"3\"\n-2.440029,52.638867,\"8\"\n-2.447324,52.716511,\"2\"\n-2.459925,52.704146,\"2\"\n-2.441481,52.680304,\"3\"\n-2.579153,52.755764,\"7\"\n-2.511782,52.709148,\"1\"\n-2.543538,52.787127,\"7\"\n-2.399176,52.745,\"5\"\n-2.445889,52.6339,\"8\"\n-2.470156,52.69224,\"6\"\n-2.449459,52.693848,\"3\"\n-2.434801,52.707747,\"2\"\n-2.479737,52.676739,\"4\"\n-2.448529,52.675333,\"3\"\n-2.491727,52.70267,\"6\"\n-2.470121,52.633897,\"8\"\n-2.46628,52.66169,\"4\"\n-2.439832,52.678153,\"3\"\n-2.464935,52.688215,\"4\"\n-2.524619,52.718171,\"1\"\n-2.496956,52.706964,\"6\"\n-2.512376,52.709325,\"1\"\n-2.47152,52.708955,\"6\"\n-2.478868,52.663618,\"4\"\n-2.531226,52.677059,\"1\"\n-2.459696,52.668278,\"4\"\n-2.361911,52.77074,\"5\"\n-2.455684,52.68061,\"3\"\n-2.466286,52.676164,\"4\"\n-2.481093,52.704602,\"6\"\n-2.543223,52.690399,\"1\"\n-2.375937,52.764314,\"5\"\n-2.481147,52.696061,\"6\"\n-2.439403,52.679593,\"3\"\n-2.435155,52.683833,\"3\"\n-2.431542,52.7074,\"2\"\n-2.549118,52.759233,\"7\"\n-2.438616,52.674921,\"3\"\n-2.446881,52.687655,\"3\"\n-2.491238,52.659162,\"4\"\n-2.457752,52.70865,\"2\"\n-2.438033,52.705308,\"2\"\n-2.515163,52.69466,\"1\"\n-2.509192,52.702866,\"1\"\n-2.474271,52.716945,\"6\"\n-2.437819,52.698747,\"2\"\n-2.466116,52.660163,\"4\"\n-2.475691,52.629739,\"8\"\n-2.514345,52.700597,\"1\"\n-2.495242,52.699419,\"6\"\n-2.442171,52.719317,\"2\"\n-2.437313,52.707289,\"2\"\n-2.437329,52.679331,\"3\"\n-2.484868,52.671324,\"4\"\n-2.493576,52.69601,\"6\"\n-2.480958,52.705771,\"6\"\n-2.488474,52.689649,\"6\"\n-2.419976,52.720566,\"2\"\n-2.441129,52.718781,\"2\"\n-2.498354,52.699766,\"6\"\n-2.450031,52.648718,\"8\"\n-2.471422,52.686122,\"4\"\n-2.444195,52.627164,\"8\"\n-2.443954,52.719939,\"2\"\n-2.539651,52.736623,\"7\"\n-2.513265,52.709411,\"1\"\n-2.457239,52.716203,\"2\"\n-2.467194,52.678048,\"4\"\n-2.422789,52.644054,\"8\"\n-2.455495,52.676745,\"3\"\n-2.493994,52.719921,\"6\"\n-2.458022,52.635472,\"8\"\n-2.362212,52.771188,\"5\"\n-2.496287,52.700224,\"6\"\n-2.426411,52.666965,\"3\"\n-2.425326,52.677037,\"3\"\n-2.470579,52.717769,\"6\"\n-2.501257,52.759355,\"7\"\n-2.533244,52.672735,\"1\"\n-2.49118,52.693503,\"6\"\n-2.453579,52.691585,\"3\"\n-2.492374,52.720647,\"6\"\n-2.523698,52.703073,\"1\"\n-2.491307,52.691615,\"6\"\n-2.529606,52.702148,\"1\"\n-2.488474,52.689649,\"6\"\n-2.475613,52.704264,\"6\"\n-2.454308,52.67639,\"3\"\n-2.450063,52.695015,\"3\"\n-2.432637,52.653547,\"8\"\n-2.409626,52.737862,\"2\"\n-2.465019,52.668258,\"4\"\n-2.44806,52.716148,\"2\"\n-2.47049,52.709499,\"6\"\n-2.512886,52.702311,\"1\"\n-2.4314,52.708029,\"2\"\n-2.496984,52.696446,\"6\"\n-2.51076,52.659709,\"1\"\n-2.490206,52.672561,\"4\"\n-2.53482,52.692865,\"1\"\n-2.546653,52.76356,\"7\"\n-2.432915,52.726543,\"2\"\n-2.457462,52.638531,\"8\"\n-2.433624,52.723394,\"2\"\n-2.55414,52.757951,\"7\"\n-2.457718,52.648868,\"8\"\n-2.496914,52.716313,\"6\"\n-2.447201,52.646661,\"8\"\n-2.466519,52.711852,\"6\"\n-2.482848,52.675917,\"4\"\n-2.441184,52.680215,\"3\"\n-2.518817,52.703274,\"1\"\n-2.456248,52.635389,\"8\"\n-2.519942,52.698325,\"1\""
    },
    {
      "name": "scale/fill",
      "format": {
        "type": "csv",
        "parse": {}
      },
      "values": "\"domain\"\n\"1\"\n\"2\"\n\"3\"\n\"4\"\n\"5\"\n\"6\"\n\"7\"\n\"8\""
    },
    {
      "name": "scale/x",
      "format": {
        "type": "csv",
        "parse": {
          "domain": "number"
        }
      },
      "values": "\"domain\"\n-2.61004215\n-2.33151885"
    },
    {
      "name": "scale/y",
      "format": {
        "type": "csv",
        "parse": {
          "domain": "number"
        }
      },
      "values": "\"domain\"\n52.6160846\n52.7980554"
    }
  ],
  "scales": [
    {
      "name": "fill",
      "type": "ordinal",
      "domain": {
        "data": "scale/fill",
        "field": "data.domain"
      },
      "points": true,
      "sort": false,
      "range": "category10"
    },
    {
      "name": "x",
      "domain": {
        "data": "scale/x",
        "field": "data.domain"
      },
      "zero": false,
      "nice": false,
      "clamp": false,
      "range": "width"
    },
    {
      "name": "y",
      "domain": {
        "data": "scale/y",
        "field": "data.domain"
      },
      "zero": false,
      "nice": false,
      "clamp": false,
      "range": "height"
    }
  ],
  "marks": [
    {
      "type": "symbol",
      "properties": {
        "update": {
          "size": {
            "value": 50
          },
          "x": {
            "scale": "x",
            "field": "data.Longitude"
          },
          "y": {
            "scale": "y",
            "field": "data.Latitude"
          },
          "fill": {
            "scale": "fill",
            "field": "data.AccCluster$cluster"
          }
        },
        "ggvis": {
          "data": {
            "value": ".0"
          }
        }
      },
      "from": {
        "data": ".0"
      }
    }
  ],
  "legends": [
    {
      "orient": "right",
      "fill": "fill",
      "title": "AccCluster$cluster"
    }
  ],
  "axes": [
    {
      "type": "x",
      "scale": "x",
      "orient": "bottom",
      "layer": "back",
      "grid": true,
      "title": "Longitude"
    },
    {
      "type": "y",
      "scale": "y",
      "orient": "left",
      "layer": "back",
      "grid": true,
      "title": "Latitude"
    }
  ],
  "padding": null,
  "ggvis_opts": {
    "keep_aspect": false,
    "resizable": true,
    "padding": {},
    "duration": 250,
    "renderer": "svg",
    "hover_duration": 0,
    "width": 672,
    "height": 480
  },
  "handlers": null
};
ggvis.getPlot("plot_id875244194").parseSpec(plot_id875244194_spec);
</script><!--/html_preserve-->

<br/>

#### CONCLUSIONS:

<br/>

Persistence is a topological property. The construction we mentioned before is called a *filtration*. Over these filtrations we calculate the dimensions of the Homology groups and this gives us the important information. The dimension of the $0$-th Homology group or $H_0$ gives us the number of connected components of our space of data. The first Homology tell us the number of holes of the space. Second Homology tell's us about voids, ans so on. 

We only talked about $H_0$ because our data is in $2D$ spatial dimensions. But we point out that this must not be a limitation. The topological analysis is not limited to spatial data.

As Machine Learning techniques are used to find new features in data, topological data analysis can be used to confirm robustness of those results and also bring to light new features hidden in the structure of data that is of topological nature.

<br/>

#### SOME REFERENCES:

<br/>

1. Carlsson, Gunnar; Zomorodian, Afra; Collins, Anne; Guibas, Leonidas J. (2005-12-01). "Persistence barcodes for shapes". International Journal of Shape Modeling. 11 (02): 149-187.

2. Carlsson, Gunnar (2009-01-01). "Topology and data". Bulletin of the American Mathematical Society. 46 (2): 255-308.

3. Nicolau M., Levine A., Clarsson G. (2010-07-23), "Topology based data analysis identifies a subgroup of breast cancer with a unique mutational profile and excellent survival", PNAS, 108(17).

4. https://data.gov.uk/

<br/>


