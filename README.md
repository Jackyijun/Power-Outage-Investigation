# Power-Outage-Investigation
**by David Sun & Yijun Luo**

>Click [me](https://jackkkkkkdzk.github.io/Power-Outage-Investigation/) to see all visualizations

>The Data used for this exlploratory analysis is [here](https://engineering.purdue.edu/LASCI/research-data/outages/outagerisks).

# Introduction
This analysis works on a dataset pertaining to the major power outages witnessed across the US, from January 2000, to July 2016. Gathered and compiled by Sayanti Mukherjee and others in [this article](https://www.sciencedirect.com/science/article/pii/S2352340918307182), the dataset includes information on the specific time of each outage, the causes related to outage, regional climate information, impact of outage, geographic and economic statistics of the affected state, regional land usage and population information. 

One of the major objectives for analyzing power outages is to understand the underlying causes, identify attributes that facilitates the incident, and summarize an overal risk factor for each region. In this analysis, we took the lasting duration of each outage as the measure for impact severity, and centered our focus around the question, **what are the major causes for the varying duration of outages? More specifically, What attributes tends to produce longer duration outages?** Understanding this question is significant, as it could lead to future research on generating a holistic risk factor considering all attributes, and helping with outage prevention in real world scenarios. 

There is a total of 1534 rows of data, each corresponding to a single observed power outage within the time frame in continental US. The major columns that are related to this investigation are:
- YEAR: year of which the power outage took place
- MONTH: month of which the power outage took place
- U.S._STATE: state of which the power outage occured
- NERC.REGION: the North American Electric Reliability Corporation (NERC) involved in this power outage
- CLIMATE.REGION: U.S. climate region of which the state belongs to
- ANOMALY.LEVEL: an indicator of the the warm and cold episodes brought by El Niño/La Niña
- CLIMATE.CATEGORY: "warm", "normal", "cold", determined by ANOMALY.LEVEL
- OUTAGE.START.DATE and TIME: the exact time the power outage happened
- OUTAGE.RESTORATION.DATE and TIME: the exact time the power was rebooted
- CAUSE.CATEGORY: general category of the cause for the incident
- CAUSE.CATEGORY.DETAIL: detailed direct cause for the incident
- OUTAGE.DURATION: the number of minutes that the outage lasted
- DEMAND.LOSS.MW: the peak demand loss or total demand loss, measured in megawatts
- CUSTOMERS.AFFECTED: number of customers effected

# Cleaning and EDA
### Data Cleaning
After loading the data, few of the columns mentioned above caught our attention. There are missing values in the OUTAGE.RESTORATION.DATE, OUTAGE.RESTORATION.TIME, CAUSE.CATEGORY.DETAIL, OUTAGE.DURATION, DEMAND.LOSS.MW, and CUSTOMERS.AFFECTED columns. The missing values regarding restoration is likely due to a permanent shutdown or unknown reasons, so we could not arbitrarily impute values to replace them. The missingness in the cause category detail column is likely NMAR, explained in the analysis section below, and it isn't necessary to replace null values, as most rows are identical to the CAUSE.CATEGORY column. The OUTAGE.DURATION, DEMAND.LOSS.MW, and CUSTOMERS.AFFECTED columns all contain NaN values and 0s, but we have determined that the 0s are valid and faithful data. OUTAGE.DURATION contains zeros as a direct result of subtracting the start time from the restoration time, so if the power was immediately restored, the duration could become 0 minutes. In both DEMAND.LOSS.MW and CUSTOMERS.AFFECTED, the zeros represent minimal amount of damage done during the outage, which indicates less than 1 megawatt of demand loss, and not significant amount of population affected. 

We added two columns, OUTAGE.START and OUTAGE.RESTORATION, to the dataframe by combining the corresponding date and time column, storing the elements as Timestamp objects. This is done to conveniently map time series data.

This is the first five rows of cleaned data

|   OBS |   YEAR |   MONTH | U.S._STATE   | POSTAL.CODE   | NERC.REGION   | CLIMATE.REGION     |   ANOMALY.LEVEL | CLIMATE.CATEGORY   | OUTAGE.START.DATE   | OUTAGE.START.TIME   | OUTAGE.RESTORATION.DATE   | OUTAGE.RESTORATION.TIME   | CAUSE.CATEGORY     | CAUSE.CATEGORY.DETAIL   |   HURRICANE.NAMES |   OUTAGE.DURATION |   DEMAND.LOSS.MW |   CUSTOMERS.AFFECTED |   RES.PRICE |   COM.PRICE |   IND.PRICE |   TOTAL.PRICE |   RES.SALES |   COM.SALES |   IND.SALES |   TOTAL.SALES |   RES.PERCEN |   COM.PERCEN |   IND.PERCEN |   RES.CUSTOMERS |   COM.CUSTOMERS |   IND.CUSTOMERS |   TOTAL.CUSTOMERS |   RES.CUST.PCT |   COM.CUST.PCT |   IND.CUST.PCT |   PC.REALGSP.STATE |   PC.REALGSP.USA |   PC.REALGSP.REL |   PC.REALGSP.CHANGE |   UTIL.REALGSP |   TOTAL.REALGSP |   UTIL.CONTRI |   PI.UTIL.OFUSA |   POPULATION |   POPPCT_URBAN |   POPPCT_UC |   POPDEN_URBAN |   POPDEN_UC |   POPDEN_RURAL |   AREAPCT_URBAN |   AREAPCT_UC |   PCT_LAND |   PCT_WATER_TOT |   PCT_WATER_INLAND | OUTAGE.START        | OUTAGE.RESTORATION   |
|------:|-------:|--------:|:-------------|:--------------|:--------------|:-------------------|----------------:|:-------------------|:--------------------|:--------------------|:--------------------------|:--------------------------|:-------------------|:------------------------|------------------:|------------------:|-----------------:|---------------------:|------------:|------------:|------------:|--------------:|------------:|------------:|------------:|--------------:|-------------:|-------------:|-------------:|----------------:|----------------:|----------------:|------------------:|---------------:|---------------:|---------------:|-------------------:|-----------------:|-----------------:|--------------------:|---------------:|----------------:|--------------:|----------------:|-------------:|---------------:|------------:|---------------:|------------:|---------------:|----------------:|-------------:|-----------:|----------------:|-------------------:|:--------------------|:---------------------|
|     1 |   2011 |       7 | Minnesota    | MN            | MRO           | East North Central |            -0.3 | normal             | 2011-07-01 00:00:00 | 17:00:00            | 2011-07-03 00:00:00       | 20:00:00                  | severe weather     | nan                     |               nan |              3060 |              nan |                70000 |       11.6  |        9.18 |        6.81 |          9.28 | 2.33292e+06 | 2.11477e+06 | 2.11329e+06 |   6.56252e+06 |      35.5491 |      32.225  |      32.2024 |         2308736 |          276286 |           10673 |           2595696 |        88.9448 |        10.644  |       0.411181 |              51268 |            47586 |          1.07738 |                 1.6 |           4802 |          274182 |       1.75139 |             2.2 |      5348119 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2011-07-01 17:00:00 | 2011-07-03 20:00:00  |
|     2 |   2014 |       5 | Minnesota    | MN            | MRO           | East North Central |            -0.1 | normal             | 2014-05-11 00:00:00 | 18:38:00            | 2014-05-11 00:00:00       | 18:39:00                  | intentional attack | vandalism               |               nan |                 1 |              nan |                  nan |       12.12 |        9.71 |        6.49 |          9.28 | 1.58699e+06 | 1.80776e+06 | 1.88793e+06 |   5.28423e+06 |      30.0325 |      34.2104 |      35.7276 |         2345860 |          284978 |            9898 |           2640737 |        88.8335 |        10.7916 |       0.37482  |              53499 |            49091 |          1.08979 |                 1.9 |           5226 |          291955 |       1.79    |             2.2 |      5457125 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2014-05-11 18:38:00 | 2014-05-11 18:39:00  |
|     3 |   2010 |      10 | Minnesota    | MN            | MRO           | East North Central |            -1.5 | cold               | 2010-10-26 00:00:00 | 20:00:00            | 2010-10-28 00:00:00       | 22:00:00                  | severe weather     | heavy wind              |               nan |              3000 |              nan |                70000 |       10.87 |        8.19 |        6.07 |          8.15 | 1.46729e+06 | 1.80168e+06 | 1.9513e+06  |   5.22212e+06 |      28.0977 |      34.501  |      37.366  |         2300291 |          276463 |           10150 |           2586905 |        88.9206 |        10.687  |       0.392361 |              50447 |            47287 |          1.06683 |                 2.7 |           4571 |          267895 |       1.70627 |             2.1 |      5310903 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2010-10-26 20:00:00 | 2010-10-28 22:00:00  |
|     4 |   2012 |       6 | Minnesota    | MN            | MRO           | East North Central |            -0.1 | normal             | 2012-06-19 00:00:00 | 04:30:00            | 2012-06-20 00:00:00       | 23:00:00                  | severe weather     | thunderstorm            |               nan |              2550 |              nan |                68200 |       11.79 |        9.25 |        6.71 |          9.19 | 1.85152e+06 | 1.94117e+06 | 1.99303e+06 |   5.78706e+06 |      31.9941 |      33.5433 |      34.4393 |         2317336 |          278466 |           11010 |           2606813 |        88.8954 |        10.6822 |       0.422355 |              51598 |            48156 |          1.07148 |                 0.6 |           5364 |          277627 |       1.93209 |             2.2 |      5380443 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2012-06-19 04:30:00 | 2012-06-20 23:00:00  |
|     5 |   2015 |       7 | Minnesota    | MN            | MRO           | East North Central |             1.2 | warm               | 2015-07-18 00:00:00 | 02:00:00            | 2015-07-19 00:00:00       | 07:00:00                  | severe weather     | nan                     |               nan |              1740 |              250 |               250000 |       13.07 |       10.16 |        7.74 |         10.43 | 2.02888e+06 | 2.16161e+06 | 1.77794e+06 |   5.97034e+06 |      33.9826 |      36.2059 |      29.7795 |         2374674 |          289044 |            9812 |           2673531 |        88.8216 |        10.8113 |       0.367005 |              54431 |            49844 |          1.09203 |                 1.7 |           4873 |          292023 |       1.6687  |             2.2 |      5489594 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2015-07-18 02:00:00 | 2015-07-19 07:00:00  |



### Univariate Analysis
Below is a choropleth for median duration of power outage with respect to each state. From this plot, we can see that the North East regions suffer longer power outages, while the West Coast in general suffer shorter ones. This could be attributed to the climate region those states belong to.
<iframe src="assets/fig_1_uni_choropleth.html" width=800 height=600 frameBorder=0></iframe>

Below is a histogram showing the distribution of outages according to duration. We can see that most outages aggregates around the lower end, and over half of them is below 600 minutes long. This means that most power outages can be fixed within a reasonable amount of time, while the others take much longer to restore.
<iframe src="assets/fig_2_uni_hist.html" width=800 height=600 frameBorder=0></iframe>


### Bivariate Analysis
Below is a barchart showing the proportion of outages caused by servere weather each year. In general, we find that the proportion of outages caused by severe weather decreases over the years. (Year 2001 does not follow this trend because it has too little recorded outages.) Presumably, this shows that U.S. has improved its power infrastructures over the years to withstand severe weather.
<iframe src="assets/fig_3_bi_hist.html" width=800 height=600 frameBorder=0></iframe>

Below is a scatterplot showing the relationship of mean total power sales versus population of each state. It suggests that total sales of power is positively correlated with the population, which means higher state population corresponds to more power consumption.
<iframe src="assets/fig_4_bi_scatter.html" width=800 height=600 frameBorder=0></iframe>


### Interesting Aggregates
The following pivot table is a breakdown of average outage duration by state and cause category. This helps to visualize which cause category has the most significant impact on length of outage for a particular state. 

**Mean outage duration measured of each state by cause category**

| U.S._STATE           |   equipment failure |   fuel supply emergency |   intentional attack |   islanding |   public appeal |   severe weather |   system operability disruption |
|:---------------------|--------------------:|------------------------:|---------------------:|------------:|----------------:|-----------------:|--------------------------------:|
| Alabama              |             nan     |                   nan   |            77        |    nan      |          nan    |          1421.75 |                         nan     |
| Arizona              |             138.5   |                   nan   |           639.6      |    nan      |          nan    |         25726.5  |                         384.5   |
| Arkansas             |             105     |                   nan   |           547.833    |      3      |         1063.71 |          2701.8  |                         nan     |
| California           |             524.81  |                  6154.6 |           946.458    |    214.857  |         2028.11 |          2928.37 |                         363.667 |
| Colorado             |             nan     |                   nan   |           117        |      2      |          nan    |          2727.25 |                         279.75  |
| Connecticut          |             nan     |                   nan   |            49.125    |    nan      |          nan    |          2262.6  |                         nan     |
| Delaware             |              50     |                   nan   |            38.9189   |    nan      |          nan    |          2153.5  |                         nan     |
| District of Columbia |             159     |                   nan   |           nan        |    nan      |          nan    |          4764.11 |                         nan     |
| Florida              |             554.5   |                   nan   |            50        |    nan      |         4320    |          6420.19 |                         205.7   |
| Georgia              |             nan     |                   nan   |           108        |    nan      |          nan    |          1422.75 |                         nan     |
| Hawaii               |             nan     |                   nan   |           nan        |    nan      |          nan    |           997.5  |                         237     |
| Idaho                |             nan     |                   nan   |           307.5      |    nan      |         1548    |           nan    |                         179.667 |
| Illinois             |             149     |                  2761   |          1450        |    nan      |          120    |          1650.7  |                         nan     |
| Indiana              |               1     |                 12240   |           421.875    |    125.333  |          nan    |          4523.29 |                        4671.6   |
| Iowa                 |             nan     |                   nan   |          5657.8      |    nan      |          nan    |          3353.67 |                         nan     |
| Kansas               |             nan     |                   nan   |           561        |    nan      |          913    |          9346    |                         nan     |
| Kentucky             |             652     |                 12570   |           108        |    nan      |          nan    |          4480.11 |                         nan     |
| Louisiana            |             176.333 |                 28170   |           nan        |    nan      |         1359.21 |          7186.93 |                        1144.67  |
| Maine                |             nan     |                  1676   |            82.6667   |    881      |          nan    |          1669.4  |                         nan     |
| Maryland             |             nan     |                   nan   |           225.32     |    nan      |          nan    |          4006.94 |                         304     |
| Massachusetts        |             nan     |                  2891   |           384.25     |    nan      |          nan    |          1556.57 |                          67     |
| Michigan             |           26435.3   |                   nan   |          3635.25     |      1      |         1078    |          4831.65 |                        2610     |
| Minnesota            |             nan     |                   nan   |           369.5      |    nan      |          nan    |          3585.55 |                         nan     |
| Mississippi          |             nan     |                   nan   |            12        |    nan      |          nan    |           nan    |                         300     |
| Missouri             |             nan     |                   nan   |           408        |    nan      |          nan    |          4483.82 |                          65     |
| Montana              |             nan     |                   nan   |            93        |     34.5    |          nan    |           nan    |                         nan     |
| Nebraska             |             nan     |                   nan   |           nan        |    nan      |          159    |          3221.33 |                         nan     |
| Nevada               |             nan     |                   nan   |           553.286    |    nan      |          nan    |           nan    |                         nan     |
| New Hampshire        |             nan     |                   nan   |            60        |    nan      |          nan    |          1597.5  |                         nan     |
| New Jersey           |             nan     |                   nan   |            91.125    |    nan      |          nan    |          6372.86 |                         748.5   |
| New Mexico           |             nan     |                    76   |           174.5      |    nan      |          nan    |           nan    |                           0     |
| New York             |             247     |                 16687.2 |           309.083    |    nan      |         2655    |          6034.58 |                        1176.57  |
| North Carolina       |             nan     |                   nan   |          1063.75     |    nan      |          nan    |          1738.93 |                          82.2   |
| North Dakota         |             nan     |                   nan   |           nan        |    nan      |          720    |           nan    |                         nan     |
| Ohio                 |             nan     |                   nan   |           327.286    |    nan      |          nan    |          4322.27 |                        1744.5   |
| Oklahoma             |             nan     |                   nan   |            75.6667   |    984      |          704    |          4206.47 |                         nan     |
| Oregon               |             200     |                   nan   |           394.105    |    nan      |          nan    |          2295.8  |                         nan     |
| Pennsylvania         |             376     |                   nan   |          1526.83     |    nan      |          nan    |          4314    |                         329     |
| South Carolina       |             nan     |                   nan   |           nan        |    nan      |          nan    |          3135    |                         nan     |
| South Dakota         |             nan     |                   nan   |           nan        |    120      |          nan    |           nan    |                         nan     |
| Tennessee            |             404     |                   nan   |           171        |    nan      |         2700    |          1386.35 |                          20     |
| Texas                |             405.6   |                 13920   |           298.769    |    nan      |         1140.41 |          3854.89 |                         810.8   |
| Utah                 |              15     |                   nan   |           142.286    |    nan      |         2275    |           957    |                         537.5   |
| Vermont              |             nan     |                   nan   |            35.4444   |    nan      |          nan    |           nan    |                         nan     |
| Virginia             |             nan     |                   nan   |             2        |    nan      |          683.5  |          1132.28 |                         241     |
| Washington           |            1204     |                     1   |           371.871    |     73.3333 |          248    |          5473.55 |                          25     |
| West Virginia        |             nan     |                   nan   |             1        |    nan      |          nan    |          9305    |                         nan     |
| Wisconsin            |             nan     |                 33971.2 |           459        |    nan      |          388    |          1527.43 |                         nan     |
| Wyoming              |              61     |                   nan   |             0.333333 |     32      |          nan    |           106    |                         nan     |


----------------
This following pivot table indicates the number of outages occured in each state, broken down by cause categories. This helps to visualize the most common cause of outages in a particular state. 

**Number of Outage of each state by cause category**

| U.S._STATE           |   equipment failure |   fuel supply emergency |   intentional attack |   islanding |   public appeal |   severe weather |   system operability disruption |
|:---------------------|--------------------:|------------------------:|---------------------:|------------:|----------------:|-----------------:|--------------------------------:|
| Alabama              |                   0 |                       0 |                    1 |           0 |               0 |                4 |                               0 |
| Alaska               |                   0 |                       0 |                    0 |           0 |               0 |                0 |                               0 |
| Arizona              |                   4 |                       0 |                   15 |           0 |               0 |                4 |                               2 |
| Arkansas             |                   1 |                       0 |                    6 |           1 |               7 |               10 |                               0 |
| California           |                  21 |                      10 |                   24 |          28 |               9 |               67 |                              39 |
| Colorado             |                   0 |                       0 |                    5 |           1 |               0 |                4 |                               4 |
| Connecticut          |                   0 |                       0 |                    8 |           0 |               0 |               10 |                               0 |
| Delaware             |                   1 |                       0 |                   37 |           0 |               0 |                2 |                               0 |
| District of Columbia |                   1 |                       0 |                    0 |           0 |               0 |                9 |                               0 |
| Florida              |                   4 |                       0 |                    2 |           0 |               3 |               26 |                              10 |
| Georgia              |                   0 |                       0 |                    1 |           0 |               0 |               16 |                               0 |
| Hawaii               |                   0 |                       0 |                    0 |           0 |               0 |                4 |                               1 |
| Idaho                |                   0 |                       0 |                    4 |           0 |               1 |                0 |                               3 |
| Illinois             |                   1 |                       1 |                    1 |           0 |               1 |               40 |                               0 |
| Indiana              |                   1 |                       1 |                    8 |           3 |               0 |               24 |                               5 |
| Iowa                 |                   0 |                       0 |                    5 |           0 |               0 |                3 |                               0 |
| Kansas               |                   0 |                       0 |                    3 |           0 |               1 |                3 |                               0 |
| Kentucky             |                   1 |                       2 |                    1 |           0 |               0 |                9 |                               0 |
| Louisiana            |                   3 |                       1 |                    0 |           0 |              14 |               14 |                               6 |
| Maine                |                   0 |                       1 |                    6 |           1 |               0 |               10 |                               0 |
| Maryland             |                   0 |                       0 |                   25 |           0 |               0 |               32 |                               1 |
| Massachusetts        |                   0 |                       1 |                    8 |           0 |               0 |                7 |                               2 |
| Michigan             |                   3 |                       0 |                    4 |           1 |               1 |               83 |                               3 |
| Minnesota            |                   0 |                       0 |                    4 |           0 |               0 |               11 |                               0 |
| Mississippi          |                   0 |                       0 |                    3 |           0 |               0 |                0 |                               1 |
| Missouri             |                   0 |                       0 |                    3 |           0 |               0 |               11 |                               1 |
| Montana              |                   0 |                       0 |                    1 |           2 |               0 |                0 |                               0 |
| Nebraska             |                   0 |                       0 |                    0 |           0 |               1 |                3 |                               0 |
| Nevada               |                   0 |                       0 |                    7 |           0 |               0 |                0 |                               0 |
| New Hampshire        |                   0 |                       0 |                   12 |           0 |               0 |                2 |                               0 |
| New Jersey           |                   0 |                       0 |                    8 |           0 |               0 |               22 |                               2 |
| New Mexico           |                   0 |                       1 |                    6 |           0 |               0 |                0 |                               1 |
| New York             |                   2 |                      12 |                   12 |           0 |               4 |               33 |                               7 |
| North Carolina       |                   0 |                       0 |                    4 |           0 |               0 |               30 |                               5 |
| North Dakota         |                   0 |                       0 |                    0 |           0 |               1 |                0 |                               0 |
| Ohio                 |                   0 |                       0 |                   14 |           0 |               0 |               26 |                               2 |
| Oklahoma             |                   0 |                       0 |                    3 |           1 |               3 |               15 |                               0 |
| Oregon               |                   1 |                       0 |                   19 |           0 |               0 |                5 |                               0 |
| Pennsylvania         |                   1 |                       0 |                    6 |           0 |               0 |               48 |                               2 |
| South Carolina       |                   0 |                       0 |                    0 |           0 |               0 |                8 |                               0 |
| South Dakota         |                   0 |                       0 |                    0 |           2 |               0 |                0 |                               0 |
| Tennessee            |                   2 |                       0 |                    6 |           0 |               1 |               20 |                               2 |
| Texas                |                   5 |                       3 |                   13 |           0 |              17 |               64 |                              20 |
| Utah                 |                   1 |                       0 |                   35 |           0 |               1 |                2 |                               2 |
| Vermont              |                   0 |                       0 |                    9 |           0 |               0 |                0 |                               0 |
| Virginia             |                   0 |                       0 |                    1 |           0 |               2 |               32 |                               1 |
| Washington           |                   1 |                       1 |                   62 |           3 |               1 |               20 |                               1 |
| West Virginia        |                   0 |                       0 |                    1 |           0 |               0 |                3 |                               0 |
| Wisconsin            |                   0 |                       4 |                    7 |           0 |               1 |                7 |                               0 |
| Wyoming              |                   1 |                       0 |                    3 |           1 |               0 |                1 |                               0 |


# Assessment of Missingness
### NMAR Analysis
The missingness mechanism of column **CAUSE.CATEGORY.DETAIL** is **NMAR**. This column appears to be documented and written by researchers, as the labels used for detailed causes are quite messy and inconsistent. For example, there are two very similar labels "Coal" and " Coal", both of which corresponds to a power outage caused by a coal power plant issue. Another occurance is the various notations of wind damage, including "heavy wind", "wind/rain", "wind storm", and "wind". These clues imply that this column is reported by hand, and the names of each label varies from one person to another. Therefore, it is very likely that the missing values are an incident of human error while collecting the information. If the cause details are unknown to the researcher, or the causes are quite obvious and not worth writing its details, then the researcher is more likely to not write anything within this column. And so, the missing values are depended on the missing values itself.

Values found in **CAUSE.CATEGORY.DETAIL**: [nan, 'vandalism', 'heavy wind', 'thunderstorm', 'winter storm',
'tornadoes', 'sabotage', 'hailstorm', 'uncontrolled loss',
'winter', 'wind storm', 'computer hardware', 'public appeal',
'storm', ' Coal', ' Natural Gas', 'hurricanes', 'wind/rain',
'snow/ice storm', 'snow/ice ', 'transmission interruption',
'flooding', 'transformer outage', 'generator trip',
'relaying malfunction', 'transmission trip', 'lightning',
'switching', 'shed load', 'line fault', 'breaker trip', 'wildfire',
' Hydro', 'majorsystem interruption', 'voltage reduction',
'transmission', 'Coal', 'substation', 'heatwave',
'distribution interruption', 'wind', 'suspicious activity',
'feeder shutdown', '100 MW loadshed', 'plant trip', 'fog', 'Hydro',
'earthquake', 'HVSubstation interruption', 'cables', 'Petroleum',
'thunderstorm; islanding', 'failure']

### Missingness Dependency
##### Missingness of Outage Duration(OUTAGE.DURATION) depends on Cause(CAUSE.CATEGORY)
We tested if the column ***OUTAGE.DURATION***'s missingness is depended on the values of column ***CAUSE.CATEGORY***.
We performed a permutation test, using total variation distance (TVD) as our test statistics, to find out the answer. 

This following grouped bar chart indicates the observed distribution of ***CAUSE.CATEGORY***, separated by the missingness of the corresponding outage duration value. 

<iframe src="assets/fig_5_missing_depends.html" width=800 height=600 frameBorder=0></iframe>

This is the resulting distribution of permutation TVDs versus observed TVD. We can see that the red line is to the right of the entire blue distribution, meaning that our test generated a p-value of approximately 0. This means that the difference between the observed distribution of cause category when duration ***is*** missing, versus the observed distribution of cause category when duration ***is not*** missing, is ***significant***. Thus, the missingness of OUTAGE.DURATION is likely dependent on the value of CAUSE.CATEGORY, making the missingness ***MAR***.
<iframe src="assets/fig_6_missing_depends_empirical.html" width=800 height=600 frameBorder=0></iframe>

##### Missingness of Outage Duration(OUTAGE.DURATION) not depends on number of Customers Affected(CUSTOMERS.AFFECTED)
We further tested if the column ***OUTAGE.DURATION***'s missingness is depended on the values of column ***CUSTOMERS.AFFECTED***.
We used the absolute difference in mean as our test statistic to perform this permutation test.

This Distribution graph reveals the distribution of customers affected when duration is missing or not missing.
<iframe src="assets/fig_7_missing_notdepends.html" width=800 height=600 frameBorder=0></iframe>

Similarly, this is the resulting distribution of permutation average difference in mean versus observed average difference in mean. We can see that the red line, the observed, is in the middle of the blue distribution. This corresponds to a p-value equal to 0.696, which is way higher than the standard significance level of 0.05. The results show that our test is inconclusive, and we could not determine if OUTAGE.DURATION's missingness is dependent on the values of CUSTOMERS.AFFECTED.
<iframe src="assets/fig_8_missing_notdepends_empirical.html" width=800 height=600 frameBorder=0></iframe>


# Hypothesis Testing
**Null Hypothesis:** Severe weather related outage durations ***are*** randomly sampled from the population of outage duration. 

**Alternative Hypothesis:** Severe weather related outage durations ***are not*** randomly sampled from the population of outage duration. 

**Observation:** outage durations caused by severe weather

**Population:** all outage durations (from data)

**Test Statistic:** mean of sampled durations

**Sample Size:** number of outage durations that has been categorized as caused by severe weather

**Significance level:** 0.05

<iframe src="assets/fig_9_hypothesis_test.html" width=800 height=600 frameBorder=0></iframe>

**P-Value:** 0.0

**Conclusion:** Since the P-value is 0, which is lower than 0.05, we shall reject our null hypothsis. Our hypothesis test suggests that the mean outage duration caused by severe weather is significantly higher than the overall outage duration. This favors our alternative hypothsis: Severe weather related outage durations ***are not*** randomly sampled from the population of outage duration. This implies that the overall duration when caused by severe weather is significantly higher than the average duration length. 

After this hypothesis test, we now know that severe weather is a major contributor to the overall duration of a power outage, and this would require attention when assessing risk factors for each region to prevent future power outages. 