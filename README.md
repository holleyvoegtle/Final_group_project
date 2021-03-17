# Biodiversity in National Parks
Final group project for UC Berkeley Extension Data Analytics Bootcamp

## Group Members
- Carmen Castanette
- Kevin Green
- Olivia Hughes
- Su-Lin Terhell
- Holley Voegtle

## Overview 
For our group project, we are looking at biodiversity in National Parks. We came to a consensus to choose this data set because we are all passionate about the environment.  
The selected dataset contains three csv files- one including species data, another containing data on the parks themselves and the third including annual visitation to the parks. 
More specifically, the species csv has information on classification of certain animals, which parks they can be found in, and how common they are among other things. 
The parks csv contains information on coordinates and acreage of the parks as well as which respective states they are located in. 
We also cleaned up the data and created an additional csv that included native vs non-native species.

### Purpose 
The initial question we investigated was to see if the increase in visitors to a park would increase the non-native species found there. Further investigation of the data also looked to see if there were any correlations to parameters such as location of the park and the presence of non-native species, grouped categories such as mammals, reptiles, plants, etc, and finally a correlation matrix. 

### Resources
- Data: 
	- https://www.kaggle.com/nationalparkservice/park-biodiversity/code 
	- https://irma.nps.gov/STATS/SSRSReports/National%20Reports/Annual%20Visitation%20By%20Park%20(1979%20-%20Last%20Calendar%20Year)

- Presentation: 

	 - Link to Google Slides presentation: https://docs.google.com/presentation/d/1YHni61PifSc4BHbQYm2np0Qa2-DJjMPLpCFDcb7-aCA/edit

- Software, Technologies, and Tools Utilized: 
	- Amazon Web Services
	- PostgreSQL
	- Tableau
	- Python
	- JavaScript
	- HTML
	- CSS
	- Jupyter Notebooks
	- SciKit Learn
	- GitHub


## Analysis

As with most data that is open source, our data needed to be extracted, transformed, and loaded (ETL). A number of DataFrames were created in Jupyter notebook (Python) and the use of Postgres was utilized for these DataFrames. From there linear regressions were tested and graphs were created. 

### Database:

For our database, we utilized Amazon Web Services to host a Postgres Relational Database Service (RDS). Various tables were loaded into Postgres using SQL through PGAdmin including our final machine learning table.  This allowed all team members to access the database server through PGAdmin as tables were updated and added throughout the course of the project. Some minor data cleaning was done to make the primary key column "Park Name" There was also a join done to add the average park visitation to the parks table  and create a table that separated all the categories per park into native or none native. The ERD is below.

