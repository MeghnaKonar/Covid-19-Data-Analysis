# Covid-19-Data-Analysis
COVID-19 data analysis using Python in Jupyter Notebook involves the use of various libraries and tools to explore, visualize, and interpret COVID-19-related datasets. Here's a brief overview of the steps you might take:

1. Importing Libraries:
Commonly used libraries for data analysis in Python include Pandas, NumPy, and Matplotlib/Seaborn for data manipulation and visualization.

<pre>
#Importing Modules
  import pandas as pd
  import numpy as np
  import matplotlib.pyplot as plt
  import plotly.graph_objs as go
  import seaborn as sns
  import plotly.express as px
  from plotly.subplots import make_subplots
  from datetime import datetime
</pre>

2. Loading Data:
Obtain a COVID-19 dataset, which could include information such as confirmed cases, deaths, and recoveries over time. Datasets are often available from reliable sources like Johns Hopkins University or government health agencies. Here, I have used dataset from Kaggle

<pre>
  covid_df = pd.read_csv("Datasets/covid_19_india.csv")
  vaccine_df = pd.read_csv("Datasets/covid_vaccine_statewise.csv")
  daily_df = pd.read_csv("Datasets/global_daily.csv")
  summary_df = pd.read_csv("Datasets/global_summary.csv")
</pre>

4. Data Exploration:
Examine the structure of the dataset, check for missing values, and understand the data types.
<pre>
  #for covid_df dataset
  covid_df.head(10)
  covid_df.info()
  covid_df.describe()

  #for vaccine_df dataset
  vaccine_df.head(7)
  vaccine_df.info()
  vaccine_df.isnull().sum()

  #for daily_df dataset
  daily_df.tail(10)
  daily_df.shape
  daily_df.columns

  #for summary_df dataset
  summary_df.shape
  summary_df.columns
  summary_df.head(10)
</pre>

5. Data Cleaning:
Handle missing values, outliers, and format data for analysis.
<pre>
  #drop the Sno, Time, ConfirmedIndianNational, ConfirmedForeignNational columns from the table
  covid_df.drop(["Sno", "Time", "ConfirmedIndianNational", "ConfirmedForeignNational"], inplace = True, axis = 1 )

  #Convert date column to datetime format
  covid_df['Date'] = pd.to_datetime(covid_df['Date'], format = '%Y-%m-%d')

  #Handling missing values
  covid_df = covid_df.fillna(0)

  #Create an active cases column
  covid_df['Active_Cases'] = covid_df['Confirmed'] - (covid_df['Cured']-covid_df['Deaths'])

  #Rename the column to vaccine date
  vaccine_df.rename(columns = { 'Updated On' : 'Vaccine_Date'}, inplace = True)

  #Handling Missing Values
  vaccine_df = vaccine_df.fillna(0)

  #Drop the columns
  vaccination = vaccine_df.drop(columns = ['Sputnik V (Doses Administered)', 'AEFI', '18-44 Years (Doses Administered)', '45-60 Years (Doses Administered)', '60+ Years (Doses Administered)'], axis = 1)

  #Remove Rows where state is India
  vaccine = vaccine_df[vaccine_df.State!= 'India']

  #Rename the columns
  vaccine.rename(columns = { "Total Individuals Vaccinated" : "Total"}, inplace = True)

  #Handling Missing Values
  daily_df = daily_df.fillna(0)
  summary_df = summary_df.fillna(0)

  #Creating a mortality rate column
  summary_df["mortality_rate"] = summary_df["total_deaths"] / summary_df["population"] * 100

  #Creating a fatality rate
  summary_df["fatality_rate"] = summary_df["total_deaths"] / summary_df["total_confirmed"] * 100

  #Group by country
  country_group = summary_df.groupby(["country","continent"], as_index = False).agg({'total_confirmed': pd.Series.count})
</pre>

7. Data Visualization:
Create visualizations to better understand trends and patterns in the data.

<pre>
  # Creating Pivot Table
  statewise = pd.pivot_table(covid_df, values = ["Confirmed", "Deaths", "Cured"], index = "State/UnionTerritory", aggfunc = "max")
  statewise["Recovery Rate"] = statewise["Cured"]*100/statewise["Confirmed"]
  statewise["Mortality Rate"] = statewise["Deaths"]*100/statewise["Confirmed"]
  statewise = statewise.sort_values(by = "Confirmed", ascending = False)
  statewise.style.background_gradient(cmap = "gist_earth")

  # Top 10 Active Cases State - Bar plot
  top_10_active_cases = covid_df.groupby(by = 'State/UnionTerritory').max()[['Active_Cases', 'Date']].sort_values(by = ['Active_Cases'], ascending = False).reset_index()
  fig = plt.figure(figsize=(18,10))
  plt.title("Top 10 states with most active cases in India", size = 25)
  ax = sns.barplot(data = top_10_active_cases.iloc[:10], y = "Active_Cases", x = "State/UnionTerritory", linewidth = 2, edgecolor = 'black')
  plt.xlabel("States")
  plt.ylabel("Total Active Cases")
  plt.show()

  # Top States with Highest Deaths - Bar Plot
  top_10_deaths = covid_df.groupby(by = 'State/UnionTerritory').max()[['Deaths', 'Date']].sort_values(by = ['Deaths'], ascending = False).reset_index()
  fig = plt.figure(figsize=(18,10))
  plt.title("Top 10 states with most deaths in India", size = 25)
  ax = sns.barplot(data = top_10_deaths.iloc[:10], y = "Deaths", x = "State/UnionTerritory", linewidth = 2, edgecolor = 'red')
  plt.xlabel("States")
  plt.ylabel("Total Death Cases")
  plt.show()

  # Growth Trend - Line Graph
  fig = plt.figure(figsize = (12,6))
  ax = sns.lineplot(data = covid_df[covid_df['State/UnionTerritory'].isin(['Maharashtra', 'Karnataka', 'Kerala', 'Tamil Nadu', 'Uttar Pradesh'])], x = "Date", 
  y = "Active_Cases", 
  hue = "State/UnionTerritory")
  ax.set_title("Top 5 Affected States in India", size =16)

  # Male vs Female vaccination - Pie Chart
  male = vaccination["Male(Individuals Vaccinated)"].sum()
  female = vaccination["Female(Individuals Vaccinated)"].sum()
  px.pie(names=["Male", "Female"], values = [male, female], title = "Male and Female Vaccination")

  # Top 5 Vaccinated States In India - Bar Plot
  fig = plt.figure(figsize = (10,5))
  plt.title("Top 5 Vaccinated States in India", size = 20)
  x = sns.barplot(data = max_vac.iloc[:10], y = max_vac.Total, x = max_vac.index, linewidth = 2, edgecolor = 'black')
  plt.xlabel("States")
  plt.ylabel("Vaccination")
  plt.show()

  # Least 5 Vaccinated States In India - Bar Plot

  fig = plt.figure(figsize = (18,9))
  plt.title("Least 5 Vaccinated States in India", size = 20)
  x = sns.barplot(data = min_vac.iloc[:10], y = min_vac.Total, x = min_vac.index, linewidth = 2, edgecolor = 'black')
  plt.xlabel("States")
  plt.ylabel("Vaccination")
  plt.show()

  #Mortality Rate by Countries in Line Graph
  line = px.line(
      summary_df,
      x = 'country',
      y = 'mortality_rate',
      color = 'continent',
      title = 'Mortality Rate by Countries'
  )
  line.show()
  
</pre>

8. Statistical Analysis:
Conduct statistical analysis to derive insights.
