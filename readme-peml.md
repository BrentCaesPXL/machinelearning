# PE ML

## Project goal

The goal of this project is to predict Citi Bike demand on an hourly level.

In phase 1, we focus on data selection, cleaning, feature preparation, and exploratory data analysis.

In phase 2, we will use the insights from this phase to train and evaluate forecasting models on 2019 data.

## Data selection

We selected data from 2016, 2017, 2018, and 2019. The data from 2016 to 2018 will be used for exploratory analysis, model training, and validation, while 2019 will be reserved for comparison and final evaluation.

We decided to exclude 2020 and 2021 because the COVID-19 pandemic had a major impact on mobility patterns. During that period, Citi Bike usage dropped significantly and no longer reflected normal travel behavior. Including these years would likely distort the model and reduce its ability to predict demand under regular conditions.

We also chose not to include 2022 because the dataset contains a large number of missing values (NaN values), which would reduce the reliability and consistency of the analysis.

## Preprocessing

First, the weather dataset was loaded and filtered to include only the years 2016 to 2018. The timestamp column was converted to a datetime format, after which the weather variables were renamed to shorter and clearer feature names, such as temperature, precipitation, rain, cloudcover, and windspeed. This made the dataset easier to use during the merging and analysis stages.

Next, the Citi Bike trip files were processed. Because the raw Citi Bike data consists of many large CSV files, the data was handled in a memory-efficient chunk-by-chunk approach rather than loading everything into memory at once. Only the columns relevant for this phase were kept: trip duration, start and stop times, start and end station IDs, and user type. Since different files used slightly different naming conventions, the column names were standardized first.

After loading the relevant fields, both `start_time` and `stop_time` were converted to datetime format using a robust parsing strategy. Trip duration was converted from seconds to minutes, resulting in a new variable called `trip_duration_min`. Several cleaning filters were then applied. Rows with missing timestamps or missing trip duration values were removed. In addition, trips with non-positive durations and trips longer than 180 minutes were excluded, because such values are likely to represent outliers, registration errors, or exceptional rides that are not representative of normal Citi Bike usage.

The `user_type` column was also cleaned. Missing values were replaced by the category `Unknown`, and textual values were standardized to a consistent format. After this, each trip was assigned to an hourly timestamp by flooring the start time to the hour, creating the variable `datetime_hour`.

The cleaned trip data was then aggregated to hourly level. For each hour, the number of trips was counted, the average trip duration was calculated, and several derived counts were created, including the number of subscriber trips, customer trips, unknown-user trips, and the number of unique start and end stations. This hourly aggregation reduced the raw trip-level data to a compact and structured format that is much more suitable for analysis and machine learning.

Afterward, the hourly Citi Bike data was merged with the hourly weather data using `datetime_hour` as the key. Remaining missing values in the weather columns were filled using interpolation combined with backward fill and forward fill, ensuring a complete and continuous time series.

The result of this preprocessing pipeline is one final merged dataset at hourly level, containing cleaned Citi Bike demand variables, cleaned weather variables, and derived temporal features. This dataset forms the basis for the exploratory analysis in phase 1 and for the forecasting models in phase 2.

## Number of Citi Bike trips per hour
<img width="855" height="386" alt="Screenshot 2026-03-31 171809" src="https://github.com/user-attachments/assets/f0f736cb-5aa7-4e5d-bd97-e049af6f1105" />

This plot shows a clear upward trend from 2016 to 2018, combined with strong seasonal variation. Trip counts are much lower in winter and much higher in spring, summer, and early autumn, while short-term fluctuations suggest the influence of commuting patterns, weekdays, and weather.

Overall, Citi Bike demand follows a structured pattern rather than random variation.

## Average number of trips by hour of the day
<img width="855" height="449" alt="Screenshot 2026-03-31 172506" src="https://github.com/user-attachments/assets/5ee476bf-2e81-4f5f-9528-3e6c2c3b8e57" />

Demand is lowest during the night, rises sharply in the early morning, and peaks around 8 AM and again around 5–6 PM. These two peaks strongly suggest commuting behavior, while daytime demand remains relatively high beyond rush hours.

This makes `hour` one of the strongest predictors of demand.

## Average Number of Trips by Day of the Week
<img width="859" height="485" alt="Screenshot 2026-03-31 172751" src="https://github.com/user-attachments/assets/e8f9741d-6be8-490a-81f0-108f53b59cfa" />

Trip counts are highest during the middle of the workweek, especially on Wednesday and Thursday, and lower on weekends, with Sunday showing the lowest demand. This indicates that Citi Bike is mainly used for weekday mobility and commuting, while weekend usage is more likely linked to leisure or non-work travel.

The day of the week is therefore an important predictor of demand.

