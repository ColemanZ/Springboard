
# Covid and Weather Predictor Capstone
*The world has been shook by the global pandemic caused by the COVID-19 virus. There were periods of time in the last few years where hospitals were overrun with patients and supplies were low. Many areas of the world completely shut down during the peaks of COVID-19 cases causing massive economic disruption. It is estimated that billions of people caught COVID-19 and millions of people died from COVID-19. This will be a predictor system to predict North Carolina's next day/week/month's confirmed COVID-19 cases using previous COVID-19 data and weather data.*
## 1. Data
Data was gathered from primarily two sources. The NOAA provides weather data across thousands of stations in the United States of America. The CSSE provides daily reports for countries and states across the world.

[COVID-19 Data](https://covid19.ncdhhs.gov/dashboard)

[North Carolina Weather Data](https://nceii.noaa.gov/cdo-web/api/v2/)

## 2. Method
Create a weighted ensemble of highest performing models to create a predictive model for future COVID-19 cases.
1. Gather weather and COVID-19 data
2. Clean data into a useable form
3. Create features to be used in model building
4. Build models 
5. Tune models
6. Ensemble models in a weighted voting regressor model
7. Determine highest performing model

## 3. Gather Weather and COVID-19 Data
[Data Wrangling Notebook](https://github.com/ColemanZ/CovidCapstoneProject/blob/main/%5BColeman%20Zimmerman%5D%20Capstone%202%20Data%20Wrangling.ipynb)
Data was gathered from two sources listed above. The COVID-19 data came in the form of a CSV file for each day. Using Glob, all CSV files in a folder were gathered and combined into one dataframe. The weather data came in the form of a JSON file and were called using an API one year at a time. The precipitation, temperature and date were gathered from the weather data and zipped into one dataframe.

* **Problem 1** Weather data missing:
    Weather data was missing the first three months (January, February, March) of every year no matter how the API was called or what station was called upon. 
* **Solution 1** Weather data downloaded:
    The NOAA allows an individual to download weather data in addition to calling it with an API. When downloaded this data was not missing. It was then read in as a CSV file and joined to the existing weather data.
    
## 4. Clean Data Into a Useable Form
[Exploratory Data Analysis Notebook](https://github.com/ColemanZ/CovidCapstoneProject/blob/main/%5BColeman%20Zimmerman%5D%20Capstone%202%20Data%20Wrangling.ipynb)
The data as presented needed some fairly heavy cleaning. Confirmed cases and deaths were given as a running total so they had to be converted into New Cases and New Deathes. Primarily only about half of the features had usable data. There was also a number of rows with all data missing. The data was also difficult to use in a daily format. 

* **Problem 1** Bad Features
    A number of features had issues. There were 6 features that only had one unique value so they provided no real value. Also the reporting in North Carolina changed over the course of the pandemic. Initially more items were reported than at the end. Things like Recovered, Actigve, Total Test Results and others ended up not being reported on halfway through the pandemic.
* **Solution 1** Drop bad features
    I decided that rather than only use the first half of the data, it would be better to use all the data and drop the features that weren't reported throughout the entire pandemic. Also all features with only one unique value were dropped.
    
* **Problem 2** Rows with empty values
    North Carolina decided to not report any new cases or deaths on Sunday or Monday as well as any holiday and missing various days in addition to this. Then on the next day that was reported they would provide the new cases and deaths for all the previous days.

* **Solution 2** Fill empty rows using next reported row
    A loop was made that would check how many days in a row were not reported. It would then take the new confirmed and new death cases from the subsequent date and fill the missing rows based on how many days it had been since reporting. 
  ![correlation_heat_map](https://user-images.githubusercontent.com/97986175/205494943-38458619-87f1-472e-a9e5-872b4a5164c1.png)
  
* **Problem 3** Daily data was volatile
    Since COVID-19 cases can jump up and down from day to day when trying to vizualize on a daily basis it was very difficult to gleam anything. 

* **Solution 3** Convert data to a weekly and monthly format
    Data was converted into weekly and monthly format, I then worked with 3 different dataframes during the vast majority of the project. At the end I decided that the weekly format provided the smoothest curve without losing too much information.
![weekly_new_con_new_deaths](https://user-images.githubusercontent.com/97986175/205494964-a111ec06-1024-40f4-9930-58c11b4672f7.png)

## 5. Create Features to be used in Model Building
[Preprocessing and Training Data Development Notebook](https://github.com/ColemanZ/Springboard/blob/main/%5BColeman%20Zimmerman%5D%20Covid%20Capstone%20Pre_Processing%20%26%20Training%20Data%20Development.ipynb)

A number of features were created for use in the eventual model building. We already touched on the "New Cases" and "New Deaths" features that were created using the running total of confirmed cases and confirmed deaths. In addition to these features I also created lag features. I first created lags for 24 days for both confirmed cases and confirmed deaths. The idea was that a high number of confirmed cases would be more likely if there had been a high number of confirmed cases leading up to that date. In line with that there should be a higher likelyhood of high deaths when confirmed cases for the last few weeks have been high. I ended up saving confirmed lags for the last 7, 14, and 21 days and then also converting those lags into a weekly and monthly format to fit the other two dataframes being used.
![image](https://user-images.githubusercontent.com/97986175/205495000-ba8540ef-9c5b-4239-a704-4517df9f8625.png)

## 6. Build Models
[Modeling Notebook](https://github.com/ColemanZ/Springboard/blob/main/%5BColeman%20Zimmerman%5D%20Covid%20Capstone%20-%20Modeling.ipynb)
After the data was cleaned and features were created I used pycaret to get a feel of what models needed to be built initially. After running that a number of times I chose to build 5 models to eventually get tuned and then ensembled. 
**The Models Built**
* Linear Regression
* Lasso Regression
* Least Angle Regression
* Lasso Lars Regression
* Bayesian Ridge

*Initial Least Angle Regression Graph*
![least_angle_regression_untuned](https://user-images.githubusercontent.com/97986175/205494747-b30371c6-4f00-498e-88bc-181555e7dd8b.png)

*Initial Bayesian Ridge Graph*
![bay_ridge_untuned](https://user-images.githubusercontent.com/97986175/205494829-2c3d58ec-2d93-496b-bd1c-8dd09b5732cd.png)

## 7. Tune Models
After all the models were built they were all tuned. Most of them changed slightly in their RMSE, MSE, or Coefficiant of Determination after tuning, but the Least Angle Regression model saw significant improvement. The Least Angle Regression model performed the best after tuning of the 5 models.
**Least Angle Regression Model**
**RMSE**:143.19
**MSE**:20504.16
**Coefficiant of Determination**:.85
*Tuned Least Angle Regression Graph*
![least_angle_regression_tuned](https://user-images.githubusercontent.com/97986175/205494793-18e1ad47-38fc-4561-b3eb-176769e5bf09.png)

## 9. Ensemble Models
Once all 5 models were tuned they were combined into an unweighted ensemble method. 
**Unweighted Ensemble Model**
**RMSE**:147.21
**MSE**:21671.59
**Coefficiant of Determination**:.84

Then I built a weighted voting regressor ensemble model and attempted different initial weights and steps. Originally I started with a .1 initial weight and steps of .1. This resulted in the models being weight .1, .1, .6, .1, and .1. Clearly showing that the third model was being used the most, so I thought if I changed the lowest weight and step to a smaller increment I would have a higher performing model. The new weighted voting regressor model had a .05 initial weight and steps of .05. This resulted in a very slow model to produce as it took over 12 hours for my computer to run. Clearly this would not be optimal in the real world. Nonetheless it provided weights of .05, .05, .8, .05, .05
**Weighted Ensemble Model**
**RMSE**:143.98
**MSE**:20730.20
**Coefficiant of Determination**:.85

Since these were not performing as well as the Least Angle Regression Model I decided to try one more thing. I took off the restriction forcing the total of all the models to equal one. At the .05 step I got .85, .05, .95, .05, and .05 as the weights for the new model. 
**Weighted Ensemble Model (with weights totalling over 1)**
**RMSE**:143.46
**MSE**:20580.42
**Coefficiant of Determination**:.85

## 10. Evaluate Models and Conclusion
It is clear from the model metrics that none of the ensembled models ended up being the strongest. The Least Angle Regression Model won out as the highest performing model and went ensembled with other models ended up being pulled down by them. As a future improvement I would try more varied kinds of models in hopes that when ensembled it would create a better model. Also if RAM was unlimited I would start at .01 and go in steps of .01 when building the voting regressor model. I may have wanted to look at other states that reported different metrics to provide more features.

## 11. Acknowledgements
Thank you to Raghunandan Patthar, my mentor at Springboard for his excellent advice throughout this project.