![ERD](https://github.com/holleyvoegtle/Final_group_project/blob/main/database/ERD.png)

### Machine Learning

#### Pre-Processing Data:

The three csv's that were acquired from kaggle and loaded into our PostgresRDS needed to be transformed and merged into a final dataframe to feed the machine learning model. Below is an outline of steps that had to be completed for the final dataframe to come together:

	- Load in tables from Postgres for initial dataframes
	- 50 Entries needed rows shifted left that were entered incorrectly into Kaggle
		- This was found when exploring the 'Nativeness' attribute in the species.csv
		- Rows were corrected and replaced in the main dataframe
	- The visitation table needed to have the 'Park Name' field adjusted to remove abbreviations to join dataframes on.
	- Sequoia and King's Canyon National Parks needed to have their data merged into a single dataframe entry.
	- The 'Park Name' Column was set as the index for all three dataframes.
	- All dataframes were checked for null values.
	- Native and Non-Native Species were totalled for each park using the "groupby" function.
	- A field for Total Species for each park was calculated by summing the total Non-Native, total Native, and total Unknown columns
	- A field for NonNative_Ratio was created by dividing the number of non-native species by the total species in each park.
	- Species in each park were grouped by "Category" and added as a row for each park.
	- The species_df, parks_df, and visitation_df were merged to create a final dataframe
	- The final biodiversity_clean_df was loaded into our Postgres RDS to be used in the machine learning Jupyter Notebook.

To explore the possibilities with the dataset, a pearson correlation matrix was generated and graphed using a seaborn heatmap. The matrix is displayed below: 

![Correlation_Matrix](https://user-images.githubusercontent.com/64506842/111089495-8a33a000-8502-11eb-87b2-e32c7a2f383d.png)

#### Model:

Our group wanted to investigate the relationship between the number of visitors that each National Park received and how many non-native (invasive) species were found there. We initially hypothesized that an increase in park visitation would also increase the number of non-native species found in the park.  We decided on a linear regression model with our X variable being the average number of park visitors over 10 years and our y variable being the total number of non-native species found in the park.  

Our initial look at the data showed a scattered positive correlation with some potential outliers (circled in red):

![Park_Visitation_NonNative_Species_withCircles](https://user-images.githubusercontent.com/64506842/111092614-e69bbd00-850c-11eb-81aa-643607489604.png)

An initial linear regression model presented a positive correlation with an R-squared value scored of 0.06 and an RMSE of 217.45.

![AverageVisitors_NonNativeSpecies](https://user-images.githubusercontent.com/64506842/111092148-a38d1a00-850b-11eb-83e7-c6e2ae70051b.png)

The Hawaii National Parks were removed from our linear regression due to the high number of invasive species that were outliers in the data. Upon removing the two Hawaii parks, the model's R-squared value increased to 0.24 with a RMSE of 100.59.

![ParkVisitation_NonNatvie_Species_Hawaii_Removed](https://user-images.githubusercontent.com/64506842/111097804-975b8980-8518-11eb-9fcf-78c35bdff8e1.PNG)

The third outlier circled in the image above was Great Smokey Mountain National Park (GSMNP) with a much higher visitaiton than the other parks in this dataset. We removed this park from the dataset and ran another linear regression model. With GSMNP removed, the R-squared value decreased to 0.14 and the RMSE increased to 101.52.

![ParkVisitation_NonNatvie_Species_Hawaii_and_GSMNP_Removed](https://user-images.githubusercontent.com/64506842/111098059-1486fe80-8519-11eb-8571-9ab00454ca4f.PNG)


Histograms of the data revealed a non-normal distribution for both of our X and y variables. We decided to do a logarithmic transformation on the variables before training and fitting the data with another linear regrssion model. 


![Average_Visitors](https://user-images.githubusercontent.com/64506842/111091922-dd115580-850a-11eb-9610-939bb521feac.png) ![log_Average_Visitors](https://user-images.githubusercontent.com/64506842/111091930-e26ea000-850a-11eb-87e7-084912786bee.png)

![NonNativeSpecies](https://user-images.githubusercontent.com/64506842/111091950-f2867f80-850a-11eb-89cb-71c329471468.png) ![log_NonNativeSpecies](https://user-images.githubusercontent.com/64506842/111091943-ec909e80-850a-11eb-8f23-3ace59c2d4ca.png)


Linear regression models were re-run on the data with logarithmic transformations. In order to do the linear regression models, two Alaska National Parks had to be removed from the dataset that had zeros as their non-native species totals.

With a logarithmic transformation done on both X and y variables, the R-squared score was calculated to be 0.16 with an RMSE of 0.75.  

![log_AverageVisitors_log_NonNative](https://user-images.githubusercontent.com/64506842/111098635-4c427600-851a-11eb-9091-97e57a6fa15d.png)

It was difficult to compare the RMSE value of the log-transformed model to the non-tranformed models due to the RMSE units being transformed. 

## Website

In order to create a dashboard that will allow us to easily display our findings we also created a website that is located here: ***PUT LINK TO WEBSITE HERE***
This includes a summary of our question and analysis, an interactive map of all the national parks we looked and to display the number of native versus non-native species, and several different visualizations that we used during our analysis. 


## Summary
 After all the data analysis and exploration there seems to be a positive correlation between the number of average park visitors and the number of invasive species although it is not a significant as we first thought. This indicates that the National Park Service might need to limit the number of visitors to the national parks or explore other ways to manage invasive species. If you would like more information on the National Park Services current efforets to limit invasive species those can be found here: https://www.nps.gov/articles/invasive-species.htm. 


