# Business Intelligence / Data Analyst Challenge

This is an open analytics challenge based on real data given to all Freckle IoT Data Analyst applicants. The work is easy but we want to see your SQL, Data Modelling, and statistics skills in action. You can complete this at any point in time prior to your on-site interview. 

Each entry in this data set is a "location-event", it is the raw feed from a . In this mini-assignment, we want to build a regular report of our ingested data so we understand when it is coming in and where from. 

**Data Dictionary:**

app_id - The identifier of the application the data event came from. There are many app_ids in 
data_source - The unique name of data source. There is a _mutually exclusive_ set of app_ids for each data source. 
lat - lattitude of the event
lng - longitude of the event
event_date - timestamp of the event
user_id - the unique user that generated the event. 

**Instructions:**

1. Fork this repo with your own id for our review.
2. Download the dataset here: https://s3.amazonaws.com/freckle-dataeng-challenge/bichallenge-loc-event-sample.csv.gz
3. Show the average number of events per user-id
--Answer: 1.2043

--Solution:
select avg(convert(float,events)) as avg_events from 
(select idfa,count(distinct location_event_time) as events from dbo.freckle_sample 
where idfa is not null
group by idfa ) b;

4. Construct a data model that normalizes the data into a fact table with the following hierarchical dimensions:
  a) Day :: Hour
  b) Data Source :: App ID--Add a column and parse out the date and times from the location_event_time column

--Solution:

--Parse out the date field using text functions
alter table dbo.FRECKLE_SAMPLE add datefinal varchar(20);

update dbo.FRECKLE_SAMPLE set datefinal=concat(substring(location_event_time,1,10),' ',SUBSTRING(location_event_time,12,8));

--Format the date so that it can be read properly

update dbo.FRECKLE_SAMPLE set datefinal=convert(datetime,datefinal);

--Add columns for the day and hour

alter table dbo.FRECKLE_SAMPLE add event_day date;
alter table dbo.FRECKLE_SAMPLE add event_hour int;

--Set the day column to be the event day and the hour column to be event hour

update dbo.FRECKLE_SAMPLE set event_day=convert(date,datefinal);
update dbo.FRECKLE_SAMPLE set event_hour=datepart(hour,datefinal);

--Check that everything worked
select * from dbo.FRECKLE_SAMPLE;

--Pull together source_data and AppID for Source::AppID heirarchy

alter table dbo.FRECKLE_SAMPLE add DataSource_AppID varchar(100);
update dbo.FRECKLE_SAMPLE set DataSource_AppID=concat(source_data,'_',app_id);

--Pull together day and hour columns for Day::Hour heirarchy

alter table dbo.FRECKLE_SAMPLE add Date_Hour varchar(100);
update dbo.FRECKLE_SAMPLE set Date_Hour=concat(event_day,'_',event_hour);

--Create a table that summarizes the number of IDs with the specified heirarchies

select DataSource_AppID,Date_Hour,count(distinct idfa) as UniqueIDs into FRECKLE_FACT from dbo.FRECKLE_SAMPLE 
group by DataSource_AppID,Date_Hour;

--Check work

select * from FRECKLE_FACT;


5. Using the data model constructed in 4, determine how many unique user-ids are represented in hour 1-2pm (13:00-14:00) for app_id 17, data_source twine

--Answer: 2,882 Unique IDs

--Solution:
select * from FRECKLE_FACT where lower(DataSource_AppID) like 'twine_17' and date_hour like'%_13%';

6. Determine what type of statistical distribution the population of events/idfa. Events per IDFA is determined by counting the total number of events per unique IDFA within the entire population (the data set already bounded at 24 hours). 

--Answer: Poisson distribution with Lambda=1.204
--Solution:

--Create a table that shows the number of events per unique ID

select idfa,count(distinct location_event_time) as events into EventsPerID from dbo.freckle_sample 
where idfa is not null
group by idfa; 

--Check the distribution of values for symmetry
select events,count(*) from EventsPerID group by events order by events;
--Events per IDFA is positively skewed

--Calculate the average and standard deviation

select avg(convert(float,events)) from EventsPerID;--1.2043

select stdev(convert(float,events)) from EventsPerID;--0.8165

--Since the number of events per ID is discrete, right skewed and has a low mean the best distribution is Poisson

--Goodness of fit tests could be performed using R or another statistical software package to further validate this model


__Please show the SQL (including DDL & DML) for your answers__
__Please use the entire dataset provided__

Please complete as much of the assignment as you have time for. How long you had time to spend on the challenge and your experience will be considered. Have some fun with it!