## Average Number of Trips by Month
<img width="861" height="455" alt="Screenshot 2026-03-31 173020" src="https://github.com/user-attachments/assets/dd9b3ee1-999e-450a-9038-b05b2cf76933" />

This plot reveals a strong annual cycle: demand is lowest in winter, increases through spring, and peaks in late summer or early autumn. This pattern shows that Citi Bike usage is strongly influenced by seasonality and weather conditions.

As a result, `month` is a valuable feature for forecasting.

## Number of trips: rain vs no rain
<img width="862" height="452" alt="Screenshot 2026-03-31 173307" src="https://github.com/user-attachments/assets/e0f6991f-84a1-48a2-a80a-70ec4f1a7ac0" />

The median number of trips is lower during rainy hours, and the whole rainy distribution is shifted slightly downward. However, the overlap between rainy and dry conditions shows that rain is not the only factor affecting demand.

## Difference in average trips by hour: Weekdays and Weekends 

<img width="618" height="322" alt="image" src="https://github.com/user-attachments/assets/c46e9a60-5ea4-47a7-b683-f187ffc24320" />

This graph clearly shows the difference between activity during the week and during the weekends. Looking at the data from during the week (the blue graph) we can clearly visualize the first peak around 8 in the morning, when most people go to work. After this peak the activity drops to a really low average before then climbing to the second peak of the day. This peak takes place around 5 in the afternoon when most people are done with work and are heading home. During the weekend (the orange graph) the trend is completely different. The amount of activity in the morning is really minimal and only starts climbing when we reach noon. After this, the trend holds for a few hours before going down. This can be caused by people sleeping in and only going for a bike trip in the afternoon when the temperature is at its highest, the trend really only starts to go down when the sun goes down in the evening. Another explanation for this behavior is tourism.

## Difference in average trips during weekdays: Rain and No Rain 

<img width="607" height="305" alt="image" src="https://github.com/user-attachments/assets/079b7353-df3d-4a13-acec-bae088ad170b" />

In this graph we visualize the impact of weather on the activity during the week. We can clearly see a drop of activity when its raining but what stands out in this graph is the fact that the peaks remain intact. We can conclude here that work traffic plays a big part in this dataset.

## Difference in average trips during weekend: Rain and No Rain 

<img width="802" height="390" alt="image" src="https://github.com/user-attachments/assets/ec1b98f2-31e1-440a-933b-cf9653b3b4f8" />

In this graph we visualise the impact of weather on the activity during the weekend. If we compare this graph against the one from during the week, we can see that weather has a bigger impact on activity during the weekends. This can be caused by the fact that less people are obliged to go to work during these days and tend to stay at home but another reason can be that tourism is at a lower amount when it rains. People tend to stay home or search for inside activities during these times.

Rain is therefore a useful predictor, but it should be combined with time-based and seasonal features.

## Temperature vs Number of Trips
<img width="861" height="449" alt="Screenshot 2026-03-31 174439" src="https://github.com/user-attachments/assets/70179a66-2146-4e0c-a108-3ca9d65c7886" />

This plot shows a clear positive relationship between temperature and bike demand. Warmer conditions are associated with more trips, while very high trip counts are rare in cold weather.

Since the spread remains wide, temperature is important but should be used together with other predictors.

## Distribution of Trip Duration
<img width="848" height="533" alt="Screenshot 2026-03-31 175248" src="https://github.com/user-attachments/assets/796e3ebe-b014-43b2-b68b-0510e575b3c7" />

Trip durations are strongly right-skewed: most rides are short, especially around 5 to 15 minutes, while long trips are relatively rare. This suggests that Citi Bike is mainly used for short urban travel rather than long-distance trips.

It also shows that trip duration is not normally distributed, which matters for later modeling.

## Number of Trips by User Type
<img width="856" height="587" alt="Screenshot 2026-03-31 175540" src="https://github.com/user-attachments/assets/9249f43f-d112-4ff3-a5ff-a3a0361f21e4" />

Subscribers make far more trips than Customers, showing that Citi Bike is mainly used by regular rather than occasional riders. This suggests that the system primarily serves recurring daily mobility needs such as commuting.

User type therefore provides useful insight into overall demand patterns.

## Number of Trips by Age Category
<img width="854" height="575" alt="Screenshot 2026-03-31 180127" src="https://github.com/user-attachments/assets/c0293a8b-5660-41b1-a879-c8bf679d6eb6" />

Citi Bike usage is dominated by Millennials, followed by Gen X, while Boomers, Unknown, and Gen Z represent much smaller shares. This suggests that the system is used mainly by working-age adults.

That pattern fits well with the strong commuting signals found in the other visualizations.

## Correlation Matrix
<img width="617" height="549" alt="image" src="https://github.com/user-attachments/assets/12bb0b25-a98f-4e7c-8615-35bcfbdef333" />

