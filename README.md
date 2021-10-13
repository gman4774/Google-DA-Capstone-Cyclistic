# Cyclistic Case Study
The following is a more in depth breakdown of my case study for the Google Data Analytics capstone project.
<br>
I was assigned to answer the question, ` How do annual members and casual riders use Cyclistic bikes differently? `

## 1. Data Organization
Previous 12 months of trip data were downloaded from [here](https://divvy-tripdata.s3.amazonaws.com/index.html).
<br />
Each month had a seperate file, so the data would have to be combined. First I import each file into its own variable:
<br />
```r
trip_202010 <- read_csv("202010-divvy-tripdata.csv")
... 
trip_202109 <- read_csv("202109-divvy-tripdata.csv")
```
Before I could combine all the data I needed to verify the column names to make sure they match:
```r
colnames(trip_202010)
...
col_names(trip_202109)
```
Once everything was matching, I combined all 12 datasets together:
```r
all_trips <- bind_rows(trip_202010, trip_202011, trip_202012, trip_202101, trip_202102, trip_202103, trip_202104, trip_202105, trip_202106, trip_202107, trip_202108, trip_202109)
```

Once combined, I inspected the new data set to make sure there were no issues.
```r
col_names(all_trips) #Verify column names and no new columns were created
str(all_trips) #Checking column data types


```
Next I inspected the member_casual column to verify there are only two types of inputs, member and casual:
```r
table(all_trips$member_casual)
```
`member_casual` did not need any further attention since they were correctly labeled.
<br>
I added columns for date, month day and year for each ride:
```r
all_trips$date <- as.Date(all_trips$started_at)
all_trips$month <- format(as.Date(all_trips$date), "%m")
all_trips$day <- format(as.Date(all_trips$date), "%d")
all_trips$year <- format(as.Date(all_trips$date), "%Y")
all_trips$day_of_week <- format(as.Date(all_trips$date), "%A")
```
Now I added a `ride_length` column to calculate each trips duration in seconds:
```r
all_trips$ride_length <- difftime(all_trips$ended_at,all_trips$started_at)
```
`ride_length` needed to be converted to numeric so calculations can be done with it:
```r
all_trips$ride_length <- as.numeric(as.character(all_trips$ride_length))
is_numeric(all_trips$ride_length)
[1] TRUE
```
The data has trips that are in the negative and trips that were bike maintenance by the company, so these two issues needed to be removed:
```r
all_trips_v2 <- all_trips[!(all_trips$start_station_name=="HQ QR" | all_trips$ride_length<0),]
```
The data is now ready for analysis.

## 2. Analysis


I began my analysis with getting a summary on the `ride_length` column:
```r
summary(all_trips_v2$ride_length)

```
      Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
      0     426     756    1321    1367 3356649
Now I needed to compare the two diffrent customers average ride time:
```r
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = mean)
```
      all_trips_v2$member_casual all_trips_v2$ride_length
    1                     casual                1894.7615
    2                     member                 833.3881
Now I'm starting to notice the diffrences with the members.
<br>
I check the minimum and maximum `ride_length` for each member type, but do not see anything significant:
```r 
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = min)

```
      all_trips_v2$member_casual all_trips_v2$ride_length
    1                     casual                        0
    2                     member                        0
```r
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = max)
```
      all_trips_v2$member_casual all_trips_v2$ride_length
    1                     casual                  3356649
    2                     member                   573467
<br>
I want to see the average ride length by each day but the days are out of order so I reorganize the data:

```r
all_trips_v2$day_of_week <- ordered(all_trips_v2$day_of_week, levels=c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"))
```
Now I am able to get the average `ride_length` for each member type on each day in order:

```r
avg_trips <- aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN = mean)
```
       all_trips_v2$member_casual all_trips_v2$day_of_week all_trips_v2$ride_length
    1                      casual                   Sunday                2203.1744
    2                      member                   Sunday                 941.6029
    3                      casual                   Monday                1874.5546
    4                      member                   Monday                 802.5074
    5                      casual                  Tuesday                1693.9814
    6                      member                  Tuesday                 785.6001
    7                      casual                Wednesday                1649.4812
    8                      member                Wednesday                 790.5415
    9                      casual                 Thursday                1624.4455
    10                     member                 Thursday                 783.2561
    11                     casual                   Friday                1809.3572
    12                     member                   Friday                 823.9839
    13                     casual                 Saturday                2057.0019
    14                     member                 Saturday                 925.1289

I notice casual members tend to ride for longer, where on average annual members ride for less than 20 minutes.
<br>
I decide to get the number of rides by rider type into a dataframe:

```r
num_trips <- all_trips_v2 %>%
    mutate(weekday = wday(started_at, label = TRUE)) %>%
    group_by(member_casual, weekday) %>%
    summarise(number_of_rides = n()
              ,average_duration = mean(ride_length)) %>%
    arrange(member_casual, weekday)
```
This give me some interesting results:
```r
   member_casual weekday number_of_rides average_duration
   <chr>         <ord>             <int>            <dbl>
 1 casual        Sun              444326            2203.
 2 casual        Mon              266087            1875.
 3 casual        Tue              251185            1694.
 4 casual        Wed              256878            1649.
 5 casual        Thu              274258            1624.
 6 casual        Fri              338704            1809.
 7 casual        Sat              522953            2057.
 8 member        Sun              344841             942.
 9 member        Mon              375433             803.
10 member        Tue              406440             786.
11 member        Wed              422341             791.
12 member        Thu              420071             783.
13 member        Fri              404994             824.
14 member        Sat              399638             925.
```

<br>
From this dataset I can see that the number of casual riders increases on the weekends(Saturday to Sunday), 
<br>
and the number of annual members is the opposite and increases during the work week(Monday through Friday).
<br>
This leads me to believe that annual members are more likely to use the bikes for their cummutes to and from work.
<br>
The data also is showing casual users tend to use the bikes for recreational activities, causing longer trip times.
<br>
<br>
With the annual member profile figured out, I apply those parameters to casual users and see how many fit the profile of an annual member:

```r
casual_only <- filter(all_trips_v2, member_casual=='casual')
casual_mempro <- filter(casual_only, ride_length<1200)#1200 seconds = 20 min
casual_mempro <- filter(casual_mempro, day_of_week!="Saturday")
casual_mempro <- filter(casual_mempro, day_of_week!="Sunday")
```

The data with only casual users had 2354391 rides total, of which after applying the annual users profile, 862322 matched.<br>
`862322/2354391`
<br>
In the last 12 months `36.6%` of casual users fit the annual members profile.
<br>
I decide to export the data and prep the graphs in Tableau.
<br>
First I backup all the cleaned data:

```r
write.csv(all_trips_v2, file='all_trips_v2.csv')

```

Then I export the number and average sets I created earlier:
```r
write.csv(num_trips, file='num_trips.csv')
write.csv(avg_trips, file='avg_trips.csv')

```


## 3. Tableau

I created the following charts in Tableau to better visually represent the data and my results.
<br>

![Number of rides per day per member type](https://raw.githubusercontent.com/gman4774/Google-DA-Capstone-Cyclistic/main/Sheet1.png?token=AUHBCPD3P6KFGK74GBFO2QTBM4L2W "number")

<br>

![Average ride length](https://raw.githubusercontent.com/gman4774/Google-DA-Capstone-Cyclistic/main/Sheet2.png?token=AUHBCPDZANT2JPGV5FO5J2TBM4L7W "average")

<br>

![Member profile applied to casual members](https://raw.githubusercontent.com/gman4774/Google-DA-Capstone-Cyclistic/b0b52335968d3df8d151e8aa6deb6b591c88a02a/profile.svg?token=AUHBCPE24CHZC5D4BQ3LBZ3BM4L7E "profile")

<br>

## 4. Presentation

<br>

After I had everything I needed, I put it together into a short powerpoint presentation [HERE](https://github.com/gman4774/Google-DA-Capstone-Cyclistic/blob/main/presentation.odp?raw=true).
