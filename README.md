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


