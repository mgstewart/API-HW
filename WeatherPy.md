
# WeatherPy

## Analysis
* Relationship of Temperature and Latitude Position: The data decisively reveals what has been known since antiquity that regions closer to the equator of the Earth are warmer. This linear relationship is best revealed by using absolute latitudes. However, this approach does mask the apparent non-linear relationship of extreme latitutdes influencing temperatures in the southern hemisphere more than the northern hemisphere. This could be explained due to a few things. The first is that there are less cities in the southern hemisphere than in the north (due to both land surface area being lower and diasporic consequences of human migration).
* Relationships of Cloudiness, Humidity, and Wind Speed are decidedly non-linear, and likely not influenced by latitude. In the case of humidity, there is a strong distribution across all humidities for positive and negative latitudes. There does appear to be a higher density of 100% humidity cities within 20 degrees of the equator, but abundant data suggest there are non-humid cities right on the equator. Windspeed also has no immediate trends based on latitude except that there appears to be a general rise in wind speed when traveling further north in the hemisphere. This could be due to the terrain influencing wind speed as steppes, tundras, and other flatlands would experience higher wind speeds. Overall, these can be fairly well explained by an understanding that temperature is largely dependent on ability of the area to receive sun radiation. Wind, humidity, and cloudiness are more complex functions that are themselves influenced by temperature (which cycles due to Earth's tilt and rotations).
* Nature of Cloudiness Data: Interestingly, despite cloud coverage is represented as a percentage, which is a continuous variable, visualization of the data reveals a decidedly categorical nature. Interestingly, although an approximation to the nearest 10% value would probably be good enough, several location report non-10-multiple cloud coverages. It is also worth noting that there are many more places that have no cloud coverage than complete cloud coverage, which makes sense given that cloud's are not permanent bodies and cities reside only on land, whereas if ocean data was included/available, there could be a higher prevalence of cloudy areas.

#### To make cleaning of the data simpler, the citipy library's dataset can be acquired and converted into a dataframe.
#### This allows a simple pandas drop_duplicates method to be used (on lat/long). See the output below the codeblock to see how approx. 30% of the dataset is identical coordinate duplicates
#### Note: this method does *not* account for near-duplicates that will remain in the dataset. By sampling a large number, (e.g. n=1000) from over 2E6 cities, it is unlikely these duplicates will affect visual trends.


```python
import openweathermapy as owm
import pandas as pd
import matplotlib.pyplot as plt
import requests
import random
from config import api_key
import seaborn as sns
#Import list of cities
citydf = pd.read_csv('worldcitiespop.txt',encoding='utf-8',low_memory=False,dtype=str)
print(citydf.info())
#Remove lat/lng duplicates
citydf = citydf.drop(columns=['AccentCity','Region','Population'])
citydf = citydf.drop_duplicates(subset=['Latitude','Longitude'],keep='first')
citydf = citydf.reset_index()
print(citydf.info())
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 3173958 entries, 0 to 3173957
    Data columns (total 7 columns):
    Country       object
    City          object
    AccentCity    object
    Region        object
    Population    object
    Latitude      object
    Longitude     object
    dtypes: object(7)
    memory usage: 169.5+ MB
    None
    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2243854 entries, 0 to 2243853
    Data columns (total 5 columns):
    index        int64
    Country      object
    City         object
    Latitude     object
    Longitude    object
    dtypes: int64(1), object(4)
    memory usage: 85.6+ MB
    None


## Define parameters for querying OWM


```python
settings = {'units':'imperial','APPID':api_key}
```

## Define the sample size below and add relevant columns to dataframe prior to querying


```python
#Build cities data frame by using pandas sample method
# initial testing will use 50 for expedited 
cities = citydf.sample(1000)
#Add columns for max temp to working dataframe
cities['Max Temp'] = ''
cities['Cloudiness'] = ''
cities['Wind Speed'] = ''
cities['Humidity'] = ''
```

## Create lists to hold the queried values as the sample is looped over


```python
#Iterate over each row of the dataframe, pulling API data into lists
# both pandas documentation and stackoverflow caution against trying
# to append/modify a dataframe being iterated over
list_of_max_temps = []
list_of_cloud = []
list_of_wind = []
list_of_humidity = []
for row, index in cities.iterrows():
    lat = index['Latitude']
    lng = index['Longitude']
    location = (lat, lng)
    try:
        data = owm.get_current(location,**settings)
        list_of_max_temps.append(data("main.temp_max"))
        list_of_cloud.append(data("clouds.all"))
        list_of_wind.append(data("wind.speed"))
        list_of_humidity.append(data("main.humidity"))
    except:
        raise print("Whoops! Something went wrong, check your lat/long coordinates!")
print('Retrieval of OWM data is finished.')
```

    Retrieval of OWM data is finished.


## Pass the lists into the cities dataframe
#### Note: some lively discussion and timetests online suggest this is a procedurally faster way to add in values than looping and using loc or at methods


```python
#Add the lists of data into the data frame
cities['Max Temp'] = list_of_max_temps
cities['Cloudiness'] = list_of_cloud
cities['Wind Speed'] = list_of_wind
cities['Humidity'] = list_of_humidity
```


```python
cities.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>Country</th>
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Max Temp</th>
      <th>Cloudiness</th>
      <th>Wind Speed</th>
      <th>Humidity</th>
      <th>Latitude(Abs)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1626321</th>
      <td>2338488</td>
      <td>ro</td>
      <td>suesti</td>
      <td>44.866667</td>
      <td>24.066667</td>
      <td>66.20</td>
      <td>64</td>
      <td>9.17</td>
      <td>68</td>
      <td>44.866667</td>
    </tr>
    <tr>
      <th>1306747</th>
      <td>1922917</td>
      <td>my</td>
      <td>kampong gapis</td>
      <td>4.150000</td>
      <td>101.9</td>
      <td>72.48</td>
      <td>88</td>
      <td>3.36</td>
      <td>99</td>
      <td>4.150000</td>
    </tr>
    <tr>
      <th>395407</th>
      <td>607148</td>
      <td>cn</td>
      <td>wangjiashan</td>
      <td>35.961085</td>
      <td>102.90309</td>
      <td>52.05</td>
      <td>64</td>
      <td>2.35</td>
      <td>94</td>
      <td>35.961085</td>
    </tr>
    <tr>
      <th>1018163</th>
      <td>1473104</td>
      <td>ir</td>
      <td>qolqoleh tappeh</td>
      <td>36.839900</td>
      <td>45.9686</td>
      <td>46.65</td>
      <td>44</td>
      <td>1.57</td>
      <td>93</td>
      <td>36.839900</td>
    </tr>
    <tr>
      <th>1537766</th>
      <td>2219449</td>
      <td>pk</td>
      <td>naru</td>
      <td>27.414766</td>
      <td>68.660212</td>
      <td>86.00</td>
      <td>0</td>
      <td>4.70</td>
      <td>45</td>
      <td>27.414766</td>
    </tr>
  </tbody>
</table>
</div>



## Add an "Absolute Latitude" (degrees positive or negative from equator) to better show temperature regression


```python
#Transform Lat values in DF into absolute values now that parsing is done
# note the values were imported as strings to make querying easily
cities['Latitude'] = pd.to_numeric(cities['Latitude'],errors='raise')
cities['Latitude(Abs)'] = abs(cities['Latitude'])
cities.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>Country</th>
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Max Temp</th>
      <th>Cloudiness</th>
      <th>Wind Speed</th>
      <th>Humidity</th>
      <th>Latitude(Abs)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1626321</th>
      <td>2338488</td>
      <td>ro</td>
      <td>suesti</td>
      <td>44.866667</td>
      <td>24.066667</td>
      <td>66.20</td>
      <td>64</td>
      <td>9.17</td>
      <td>68</td>
      <td>44.866667</td>
    </tr>
    <tr>
      <th>1306747</th>
      <td>1922917</td>
      <td>my</td>
      <td>kampong gapis</td>
      <td>4.150000</td>
      <td>101.9</td>
      <td>72.48</td>
      <td>88</td>
      <td>3.36</td>
      <td>99</td>
      <td>4.150000</td>
    </tr>
    <tr>
      <th>395407</th>
      <td>607148</td>
      <td>cn</td>
      <td>wangjiashan</td>
      <td>35.961085</td>
      <td>102.90309</td>
      <td>52.05</td>
      <td>64</td>
      <td>2.35</td>
      <td>94</td>
      <td>35.961085</td>
    </tr>
    <tr>
      <th>1018163</th>
      <td>1473104</td>
      <td>ir</td>
      <td>qolqoleh tappeh</td>
      <td>36.839900</td>
      <td>45.9686</td>
      <td>46.65</td>
      <td>44</td>
      <td>1.57</td>
      <td>93</td>
      <td>36.839900</td>
    </tr>
    <tr>
      <th>1537766</th>
      <td>2219449</td>
      <td>pk</td>
      <td>naru</td>
      <td>27.414766</td>
      <td>68.660212</td>
      <td>86.00</td>
      <td>0</td>
      <td>4.70</td>
      <td>45</td>
      <td>27.414766</td>
    </tr>
  </tbody>
</table>
</div>



## Plot the association of max temperature(F) against the city's latitude


```python
sns.set_style('darkgrid')
temp_ax = sns.regplot(x=cities['Latitude'],y=cities['Max Temp'],fit_reg=False)
temp_ax.set(xlabel='Degrees Latitude',ylabel='Max Temp (F)',title='Max Temp(F) of random cities vs. Latitude')
```




    [Text(0,0.5,'Max Temp (F)'),
     Text(0.5,0,'Degrees Latitude'),
     Text(0.5,1,'Max Temp(F) of random cities vs. Latitude')]




![png](output_16_1.png)


## Plot the same temperature data against absolute latitude (DO NOT GRADE)
#### Note: While not requested in the homework, this is a vastly better visualization approach in showing the linear relationship we expect with temperature and "closeness" to the equator.


```python
abs_temp_ax = sns.regplot(x=cities['Latitude(Abs)'],y=cities['Max Temp'])
abs_temp_ax.set(xlabel='Degrees Lat. from Equator',ylabel='Max Temp (F)',title='Max Temp(F) of random cities vs. Lat. from Equator')
```




    [Text(0,0.5,'Max Temp (F)'),
     Text(0.5,0,'Degrees Lat. from Equator'),
     Text(0.5,1,'Max Temp(F) of random cities vs. Lat. from Equator')]




![png](output_18_1.png)


## Plot Humidity against Latitude


```python
humid_ax = sns.regplot(x=cities['Latitude'],y=cities['Humidity'],fit_reg=False)
humid_ax.set(ylabel='Humidity(%)',title='Humidity vs City Latitude')
```




    [Text(0,0.5,'Humidity(%)'), Text(0.5,1,'Humidity vs City Latitude')]




![png](output_20_1.png)


## Plot Cloudiness against Latitude


```python
cloud_ax = sns.regplot(x=cities['Latitude'],y=cities['Cloudiness'],fit_reg=False)
cloud_ax.set(ylabel='Cloudiness(%)',title='Cloudiness vs. City Latitude')
```




    [Text(0,0.5,'Cloudiness(%)'), Text(0.5,1,'Cloudiness vs. City Latitude')]




![png](output_22_1.png)


## Plot Wind Speed against Latitude


```python
wind_ax = sns.regplot(x=cities['Latitude'],y=cities['Wind Speed'],fit_reg=False)
wind_ax.set(ylabel='Wind Speed (mph)',title='Wind Speed (mph) vs City Latitutde')
```




    [Text(0,0.5,'Wind Speed (mph)'),
     Text(0.5,1,'Wind Speed (mph) vs City Latitutde')]




![png](output_24_1.png)