The strongest positive correlations with `trip_count` are `hour`, `temperature`, and `month`, showing that time of day, weather, and season all matter. Rain-related variables have only weak negative correlations, while `precipitation` and `rain` are highly correlated with each other, indicating redundancy.

Overall, the matrix confirms that bike demand is influenced by multiple factors rather than one dominant variable.

## Hourly trip volume heatmap
<img width="1122" height="538" alt="Screenshot 2026-03-31 204739" src="https://github.com/user-attachments/assets/b588b9f7-2302-4227-91e7-ad2efef2f652" />

The heatmap highlights a strong contrast between weekdays and weekends. Weekdays show clear morning and evening commuting peaks, while weekends have a broader daytime pattern without a sharp morning peak.

This confirms that both `hour` and `day_of_week` are highly relevant for forecasting demand.

## Weekly Trip Volume Heatmap
<img width="1214" height="692" alt="Screenshot 2026-03-31 205002" src="https://github.com/user-attachments/assets/3e81a645-2879-4c3e-96be-746c0f9996d1" />

This heatmap shows both long-term growth and strong seasonality. Weekly demand is much lower in winter, much higher in warmer periods, and generally stronger in 2017 and 2018 than in 2016.

These patterns suggest that both week-level seasonal timing and year-over-year growth should be considered in the analysis.

## Top 10 most popular start and end stations
<img width="1731" height="760" alt="Screenshot 2026-03-31 205301" src="https://github.com/user-attachments/assets/0560b825-7265-4611-82e0-84811b716d1a" />

These charts show that usage is concentrated around a relatively small number of highly active stations. Pershing Square North stands out as the busiest station in both the start and end rankings, suggesting that it functions as a major hub in the network.

This highlights the importance of station location in understanding demand.

## Subscriber vs customer breakdown over time
<img width="1312" height="515" alt="Screenshot 2026-03-31 205739" src="https://github.com/user-attachments/assets/b2adfee6-3309-4a37-9c2a-736127f21cb5" />

Subscribers dominate total trip volume throughout the entire period, while Customers remain a much smaller group. Both groups show a clear seasonal pattern, but overall demand growth is driven mainly by Subscribers.

This suggests that Citi Bike demand is strongly shaped by recurring rider behavior, with a smaller but still relevant casual segment.

## Geographic density map of station usage
<img width="761" height="459" alt="Screenshot 2026-03-31 205931" src="https://github.com/user-attachments/assets/8c29d670-9132-4df9-ba3d-0273cd3d8df4" />

This map shows that station usage is spatially concentrated in Manhattan, especially in Midtown and Lower Manhattan. These areas appear as the strongest hotspots, suggesting that they function as major mobility hubs.

This confirms that Citi Bike demand is clustered around a limited number of central urban zones rather than evenly distributed across the network.

## Explanation of how it was created

To create this map, both start stations and end stations were extracted from the dataset, together with their station IDs, names, and geographic coordinates. These records were combined so that the map reflects total station usage instead of only departures or only arrivals.

Next, the total number of occurrences per station was counted to create a `usage_count`. Finally, the station coordinates and their usage counts were visualized in an interactive heatmap using `folium` and the `HeatMap` plugin.

## Conclusion: phase 1

In phase 1, we cleaned, transformed, and explored the Citi Bike dataset in order to understand the main factors that drive bike demand and to prepare the data for forecasting. The results show that Citi Bike demand is not random, but follows clear and recurring temporal, seasonal, weather-related, and behavioral patterns.

The hourly and weekly analyses revealed strong growth between 2016 and 2018, combined with a clear seasonal cycle in which demand is lowest in winter and highest during the warmer months. In addition, the daily and weekly usage patterns showed that Citi Bike demand is strongly linked to urban mobility routines, with pronounced morning and evening peaks on weekdays and a different, more leisure-oriented profile during weekends.

The exploratory analysis also demonstrated that weather conditions influence demand in meaningful ways. Rain generally lowers the number of trips, while higher temperatures are associated with higher usage. At the same time, the correlation analysis confirmed that no single feature explains demand on its own. Instead, Citi Bike demand should be treated as a multi-factor problem in which time-based, seasonal, and weather-related variables all contribute useful information.

The additional station-based analyses further showed that usage is spatially concentrated around a limited number of major hubs, especially in Manhattan, while the rider breakdown confirmed that the system is dominated by subscribers and working-age users. This reinforces the interpretation that Citi Bike primarily functions as a practical, recurring transportation service rather than an occasional leisure-only system.

Overall, phase 1 provides a strong foundation for the next stage of the project. The data has been cleaned and structured at hourly level, the most relevant features have been identified, and the exploratory analysis has produced clear evidence of predictable demand patterns. This means the dataset is suitable for forecasting, and phase 2 can now focus on training, comparing, and evaluating machine learning models on 2019 data, in line with the project requirements.
