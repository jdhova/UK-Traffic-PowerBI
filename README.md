

## Introduction and Overview

***

This project is Aimed at re-creating the UK Road Traffic Dataset POWER BI Dash-Board.


### Tools and Languages:
	SSMS, Power BI and SQL




[Original-Project-Link](https://maps.dft.gov.uk/road-casualties/index.html)



### ------------------------------------CREATE SCHEMA----------------------------------------------


```
create schema rpt

create schema dim

create schema vw

create schema stg 

```

                                                   
### --------------------------------------- DIM Starts Here--------------------------------------------

  

#### --------------------------------Casualty Age_of_casualty  DIM------------------------------------

```
SELECT  Distinct [age_of_casualty] as 'Age_of_Casualty'
		,(case
		when age_of_casualty >= 0 and age_of_casualty <=15 Then '0-15'
		when age_of_casualty >= 16 and age_of_casualty <=24 Then '16-24'
		when age_of_casualty >= 25 and age_of_casualty <=59 Then '25-59'
		when age_of_casualty >= 60  Then '60+'
		ELSE 'Unknown'
		END) as 'Age_Bracket'
	--	into dim.Casualty_Age_Group
  FROM [UK_Staging].[stg].[casualites]
  ORDER BY 1

```
 
#### ---------------------------------Casualty_Sex_of_casualty  DIM------------------------------------

  ```
   SELECT Distinct [sex_of_casualty]
      ,(Case
			WHEN sex_of_casualty = 1 THEN 'Male'
			WHEN sex_of_casualty = 2 THEN 'Female'
			ELSE 'Unknown'
	  END) as 'gender'
Into dim.Casualty_Sex
FROM [UK_Staging].[stg].[casualites]
order by 1
```



#### --------------------------------Casualty_Class_of_casualty  DIM------------------------------------

```
SELECT Distinct [casualty_class]
	,(Case 
		When casualty_class = 1 Then 'Driver or rider'
		When casualty_class = 2 Then 'Passenger'
		When casualty_class = 3 Then 'Pedestrian'
		Else 'Unknown'
	  
	End) as Casualty_Class_Label
Into dim.Casualty_Class     
FROM [stg].[casualites]
Order by 1

```


#### -------------------------Road_User_Type DIM --------------------------------------------------------

```
SELECT distinct casualty_type as 'Road_Type'
     ,(CASE
        WHEN casualty_type = 0 THEN 'Pedestrian'
        WHEN casualty_type = 1 THEN 'Pedal Cyclist'
        WHEN casualty_type IN (8, 9, 10) THEN 'Car Occupant'
        WHEN casualty_type = 11 THEN 'Bus Occupant'
        WHEN casualty_type = 19 THEN 'Van Occupant'
        WHEN casualty_type IN (2,3,4,5,23,97)  THEN 'Motor Cyclist'
        WHEN casualty_type IN (20, 21) THEN 'HGV Occupant'
        WHEN casualty_type IN (16,17,18,22,90,98) THEN 'Other Veh Occupant'
        Else 'Unknown'
     END) as 'Road Type User'
INTO dim.Road_User_Type
FROM stg.casualites
ORDER BY 1

```

#### -------------------------Severity_level_Type DIM --------------------------------------------------------

```
SELECT Distinct [accident_severity]
	,(Case
		When accident_severity = 1 then 'Fatal'
		When accident_severity = 2 then 'Serious'
		When accident_severity = 3 then 'Slight'
		Else 'Unknown'
	End) as severity_level
Into dim.severity_level
FROM [stg].[accidents]
order by 1

```

#### -------------------------Accidents DIM ------------------------------------------------------------------

```
SELECT [accident_index]
      ,[accident_year]
      ,[accident_reference]
      ,[police_force]
      ,[accident_severity] 
      ,[acc_date]
      ,[day_of_week]
      ,[time]  
      ,[road_type]
      ,[speed_limit] 
      ,[light_conditions]
      ,[weather_conditions]
      ,[road_surface_conditions]
      ,[special_conditions_at_site]
      ,[carriageway_hazards]
      ,[urban_or_rural_area]   
	  ,( Case
		when first_road_class in (1,2) Then '101' 
		when urban_or_rural_area = 1 and first_road_class not in (1,2) Then '102' 
		when urban_or_rural_area = 2 and first_road_class not in (1,2) Then '103'
		Else '-99'
	   End) as 'calc_road_type'
	 into dim. accidents
  FROM [UK_Staging].[stg].[accidents]

 ``` 
 
  
  
#### --------------------------Road_Type DIM ----------------------------------------------------------------

```

SELECT  Distinct [calc_road_type]
		,( Case
		when calc_road_type = 101  Then 'Motorway'
		when calc_road_type  = 102 Then 'Urban'
		when calc_road_type= 103 Then 'Rural'
		Else 'Unknown'
	   End) as 'road_type'
	into dim.Road_Type
  FROM [UK_Staging].[dim].[accidents]
  
  ```
  
  #### -------------------------Light_Condition_Label DIM --------------------------------------------------------


``` 
SELECT distinct [light_conditions] as 'Light_Conditions'
		,(case 
		when light_conditions = 1 then 'DayLight'
		when light_conditions = 4 then 'Darkness-Lights-Lit'
		when light_conditions = 5 then 'Darkness-Lights-UnLit'
		when light_conditions = 6 then 'Darkness-No-Lighting'
		when light_conditions = 7 then 'Darkness-Light-Unknown'
		when light_conditions = -1 then 'Missing data or out of range'
		else 'Unknown'
	END ) as 'Light_Condition_Label'
		INTO dim.Light_Conditions
      FROM [UK_Staging].[dim].[accidents]
  


 ``` 

#### -----------------Speed_Limit DIM ---------------------------------------------------------------------------------

```
    select DISTINCT [speed_limit] as 'Speed_Limit'
	,(
	case 
		when speed_limit  < 0  then 'Car Parked'
		when speed_limit  >= 2 and  speed_limit <= 20 then '20'
		when speed_limit  >= 21 and  speed_limit <= 30 then '30'
		when speed_limit  >= 31 and  speed_limit <= 40 then '40'
		when speed_limit  >= 41 and  speed_limit <= 50 then '50'
		when speed_limit  >= 51 and  speed_limit <= 60 then '60'
		when speed_limit  >= 51 and  speed_limit <= 70 then '70'
		else 'Over Limit'
	end) as 'Speed_Labels'
	--INTO dim.Speed_Limit
	  FROM [UK_Staging].[dim].[accidents]
	  ORDER BY 1
```	  
	
	 
  #### -------------------------Day_Of_The_Week DIM --------------------------------------------------------	 
  ```
	  GO
SELECT distinct day_of_week as 'Day_Of_The_Week'
	,(case
		when day_of_week =  1 then 'Sunday'
		when day_of_week =  2 then 'Monday'
		when day_of_week =  3 then 'Tuesday'
		when day_of_week =  4 then 'Wednesday'
		when day_of_week =  5 then 'Thursday'
		when day_of_week =  6 then 'Friday'
		when day_of_week =  7 then 'Saturday'

	END) as 'Day of the Week Label'
    INTO dim.Day_Of_The_Week
  FROM [UK_Staging].[dim].[accidents]

 ``` 
 


#### -------------------------------Weather_Conditions_Condition_Label DIM ------------------------------------------
	


 ```

  SELECT distinct [weather_conditions] as 'Weather_Conditions'
	,(CASE
		when weather_conditions = 1 then 'Fine and no High Winds'
		when weather_conditions = 2 then 'Raining no High Winds'
		when weather_conditions = 3 then 'Snowing no High Winds'
		when weather_conditions = 4 then 'Fine and no High Winds'
		when weather_conditions = 5 then 'Raining plus High Winds'
		when weather_conditions = 6 then 'Snowing plus High Winds'
		when weather_conditions = 7 then 'Fog and Mist'
		when weather_conditions = 8 then 'Other'
		when weather_conditions = 9 then 'Unknown'
		when weather_conditions = -1 then 'Data Missing or not in range'
		END) AS 'Weather_Conditions_Label'
   INTO dim.Weather_Conditions
  FROM [UK_Staging].[stg].[accidents]
  ORDER BY 1



 ```

 ### ---------------------Views Starts Here---------------------------------------------------------------------------------
 
 
 
 ```
--CREATE SCHEMA vw
GO
CREATE OR ALTER VIEW vw.Casualty_Age_Group 
AS
SELECT * FROM dim.Casualty_Age_Group
GO 

CREATE OR ALTER VIEW vw.Casualty_Class 
AS
SELECT * FROM dim.Casualty_Class
GO

CREATE OR ALTER VIEW vw.Casualty_Sex 
AS
SELECT * FROM dim.Casualty_Sex

GO


CREATE OR ALTER VIEW vw.Day_Of_The_Week
AS
SELECT * FROM dim.Day_Of_The_Week

GO

CREATE OR ALTER VIEW vw.Light_Conditions
AS
SELECT * FROM dim.Light_Conditions

GO

CREATE OR ALTER VIEW vw.Local_Highway_Authority
AS
SELECT * FROM dim.Local_Highway_Authority

GO


CREATE OR ALTER VIEW vw.Police_Force_Area
AS
SELECT * FROM dim.Police_Force_Area

GO

CREATE OR ALTER VIEW vw.Road_Type
AS
SELECT * FROM dim.Road_Type

GO 

CREATE OR ALTER VIEW vw.Road_User_Type
AS
SELECT * FROM dim.Road_User_Type

GO

CREATE OR ALTER VIEW vw.Severity_Level
AS
SELECT * FROM dim.Severity_Level

GO


CREATE OR ALTER VIEW vw.Speed_Limit
AS
SELECT * FROM dim.Speed_Limit

GO

CREATE OR ALTER VIEW vw.Weather_Conditions
AS
SELECT * FROM dim.Weather_Conditions

GO

CREATE OR ALTER VIEW vw.fAccidents
AS
SELECT * FROM [dim].[accidents]

GO

CREATE OR ALTER VIEW vw.fCasualites
AS
SELECT * FROM stg.casualites


```



### ----------------------------------------------dim. Accidents -----------------------------------------------------------------

```

SELECT  [accident_index]
      ,[accident_year]
      ,[accident_reference]
      ,[location_easting_osgr]
      ,[location_northing_osgr]
      ,[longitude]
      ,[latitude]
      ,[police_force]
      ,[accident_severity]
      ,[number_of_vehicles]
      ,[number_of_casualties]
      ,[acc_date]
      ,[day_of_week]
      ,[time]
      ,[local_authority_district]
      ,[local_authority_ons_district]
      ,[local_authority_highway]
      ,[first_road_class]
      ,[first_road_number]
      ,[road_type]
      ,[speed_limit]
      ,[junction_detail]
      ,[junction_control]
      ,[second_road_class]
      ,[second_road_number]
      ,[pedestrian_crossing_human_control]
      ,[pedestrian_crossing_physical_facilities]
      ,[light_conditions]
      ,[weather_conditions]
      ,[road_surface_conditions]
      ,[special_conditions_at_site]
      ,[carriageway_hazards]
      ,[urban_or_rural_area]
      ,[did_police_officer_attend_scene_of_accident]
      ,[trunk_road_flag]
      ,[lsoa_of_accident_location]
	  ,( Case
		when first_road_class in (1,2) Then '101' 
		when urban_or_rural_area = 1 and first_road_class not in (1,2) Then '102' 
		when urban_or_rural_area = 2 and first_road_class not in (1,2) Then '103'
		Else '-99'
	   End) as 'calc_road_type'
	   into dim. accidents
  FROM [UK_Staging].[stg].[accidents]
  

```


#### -----------------------------------------Calendar-------------------------------------------------------------------

```
DECLARE @StartDate  date = '20180101';
DECLARE @CutoffDate date = DATEADD(DAY, -1, DATEADD(YEAR, 11, @StartDate));

;WITH seq(n) AS 
(
  SELECT 0 UNION ALL SELECT n + 1 FROM seq
  WHERE n < DATEDIFF(DAY, @StartDate, @CutoffDate)
),
d(d) AS 
(
  SELECT DATEADD(DAY, n, @StartDate) FROM seq
),
src AS
(
  SELECT
   bkDateKey   = CAST(REPLACE(CONVERT(varchar(10), d),'-','') as INT),
    Date         = CONVERT(date, d),
	--DateKey		   = CAST(REPLACE(CONVERT(varchar(10), d),'-','') as INT),
    DayofMonth   = DATEPART(DAY,       d),
    DayName      = DATENAME(WEEKDAY,   d),
    WeekOfYear    = DATEPART(WEEK,      d),
    ISOWeek      = DATEPART(ISO_WEEK,  d),
    DayOfWeek    = DATEPART(WEEKDAY,   d),
    Month        = DATEPART(MONTH,     d),
    MonthName    = DATENAME(MONTH,     d),
	MonthAbbrev  = LEFT(DATENAME(MONTH, d),3),
    Quarter      = DATEPART(Quarter,   d),
	Qtr          =(CASE
						WHEN DATEPART(Quarter,   d) = 1 THEN 'Q1'
						WHEN DATEPART(Quarter,   d) = 2 THEN 'Q2'
						WHEN DATEPART(Quarter,   d) = 3 THEN 'Q3'
						WHEN DATEPART(Quarter,   d) = 4 THEN 'Q4'
						ELSE 'Err'
					END
					),
    Year         = DATEPART(YEAR,      d),
    FirstOfMonth = DATEFROMPARTS(YEAR(d), MONTH(d), 1),
    LastOfYear   = DATEFROMPARTS(YEAR(d), 12, 31),
    DayOfYear    = DATEPART(DAYOFYEAR, d)
  FROM d
)
SELECT * 
INTO dim.Calendar
FROM src
  ORDER BY Date
  OPTION (MAXRECURSION 0);
 ```
 
  #### ---------------------------------------View Starts here-------------------------------------------------------------

  
```

SELECT COUNT (*)
from stg.casualites fc
inner join [vw].[fAccidents] fa
ON fc.accident_index = fa.accident_index 
inner join [dim].[Calendar] cal
on cal.[Date] = fa.[acc_date]
inner join [dim].[Casualty_Age_Group] age
on fc.age_of_casualty= age.[Age_of_Casualty]
inner join [dim].[Local_Highway_Authority] llha
on llha.[code] = fa.[local_authority_highway]
inner join [dim].[Police_Force_Areas] pol
on pol.[code] = fa.[police_force]
inner join [vw].[Speed_Limit] speed
on fa.[speed_limit] = speed.[Speed_Limit]
join [dim].[Casualty_Sex] sx
on fc.sex_of_casualty = sx.[sex_of_casualty]
inner join [dim].[Road_User_Type]  rut
on fc.casualty_type = rut.[Road_Type] 
inner join [dim].[severity_level] sev
on fc.casualty_severity =sev.[accident_severity]
inner join [dim].[Casualty_Class] class
on fc.casualty_class = class.[casualty_class]
inner join [dim].[Road_Type] rtp
on rtp.[calc_road_type] = fa.[calc_road_type]

```





===================================================================================================================================================




```

CREATE OR ALTER VIEW rpt.Traffic_UK
AS
select fa.[accident_year]
	,sx.[sex_of_casualty] as 'Sex_of_casualty'
	,rut.[Road_Type]
	,rut.[Road Type User]
	,llha.[City_Label] as 'Local_Auth_Highway'
	,pol.[label] as 'Police_Force_Area'
	,sev.[severity_level]
	,rtp.[road_type] as 'Road-type'
	,speed.[Speed_Limit]
	,sx.[gender] as 'Casualty_Sex-gender'
	,age.[Age_Bracket]
	,class.[Casualty_Class_Label]
--class.[casualty_class]
	,1 as 'Casualties'
from stg.casualites fc
inner join [vw].[fAccidents] fa
ON fc.accident_index = fa.accident_index 
inner join [dim].[Calendar] cal
on cal.[Date] = fa.[acc_date]
inner join [dim].[Casualty_Age_Group] age
on fc.age_of_casualty= age.[Age_of_Casualty]
inner join [dim].[Local_Highway_Authority] llha
on llha.[code] = fa.[local_authority_highway]
inner join [dim].[Police_Force_Areas] pol
on pol.[code] = fa.[police_force]
inner join [vw].[Speed_Limit] speed
on fa.[speed_limit] = speed.[Speed_Limit]
join [dim].[Casualty_Sex] sx
on fc.sex_of_casualty = sx.[sex_of_casualty]
inner join [dim].[Road_User_Type]  rut
on fc.casualty_type = rut.[Road_Type] 
inner join [dim].[severity_level] sev
on fc.casualty_severity =sev.[accident_severity]
inner join [dim].[Casualty_Class] class
on fc.casualty_class = class.[casualty_class]
inner join [dim].[Road_Type] rtp
on rtp.[calc_road_type] = fa.[calc_road_type]

```