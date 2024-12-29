```python
import os
import pandas as pd
import matplotlib.pyplot as plt
import plotly.express as px
import numpy as np
import json
import seaborn as sns
import warnings
from urllib.request import urlopen
from plotly import graph_objects as go
from sklearn import datasets
from sklearn.linear_model import LinearRegression
from dotenv import load_dotenv
```


```python
load_dotenv()
```




    True




```python
Sub_National_Energy_Consumption_1 = os.getenv('Sub_National_Energy_Consumption_1')
Sub_National_Gas_Consumption_2 = os.getenv('Sub_National_Gas_Consumption_2')
UK_Population_3 = os.getenv('UK_Population_3') 
Local_Authority_District_to_Region_4 = os.getenv('Local_Authority_District_to_Region_4') 
Domestic_Energy_Bills_5 = os.getenv('Domestic_Energy_Bills_5') 
Energy_Import_Costs_6 = os.getenv('Energy_Import_Costs_6') 
Average_Earning_7 = os.getenv('Average_Earning_7')
```


```python
warnings.filterwarnings("ignore", category=FutureWarning)
```


```python
#Defining the sheet names to read from the excel
Years = [str(n) for n in list(range(2005,2023))]

#Defining the dataframe which will hold the Energy consumption data as the concatination of sheets in the excel with the Year appended
Energy_Consumption = pd.concat(((pd.read_excel(io=Sub_National_Energy_Consumption_1, sheet_name=Year, header=5, nrows=11, skiprows=[6,7,8,9,10,18,19]).assign(Year = Year))for Year in Years), ignore_index=True)
Energy_Consumption['Country or region'].unique()
```




    array(['North East', 'North West', 'Yorkshire and The Humber',
           'East Midlands', 'West Midlands', 'East', 'London', 'South East',
           'South West'], dtype=object)




```python
#Getting the sum of energy used in England for each year
Total_Energy_Consumption = Energy_Consumption.groupby('Year').sum()

#Rename the column for improved readability
Total_Energy_Consumption.rename(columns= {'All fuels:\nTotal': 'Total Energy Consumption (ktoe)'},inplace=True)
Total_Energy_Consumption['Total Energy Consumption (GWh)'] = Total_Energy_Consumption['Total Energy Consumption (ktoe)'] * 11.63
Total_Energy_Consumption[['Total Energy Consumption (GWh)','Total Energy Consumption (ktoe)']]
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
      <th>Total Energy Consumption (GWh)</th>
      <th>Total Energy Consumption (ktoe)</th>
    </tr>
    <tr>
      <th>Year</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2005</th>
      <td>1.477577e+06</td>
      <td>127048.781758</td>
    </tr>
    <tr>
      <th>2006</th>
      <td>1.440840e+06</td>
      <td>123889.897740</td>
    </tr>
    <tr>
      <th>2007</th>
      <td>1.417789e+06</td>
      <td>121907.900523</td>
    </tr>
    <tr>
      <th>2008</th>
      <td>1.364505e+06</td>
      <td>117326.330106</td>
    </tr>
    <tr>
      <th>2009</th>
      <td>1.294634e+06</td>
      <td>111318.450564</td>
    </tr>
    <tr>
      <th>2010</th>
      <td>1.306030e+06</td>
      <td>112298.362641</td>
    </tr>
    <tr>
      <th>2011</th>
      <td>1.257540e+06</td>
      <td>108128.997039</td>
    </tr>
    <tr>
      <th>2012</th>
      <td>1.252130e+06</td>
      <td>107663.825360</td>
    </tr>
    <tr>
      <th>2013</th>
      <td>1.236627e+06</td>
      <td>106330.742765</td>
    </tr>
    <tr>
      <th>2014</th>
      <td>1.246193e+06</td>
      <td>107153.334706</td>
    </tr>
    <tr>
      <th>2015</th>
      <td>1.242000e+06</td>
      <td>106792.796861</td>
    </tr>
    <tr>
      <th>2016</th>
      <td>1.234045e+06</td>
      <td>106108.794015</td>
    </tr>
    <tr>
      <th>2017</th>
      <td>1.251013e+06</td>
      <td>107567.781376</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>1.253871e+06</td>
      <td>107813.486291</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>1.247716e+06</td>
      <td>107284.307060</td>
    </tr>
    <tr>
      <th>2020</th>
      <td>1.157838e+06</td>
      <td>99556.185044</td>
    </tr>
    <tr>
      <th>2021</th>
      <td>1.173627e+06</td>
      <td>100913.755388</td>
    </tr>
    <tr>
      <th>2022</th>
      <td>1.143604e+06</td>
      <td>98332.257045</td>
    </tr>
  </tbody>
</table>
</div>




```python
#For this datasource not all the sheets have the same format, so the read_excel function must be adjusted for each.

#Defining the sheet names to read from the excel
Years1 = [str(n) for n in list(range(2005,2014))]
Gas_Consumption1 = pd.concat(((pd.read_excel(io=Sub_National_Gas_Consumption_2, sheet_name=Year, header=5, nrows=9, skiprows=[6,7]).assign(Year = Year))for Year in Years1), ignore_index=True)

#In this one Inner and outerlondon are differinciated
Years2 = [str(n) for n in list(range(2014,2015))]
Gas_Consumption2 = pd.concat(((pd.read_excel(io=Sub_National_Gas_Consumption_2, sheet_name=Year, header=5, nrows=10, skiprows=[6,7]).assign(Year = Year))for Year in Years2), ignore_index=True)

#Removing the sub regions of Inner and outer London as as sum of these 'London is already included' this is to avoid double counting
Years3 = [str(n) for n in list(range(2015,2023))]
Gas_Consumption3 = pd.concat(((pd.read_excel(io=Sub_National_Gas_Consumption_2, sheet_name=Year, header=5, nrows=11, skiprows=[6,7,8,16,17]).assign(Year = Year))for Year in Years3), ignore_index=True)


#Combined data from all sheets into one dataframe
Gas_Consumption = pd.concat([Gas_Consumption1, Gas_Consumption2, Gas_Consumption3])

```


```python
#Getting the sum of gas used in England for each year
Total_Gas_Consumption = Gas_Consumption.groupby('Year').sum()

#Rename the column for improved readability
Total_Gas_Consumption.rename(columns= {'Total consumption\n(GWh):\nAll meters': 'Total Gas Consumption (GWh)'},inplace=True)
Total_Gas_Consumption['Total Gas Consumption (GWh)'] 
```




    Year
    2005    571884.940304
    2006    537493.963018
    2007    523183.686839
    2008    499308.585633
    2009    458351.437002
    2010    460366.820221
    2011    436641.855643
    2012    433035.246141
    2013    424176.434549
    2014    426286.006022
    2015    414952.969360
    2016    412096.411620
    2017    424670.572211
    2018    427583.693925
    2019    426309.929563
    2020    436137.014968
    2021    422110.562024
    2022    380574.844003
    Name: Total Gas Consumption (GWh), dtype: float64




```python
#Joining the Energy and Gas consumption by Year dataframes together
Energy_Gas_Consumption_Combined = pd.merge(Total_Energy_Consumption,Total_Gas_Consumption, how ='left', on=['Year','Year'])

#Calculating the percentage of total energy consumption gas consumption makes up
Energy_Gas_Consumption_Combined['Gas_Consumption_as_Percent'] = Energy_Gas_Consumption_Combined['Total Gas Consumption (GWh)']/Energy_Gas_Consumption_Combined['Total Energy Consumption (GWh)']*100
Energy_Gas_Consumption_Combined['Other_Consumption_as_Percent'] = 100 - Energy_Gas_Consumption_Combined['Gas_Consumption_as_Percent']
#fixing the year index
Energy_Gas_Consumption_Combined = Energy_Gas_Consumption_Combined.reset_index(names=['Year_x','Year'])
Energy_Gas_Consumption_Combined = Energy_Gas_Consumption_Combined.drop(columns='Year_x', axis=0)

#Printing the dataframe columns which will be plotted
Energy_Gas_Consumption_Combined[['Year', 'Total Energy Consumption (GWh)','Total Gas Consumption (GWh)','Gas_Consumption_as_Percent','Other_Consumption_as_Percent']]

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
      <th>Year</th>
      <th>Total Energy Consumption (GWh)</th>
      <th>Total Gas Consumption (GWh)</th>
      <th>Gas_Consumption_as_Percent</th>
      <th>Other_Consumption_as_Percent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005</td>
      <td>1.477577e+06</td>
      <td>571884.940304</td>
      <td>38.704231</td>
      <td>61.295769</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2006</td>
      <td>1.440840e+06</td>
      <td>537493.963018</td>
      <td>37.304222</td>
      <td>62.695778</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2007</td>
      <td>1.417789e+06</td>
      <td>523183.686839</td>
      <td>36.901382</td>
      <td>63.098618</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2008</td>
      <td>1.364505e+06</td>
      <td>499308.585633</td>
      <td>36.592648</td>
      <td>63.407352</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2009</td>
      <td>1.294634e+06</td>
      <td>458351.437002</td>
      <td>35.403951</td>
      <td>64.596049</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2010</td>
      <td>1.306030e+06</td>
      <td>460366.820221</td>
      <td>35.249331</td>
      <td>64.750669</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2011</td>
      <td>1.257540e+06</td>
      <td>436641.855643</td>
      <td>34.721899</td>
      <td>65.278101</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2012</td>
      <td>1.252130e+06</td>
      <td>433035.246141</td>
      <td>34.583881</td>
      <td>65.416119</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2013</td>
      <td>1.236627e+06</td>
      <td>424176.434549</td>
      <td>34.301094</td>
      <td>65.698906</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2014</td>
      <td>1.246193e+06</td>
      <td>426286.006022</td>
      <td>34.207054</td>
      <td>65.792946</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2015</td>
      <td>1.242000e+06</td>
      <td>414952.969360</td>
      <td>33.410056</td>
      <td>66.589944</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2016</td>
      <td>1.234045e+06</td>
      <td>412096.411620</td>
      <td>33.393946</td>
      <td>66.606054</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2017</td>
      <td>1.251013e+06</td>
      <td>424670.572211</td>
      <td>33.946128</td>
      <td>66.053872</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2018</td>
      <td>1.253871e+06</td>
      <td>427583.693925</td>
      <td>34.101095</td>
      <td>65.898905</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2019</td>
      <td>1.247716e+06</td>
      <td>426309.929563</td>
      <td>34.167211</td>
      <td>65.832789</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2020</td>
      <td>1.157838e+06</td>
      <td>436137.014968</td>
      <td>37.668210</td>
      <td>62.331790</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2021</td>
      <td>1.173627e+06</td>
      <td>422110.562024</td>
      <td>35.966331</td>
      <td>64.033669</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2022</td>
      <td>1.143604e+06</td>
      <td>380574.844003</td>
      <td>33.278547</td>
      <td>66.721453</td>
    </tr>
  </tbody>
</table>
</div>




```python
Energy_Consumption_Fig = px.line(Energy_Gas_Consumption_Combined, x='Year', y=['Total Energy Consumption (GWh)','Total Gas Consumption (GWh)'])

Energy_Consumption_Fig.update_layout(
    title="Energy Consumption in the UK by year (GWh)",
    xaxis_title="Year",
    yaxis_title="Energy Consumption (GWh)",
    title_x=0.5, 
    legend=dict(
        orientation="h", 
        yanchor="bottom", 
        y=0.88, 
        xanchor="center",  
        x=0.5,
        title = None
    ),
    height = 700,
    font=dict(
        size=26 
    )
)

Energy_Consumption_Fig
```




```python
Gas_Consumption_as_Percentage_for_Chart = Energy_Gas_Consumption_Combined[['Year','Gas_Consumption_as_Percent','Other_Consumption_as_Percent']]
Gas_Consumption_as_Percentage_for_Chart = Gas_Consumption_as_Percentage_for_Chart.set_index('Year')
#Gas_Consumption_as_Percentage_for_Chart
#Gas_Proportion_Fig = Gas_Consumption_as_Percentage_for_Chart.plot(kind = 'bar', stacked = True )
Gas_Consumption_as_Percentage_for_Chart_Bar = px.bar(Gas_Consumption_as_Percentage_for_Chart)
Gas_Consumption_as_Percentage_for_Chart_Bar.update_layout(
    margin={"r": 10, "t": 140, "l": 10, "b": 10},
    title='Gas as a Portion of Total Energy Consumption',
    xaxis_title="Year",
    yaxis_title="Percentage Consumption (%)",
    title_x=0.5,
    legend=dict(
        orientation="h", 
        yanchor="bottom", 
        y=1, 
        xanchor="center",  
        x=0.5,
        title = None
    ),
    height = 700,
    font=dict(
        size=26 
    ))


```



The Importation of Population Data


```python
#The Importation of Population Data

#Source for this data it's the 'Mid-2011 to mid-2022 edition of this dataset'
#https://www.ons.gov.uk/peoplepopulationandcommunity/populationandmigration/populationestimates/datasets/populationestimatesforukenglandandwalesscotlandandnorthernireland

UK_Population_Raw_Data = pd.read_excel(io=UK_Population_3,sheet_name='MYEB1', header=1)
UK_Population_Raw_Data
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
      <th>ladcode23</th>
      <th>laname23</th>
      <th>country</th>
      <th>sex</th>
      <th>age</th>
      <th>population_2011</th>
      <th>population_2012</th>
      <th>population_2013</th>
      <th>population_2014</th>
      <th>population_2015</th>
      <th>population_2016</th>
      <th>population_2017</th>
      <th>population_2018</th>
      <th>population_2019</th>
      <th>population_2020</th>
      <th>population_2021</th>
      <th>population_2022</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>E06000001</td>
      <td>Hartlepool</td>
      <td>E</td>
      <td>F</td>
      <td>0</td>
      <td>555</td>
      <td>565</td>
      <td>508</td>
      <td>513</td>
      <td>517</td>
      <td>507</td>
      <td>508</td>
      <td>463</td>
      <td>495</td>
      <td>455</td>
      <td>446</td>
      <td>430</td>
    </tr>
    <tr>
      <th>1</th>
      <td>E06000001</td>
      <td>Hartlepool</td>
      <td>E</td>
      <td>F</td>
      <td>1</td>
      <td>584</td>
      <td>557</td>
      <td>561</td>
      <td>506</td>
      <td>515</td>
      <td>522</td>
      <td>526</td>
      <td>501</td>
      <td>462</td>
      <td>498</td>
      <td>477</td>
      <td>461</td>
    </tr>
    <tr>
      <th>2</th>
      <td>E06000001</td>
      <td>Hartlepool</td>
      <td>E</td>
      <td>F</td>
      <td>2</td>
      <td>561</td>
      <td>570</td>
      <td>565</td>
      <td>556</td>
      <td>509</td>
      <td>518</td>
      <td>521</td>
      <td>516</td>
      <td>527</td>
      <td>459</td>
      <td>506</td>
      <td>489</td>
    </tr>
    <tr>
      <th>3</th>
      <td>E06000001</td>
      <td>Hartlepool</td>
      <td>E</td>
      <td>F</td>
      <td>3</td>
      <td>565</td>
      <td>567</td>
      <td>578</td>
      <td>564</td>
      <td>552</td>
      <td>532</td>
      <td>534</td>
      <td>523</td>
      <td>531</td>
      <td>522</td>
      <td>463</td>
      <td>512</td>
    </tr>
    <tr>
      <th>4</th>
      <td>E06000001</td>
      <td>Hartlepool</td>
      <td>E</td>
      <td>F</td>
      <td>4</td>
      <td>546</td>
      <td>552</td>
      <td>557</td>
      <td>582</td>
      <td>564</td>
      <td>557</td>
      <td>530</td>
      <td>538</td>
      <td>514</td>
      <td>520</td>
      <td>523</td>
      <td>468</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>65697</th>
      <td>W06000024</td>
      <td>Merthyr Tydfil</td>
      <td>W</td>
      <td>M</td>
      <td>86</td>
      <td>62</td>
      <td>58</td>
      <td>56</td>
      <td>75</td>
      <td>81</td>
      <td>66</td>
      <td>74</td>
      <td>71</td>
      <td>74</td>
      <td>63</td>
      <td>85</td>
      <td>84</td>
    </tr>
    <tr>
      <th>65698</th>
      <td>W06000024</td>
      <td>Merthyr Tydfil</td>
      <td>W</td>
      <td>M</td>
      <td>87</td>
      <td>46</td>
      <td>55</td>
      <td>43</td>
      <td>46</td>
      <td>64</td>
      <td>69</td>
      <td>61</td>
      <td>63</td>
      <td>62</td>
      <td>58</td>
      <td>54</td>
      <td>76</td>
    </tr>
    <tr>
      <th>65699</th>
      <td>W06000024</td>
      <td>Merthyr Tydfil</td>
      <td>W</td>
      <td>M</td>
      <td>88</td>
      <td>48</td>
      <td>36</td>
      <td>46</td>
      <td>35</td>
      <td>39</td>
      <td>55</td>
      <td>61</td>
      <td>61</td>
      <td>56</td>
      <td>55</td>
      <td>50</td>
      <td>50</td>
    </tr>
    <tr>
      <th>65700</th>
      <td>W06000024</td>
      <td>Merthyr Tydfil</td>
      <td>W</td>
      <td>M</td>
      <td>89</td>
      <td>32</td>
      <td>44</td>
      <td>33</td>
      <td>41</td>
      <td>27</td>
      <td>31</td>
      <td>46</td>
      <td>49</td>
      <td>52</td>
      <td>43</td>
      <td>50</td>
      <td>44</td>
    </tr>
    <tr>
      <th>65701</th>
      <td>W06000024</td>
      <td>Merthyr Tydfil</td>
      <td>W</td>
      <td>M</td>
      <td>90</td>
      <td>103</td>
      <td>116</td>
      <td>130</td>
      <td>131</td>
      <td>136</td>
      <td>128</td>
      <td>112</td>
      <td>117</td>
      <td>130</td>
      <td>127</td>
      <td>122</td>
      <td>135</td>
    </tr>
  </tbody>
</table>
<p>65702 rows × 17 columns</p>
</div>




```python
UK_Population_Raw_Data[UK_Population_Raw_Data['laname23']== '']
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
      <th>ladcode23</th>
      <th>laname23</th>
      <th>country</th>
      <th>sex</th>
      <th>age</th>
      <th>population_2011</th>
      <th>population_2012</th>
      <th>population_2013</th>
      <th>population_2014</th>
      <th>population_2015</th>
      <th>population_2016</th>
      <th>population_2017</th>
      <th>population_2018</th>
      <th>population_2019</th>
      <th>population_2020</th>
      <th>population_2021</th>
      <th>population_2022</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
#Source for Lookup table:
#https://geoportal.statistics.gov.uk/datasets/ons::local-authority-district-to-region-april-2020-lookup-in-en/explore
#In excel Find and replace was used to change all instances of 'East of England' to 'East'
LAD20_to_RGN20_Conversion_Table = pd.read_excel(io=Local_Authority_District_to_Region_4)
LAD20_to_RGN20_Conversion_Table
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
      <th>LAD20CD</th>
      <th>LAD20NM</th>
      <th>RGN20CD</th>
      <th>RGN20NM</th>
      <th>FID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>E06000012</td>
      <td>North East Lincolnshire</td>
      <td>E12000003</td>
      <td>Yorkshire and The Humber</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>E06000013</td>
      <td>North Lincolnshire</td>
      <td>E12000003</td>
      <td>Yorkshire and The Humber</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>E06000014</td>
      <td>York</td>
      <td>E12000003</td>
      <td>Yorkshire and The Humber</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>E07000163</td>
      <td>Craven</td>
      <td>E12000003</td>
      <td>Yorkshire and The Humber</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>E07000164</td>
      <td>Hambleton</td>
      <td>E12000003</td>
      <td>Yorkshire and The Humber</td>
      <td>5.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>315</th>
      <td>E06000062</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>South East</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>316</th>
      <td>E06000063</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>South East</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>317</th>
      <td>E06000064</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>South East</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>318</th>
      <td>E06000065</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>South East</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>319</th>
      <td>E06000066</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>South East</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>320 rows × 5 columns</p>
</div>




```python
LAD20_to_RGN20_Conversion_Table['RGN20NM'].unique()

```




    array(['Yorkshire and The Humber', 'East Midlands', 'North East',
           'North West', 'London', 'South East', 'West Midlands',
           'South West', 'East'], dtype=object)




```python
#Join the RGN20NM onto the UK_Population_Density_Raw_Data table
UK_Population_Raw_Data_With_Region = pd.merge(UK_Population_Raw_Data, LAD20_to_RGN20_Conversion_Table, left_on='ladcode23', right_on='LAD20CD', how='left')

#This checks if all rows relating to england have had a RGN20NM successfully allocated
len(UK_Population_Raw_Data_With_Region[UK_Population_Raw_Data_With_Region['RGN20NM'].notna()]) - len(UK_Population_Raw_Data_With_Region[UK_Population_Raw_Data_With_Region['country'] == 'E'])

#This initially returned a difference of 1092
#After manual adjustments this turned to 0
```




    0




```python
Region_Allocation_Erros = UK_Population_Raw_Data_With_Region[
    (UK_Population_Raw_Data_With_Region['RGN20NM'].isna()) &
    (UK_Population_Raw_Data_With_Region['country'] == 'E')]

Region_Allocation_Erros['ladcode23'].unique()

#The output here initially was: 'array(['E06000061', 'E06000062', 'E06000063', 'E06000064', 'E06000065', 'E06000066'], dtype=object)'
#These values where allocated manually in the LAD20_to_RGN20_Conversion_Table excel. The the join was performed again.

```




    array([], dtype=object)




```python
#Group by Region
Population_by_Region_Pivoted = UK_Population_Raw_Data_With_Region.groupby('RGN20NM',as_index=False).agg({
    'population_2011' : 'sum',
    'population_2012' : 'sum',
    'population_2013' : 'sum',
    'population_2014' : 'sum',
    'population_2015' : 'sum',
    'population_2016' : 'sum',
    'population_2017' : 'sum',
    'population_2018' : 'sum',
    'population_2019' : 'sum',
    'population_2020' : 'sum',
    'population_2021' : 'sum',
    'population_2022' : 'sum',
})

Population_by_Region_Pivoted

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
      <th>RGN20NM</th>
      <th>population_2011</th>
      <th>population_2012</th>
      <th>population_2013</th>
      <th>population_2014</th>
      <th>population_2015</th>
      <th>population_2016</th>
      <th>population_2017</th>
      <th>population_2018</th>
      <th>population_2019</th>
      <th>population_2020</th>
      <th>population_2021</th>
      <th>population_2022</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>East</td>
      <td>5862418</td>
      <td>5915033</td>
      <td>5970484</td>
      <td>6039521</td>
      <td>6098262</td>
      <td>6161075</td>
      <td>6202843</td>
      <td>6237376</td>
      <td>6274179</td>
      <td>6297779</td>
      <td>6349646</td>
      <td>6401418</td>
    </tr>
    <tr>
      <th>1</th>
      <td>East Midlands</td>
      <td>3843481</td>
      <td>3867559</td>
      <td>3893081</td>
      <td>3920956</td>
      <td>3948451</td>
      <td>3984949</td>
      <td>4018910</td>
      <td>4045911</td>
      <td>4066985</td>
      <td>4076473</td>
      <td>4094434</td>
      <td>4142077</td>
    </tr>
    <tr>
      <th>2</th>
      <td>London</td>
      <td>8204407</td>
      <td>8320767</td>
      <td>8438987</td>
      <td>8547192</td>
      <td>8659545</td>
      <td>8743651</td>
      <td>8776229</td>
      <td>8833335</td>
      <td>8889743</td>
      <td>8867008</td>
      <td>8804769</td>
      <td>8869043</td>
    </tr>
    <tr>
      <th>3</th>
      <td>North East</td>
      <td>2596441</td>
      <td>2598972</td>
      <td>2603299</td>
      <td>2610482</td>
      <td>2611804</td>
      <td>2618253</td>
      <td>2623787</td>
      <td>2629393</td>
      <td>2636676</td>
      <td>2637426</td>
      <td>2647493</td>
      <td>2682069</td>
    </tr>
    <tr>
      <th>4</th>
      <td>North West</td>
      <td>6556144</td>
      <td>6580509</td>
      <td>6607066</td>
      <td>6642793</td>
      <td>6683933</td>
      <td>6735195</td>
      <td>6777072</td>
      <td>6817175</td>
      <td>6864096</td>
      <td>6882166</td>
      <td>6923363</td>
      <td>7012739</td>
    </tr>
    <tr>
      <th>5</th>
      <td>South East</td>
      <td>10979355</td>
      <td>11073623</td>
      <td>11162565</td>
      <td>11261890</td>
      <td>11354048</td>
      <td>11459199</td>
      <td>11529391</td>
      <td>11588320</td>
      <td>11645604</td>
      <td>11679917</td>
      <td>11775410</td>
      <td>11884007</td>
    </tr>
    <tr>
      <th>6</th>
      <td>South West</td>
      <td>4769250</td>
      <td>4800501</td>
      <td>4835318</td>
      <td>4877847</td>
      <td>4922695</td>
      <td>4972176</td>
      <td>5015771</td>
      <td>5047542</td>
      <td>5077805</td>
      <td>5095680</td>
      <td>5139722</td>
      <td>5189846</td>
    </tr>
    <tr>
      <th>7</th>
      <td>West Midlands</td>
      <td>5608667</td>
      <td>5640325</td>
      <td>5676945</td>
      <td>5716882</td>
      <td>5756287</td>
      <td>5812248</td>
      <td>5855052</td>
      <td>5889328</td>
      <td>5920977</td>
      <td>5931924</td>
      <td>5956226</td>
      <td>6017026</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Yorkshire and The Humber</td>
      <td>4687006</td>
      <td>4709523</td>
      <td>4730941</td>
      <td>4752756</td>
      <td>4773651</td>
      <td>4802288</td>
      <td>4820493</td>
      <td>4836148</td>
      <td>4853991</td>
      <td>4857588</td>
      <td>4863828</td>
      <td>4914317</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Unpivot the year columns

Population_by_Region_and_Year = pd.melt(Population_by_Region_Pivoted, id_vars='RGN20NM')
Population_by_Region_and_Year = Population_by_Region_and_Year.rename(columns={'RGN20NM':'Region','variable':'Year','value':'Population'})
Population_by_Region_and_Year['Year'] = Population_by_Region_and_Year['Year'].apply(lambda x: x.split('_')[1])

Population_by_Region_and_Year
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
      <th>Region</th>
      <th>Year</th>
      <th>Population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>East</td>
      <td>2011</td>
      <td>5862418</td>
    </tr>
    <tr>
      <th>1</th>
      <td>East Midlands</td>
      <td>2011</td>
      <td>3843481</td>
    </tr>
    <tr>
      <th>2</th>
      <td>London</td>
      <td>2011</td>
      <td>8204407</td>
    </tr>
    <tr>
      <th>3</th>
      <td>North East</td>
      <td>2011</td>
      <td>2596441</td>
    </tr>
    <tr>
      <th>4</th>
      <td>North West</td>
      <td>2011</td>
      <td>6556144</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>103</th>
      <td>North West</td>
      <td>2022</td>
      <td>7012739</td>
    </tr>
    <tr>
      <th>104</th>
      <td>South East</td>
      <td>2022</td>
      <td>11884007</td>
    </tr>
    <tr>
      <th>105</th>
      <td>South West</td>
      <td>2022</td>
      <td>5189846</td>
    </tr>
    <tr>
      <th>106</th>
      <td>West Midlands</td>
      <td>2022</td>
      <td>6017026</td>
    </tr>
    <tr>
      <th>107</th>
      <td>Yorkshire and The Humber</td>
      <td>2022</td>
      <td>4914317</td>
    </tr>
  </tbody>
</table>
<p>108 rows × 3 columns</p>
</div>




```python
Population_by_Region_and_Year_Line = px.line(Population_by_Region_and_Year, x= 'Year', y='Population', color='Region')

Population_by_Region_and_Year_Line
Population_by_Region_and_Year_Line.update_layout(
        margin={"r": 10, "t": 170, "l": 10, "b": 10},
    title="Population by Region and Year",
    xaxis_title="Year",
    yaxis_title="Population",
    title_x=0.5,
    title_y=0.95, 
    legend=dict(
        orientation="h", 
        yanchor="bottom", 
        y=1, 
        xanchor="center",  
        x=0.5,
        title = None
    ,font=dict(
        size=26 
    )),
    height = 1000,
    font=dict(
        size=26 
    ))
    
Population_by_Region_and_Year_Line

```




```python
Population_by_Region_and_Year_Pivot = Population_by_Region_and_Year.pivot(index = 'Year', columns= 'Region')
Population_by_Region_and_Year_Pivot['Year'] = Population_by_Region_and_Year_Pivot.index
Population_by_Region_and_Year_Pivot.corr()

#High corrolation with Year so exterapolation is viable
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }

    .dataframe thead tr:last-of-type th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th></th>
      <th colspan="9" halign="left">Population</th>
      <th>Year</th>
    </tr>
    <tr>
      <th></th>
      <th>Region</th>
      <th>East</th>
      <th>East Midlands</th>
      <th>London</th>
      <th>North East</th>
      <th>North West</th>
      <th>South East</th>
      <th>South West</th>
      <th>West Midlands</th>
      <th>Yorkshire and The Humber</th>
      <th></th>
    </tr>
    <tr>
      <th></th>
      <th>Region</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="9" valign="top">Population</th>
      <th>East</th>
      <td>1.000000</td>
      <td>0.995639</td>
      <td>0.952173</td>
      <td>0.920096</td>
      <td>0.984919</td>
      <td>0.999387</td>
      <td>0.997129</td>
      <td>0.996699</td>
      <td>0.994990</td>
      <td>0.992994</td>
    </tr>
    <tr>
      <th>East Midlands</th>
      <td>0.995639</td>
      <td>1.000000</td>
      <td>0.936214</td>
      <td>0.937375</td>
      <td>0.994076</td>
      <td>0.996868</td>
      <td>0.998976</td>
      <td>0.999739</td>
      <td>0.997280</td>
      <td>0.995321</td>
    </tr>
    <tr>
      <th>London</th>
      <td>0.952173</td>
      <td>0.936214</td>
      <td>1.000000</td>
      <td>0.786523</td>
      <td>0.895828</td>
      <td>0.943988</td>
      <td>0.931793</td>
      <td>0.942203</td>
      <td>0.945407</td>
      <td>0.917723</td>
    </tr>
    <tr>
      <th>North East</th>
      <td>0.920096</td>
      <td>0.937375</td>
      <td>0.786523</td>
      <td>1.000000</td>
      <td>0.963696</td>
      <td>0.931897</td>
      <td>0.937396</td>
      <td>0.931297</td>
      <td>0.940663</td>
      <td>0.939365</td>
    </tr>
    <tr>
      <th>North West</th>
      <td>0.984919</td>
      <td>0.994076</td>
      <td>0.895828</td>
      <td>0.963696</td>
      <td>1.000000</td>
      <td>0.988930</td>
      <td>0.994409</td>
      <td>0.992083</td>
      <td>0.989665</td>
      <td>0.994604</td>
    </tr>
    <tr>
      <th>South East</th>
      <td>0.999387</td>
      <td>0.996868</td>
      <td>0.943988</td>
      <td>0.931897</td>
      <td>0.988930</td>
      <td>1.000000</td>
      <td>0.998226</td>
      <td>0.997300</td>
      <td>0.996488</td>
      <td>0.994499</td>
    </tr>
    <tr>
      <th>South West</th>
      <td>0.997129</td>
      <td>0.998976</td>
      <td>0.931793</td>
      <td>0.937396</td>
      <td>0.994409</td>
      <td>0.998226</td>
      <td>1.000000</td>
      <td>0.998837</td>
      <td>0.995347</td>
      <td>0.997157</td>
    </tr>
    <tr>
      <th>West Midlands</th>
      <td>0.996699</td>
      <td>0.999739</td>
      <td>0.942203</td>
      <td>0.931297</td>
      <td>0.992083</td>
      <td>0.997300</td>
      <td>0.998837</td>
      <td>1.000000</td>
      <td>0.997378</td>
      <td>0.994335</td>
    </tr>
    <tr>
      <th>Yorkshire and The Humber</th>
      <td>0.994990</td>
      <td>0.997280</td>
      <td>0.945407</td>
      <td>0.940663</td>
      <td>0.989665</td>
      <td>0.996488</td>
      <td>0.995347</td>
      <td>0.997378</td>
      <td>1.000000</td>
      <td>0.989635</td>
    </tr>
    <tr>
      <th>Year</th>
      <th></th>
      <td>0.992994</td>
      <td>0.995321</td>
      <td>0.917723</td>
      <td>0.939365</td>
      <td>0.994604</td>
      <td>0.994499</td>
      <td>0.997157</td>
      <td>0.994335</td>
      <td>0.989635</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
def extrapolate_population_for_region_v1(region):
    #Filtering to handle each region seperately
    Population_by_Region_and_Year_for_Extrapoloation = Population_by_Region_and_Year[Population_by_Region_and_Year['Region']==region]

    X = Population_by_Region_and_Year_for_Extrapoloation['Year'].values.reshape(-1,1)
    y = Population_by_Region_and_Year_for_Extrapoloation['Population']

    #Fitting the linear regession model
    Population_by_Region_and_Year_Model = LinearRegression()
    Population_by_Region_and_Year_Model.fit(X,y)
    
    #Extrapolating for the years where data is missing
    years_to_extrapolate = np.array([2005, 2006, 2007, 2008, 2009, 2010]).reshape(-1, 1)

    population_predictions = Population_by_Region_and_Year_Model.predict(years_to_extrapolate)
    population_predictions = population_predictions.astype(int)
    
    #Bring the predictions into a dataframe
    Predictions_df = pd.DataFrame({'Year':years_to_extrapolate.flatten(),'Population':Population_by_Region_and_Year_Model.predict(years_to_extrapolate),'Region':region,'Data_Type': 'Extrapolated'})
    
    return Predictions_df
```


```python
Population_by_Region_and_Year['Data_Type'] = "Original"
```


```python
#Create a list of regions for which to loop the extrapolate function
Regions = list(Population_by_Region_and_Year['Region'].unique())

#Concatating all the extrapolated values into one dataframe
Population_by_Region_and_Year_Extrapolated = pd.concat((extrapolate_population_for_region_v1(region) for region in Regions), ignore_index=True)

#Combining the Original data with the extrapolated data
Population_by_Region_and_Year_Combined = pd.concat([Population_by_Region_and_Year,Population_by_Region_and_Year_Extrapolated], ignore_index=True)

```


```python
# Change the datatype of the Year column to be integer (required step to fix later graph)
Population_by_Region_and_Year_Combined['Year'] = Population_by_Region_and_Year_Combined['Year'].astype(int)

Population_by_Region_and_Year_Combined = Population_by_Region_and_Year_Combined.sort_values(by='Year')
```


```python
Population_by_Region_and_Year_Combined[Population_by_Region_and_Year_Combined['Region'] =="Yorkshire and The Humber"]
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
      <th>Region</th>
      <th>Year</th>
      <th>Population</th>
      <th>Data_Type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>156</th>
      <td>Yorkshire and The Humber</td>
      <td>2005</td>
      <td>4.579557e+06</td>
      <td>Extrapolated</td>
    </tr>
    <tr>
      <th>157</th>
      <td>Yorkshire and The Humber</td>
      <td>2006</td>
      <td>4.598744e+06</td>
      <td>Extrapolated</td>
    </tr>
    <tr>
      <th>158</th>
      <td>Yorkshire and The Humber</td>
      <td>2007</td>
      <td>4.617932e+06</td>
      <td>Extrapolated</td>
    </tr>
    <tr>
      <th>159</th>
      <td>Yorkshire and The Humber</td>
      <td>2008</td>
      <td>4.637119e+06</td>
      <td>Extrapolated</td>
    </tr>
    <tr>
      <th>160</th>
      <td>Yorkshire and The Humber</td>
      <td>2009</td>
      <td>4.656306e+06</td>
      <td>Extrapolated</td>
    </tr>
    <tr>
      <th>161</th>
      <td>Yorkshire and The Humber</td>
      <td>2010</td>
      <td>4.675493e+06</td>
      <td>Extrapolated</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Yorkshire and The Humber</td>
      <td>2011</td>
      <td>4.687006e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Yorkshire and The Humber</td>
      <td>2012</td>
      <td>4.709523e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Yorkshire and The Humber</td>
      <td>2013</td>
      <td>4.730941e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Yorkshire and The Humber</td>
      <td>2014</td>
      <td>4.752756e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Yorkshire and The Humber</td>
      <td>2015</td>
      <td>4.773651e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>53</th>
      <td>Yorkshire and The Humber</td>
      <td>2016</td>
      <td>4.802288e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>62</th>
      <td>Yorkshire and The Humber</td>
      <td>2017</td>
      <td>4.820493e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>71</th>
      <td>Yorkshire and The Humber</td>
      <td>2018</td>
      <td>4.836148e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>80</th>
      <td>Yorkshire and The Humber</td>
      <td>2019</td>
      <td>4.853991e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>89</th>
      <td>Yorkshire and The Humber</td>
      <td>2020</td>
      <td>4.857588e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>98</th>
      <td>Yorkshire and The Humber</td>
      <td>2021</td>
      <td>4.863828e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>107</th>
      <td>Yorkshire and The Humber</td>
      <td>2022</td>
      <td>4.914317e+06</td>
      <td>Original</td>
    </tr>
  </tbody>
</table>
</div>




```python
Population_by_Region_and_Year_Combined_Line = px.line(Population_by_Region_and_Year_Combined, x= 'Year', y='Population', color='Region')

Population_by_Region_and_Year_Combined_Line
Population_by_Region_and_Year_Combined_Line.update_layout(
        margin={"r": 10, "t": 170, "l": 10, "b": 10},
    title="Population by Region and Year (Extrapolation Model 1)",
    xaxis_title="Year",
    yaxis_title="Population",
    title_x=0.5,
    title_y=0.95, 
    legend=dict(
        orientation="h", 
        yanchor="bottom", 
        y=1, 
        xanchor="center",  
        x=0.5,
        title = None
    ,font=dict(
        size=26 
    )),
    height = 1000,
    font=dict(
        size=26 
    ))

#Include commentry about the 2010 prediction for London this seems unlikely
```




```python
#Lets check the R^2 value for this way of modeling
```


```python
def extrapolate_population_for_region_r2_model1(region):
    #Filtering to handle each region seperately
    Population_by_Region_and_Year_Region = Population_by_Region_and_Year[Population_by_Region_and_Year['Region']==region]

    X = Population_by_Region_and_Year_Region['Year'].values.reshape(-1,1)
    y = Population_by_Region_and_Year_Region['Population']

    #Fitting the linear regession model
    Population_by_Region_and_Year_Model = LinearRegression()
    Population_by_Region_and_Year_Model.fit(X,y)
    
    R2 = Population_by_Region_and_Year_Model.score(X, y)
    
    return R2
```


```python
def extrapolate_population_for_region_r2_model2(region):
    #Filtering to handle each region seperately
    Population_by_Region_and_Year_Region = Population_by_Region_and_Year[(Population_by_Region_and_Year['Region']==region) &
                                                                            (Population_by_Region_and_Year['Year']!='2020') & 
                                                                            (Population_by_Region_and_Year['Year']!='2021') &
                                                                            (Population_by_Region_and_Year['Year']!='2022')]
    
    X = Population_by_Region_and_Year_Region['Year'].values.reshape(-1,1)
    y = Population_by_Region_and_Year_Region['Population']

    #Fitting the linear regession model
    Population_by_Region_and_Year_Model = LinearRegression()
    Population_by_Region_and_Year_Model.fit(X,y)
    
    R2 = Population_by_Region_and_Year_Model.score(X, y)
    
    return R2
```


```python
R2_outcomes = pd.DataFrame(columns=['Region','Model1_R2','Model2_R2'])
```


```python
for region in Regions:
    new_row_for_r2 = pd.DataFrame({'Region': [region],'Model1_R2':[extrapolate_population_for_region_r2_model1(region)],'Model2_R2':[extrapolate_population_for_region_r2_model2(region)]})
    R2_outcomes = pd.concat([R2_outcomes, new_row_for_r2], ignore_index=True)
    
R2_outcomes
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
      <th>Region</th>
      <th>Model1_R2</th>
      <th>Model2_R2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>East</td>
      <td>0.986038</td>
      <td>0.989760</td>
    </tr>
    <tr>
      <th>1</th>
      <td>East Midlands</td>
      <td>0.990663</td>
      <td>0.996692</td>
    </tr>
    <tr>
      <th>2</th>
      <td>London</td>
      <td>0.842216</td>
      <td>0.967706</td>
    </tr>
    <tr>
      <th>3</th>
      <td>North East</td>
      <td>0.882407</td>
      <td>0.986730</td>
    </tr>
    <tr>
      <th>4</th>
      <td>North West</td>
      <td>0.989237</td>
      <td>0.990682</td>
    </tr>
    <tr>
      <th>5</th>
      <td>South East</td>
      <td>0.989027</td>
      <td>0.991894</td>
    </tr>
    <tr>
      <th>6</th>
      <td>South West</td>
      <td>0.994322</td>
      <td>0.996009</td>
    </tr>
    <tr>
      <th>7</th>
      <td>West Midlands</td>
      <td>0.988701</td>
      <td>0.996162</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Yorkshire and The Humber</td>
      <td>0.979378</td>
      <td>0.996405</td>
    </tr>
  </tbody>
</table>
</div>




```python
R2_outcomes['Percentage_Improved'] = (R2_outcomes['Model2_R2'] / R2_outcomes['Model1_R2'] - 1)*100
R2_outcomes
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
      <th>Region</th>
      <th>Model1_R2</th>
      <th>Model2_R2</th>
      <th>Percentage_Improved</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>East</td>
      <td>0.986038</td>
      <td>0.989760</td>
      <td>0.377469</td>
    </tr>
    <tr>
      <th>1</th>
      <td>East Midlands</td>
      <td>0.990663</td>
      <td>0.996692</td>
      <td>0.608602</td>
    </tr>
    <tr>
      <th>2</th>
      <td>London</td>
      <td>0.842216</td>
      <td>0.967706</td>
      <td>14.899989</td>
    </tr>
    <tr>
      <th>3</th>
      <td>North East</td>
      <td>0.882407</td>
      <td>0.986730</td>
      <td>11.822613</td>
    </tr>
    <tr>
      <th>4</th>
      <td>North West</td>
      <td>0.989237</td>
      <td>0.990682</td>
      <td>0.146073</td>
    </tr>
    <tr>
      <th>5</th>
      <td>South East</td>
      <td>0.989027</td>
      <td>0.991894</td>
      <td>0.289901</td>
    </tr>
    <tr>
      <th>6</th>
      <td>South West</td>
      <td>0.994322</td>
      <td>0.996009</td>
      <td>0.169629</td>
    </tr>
    <tr>
      <th>7</th>
      <td>West Midlands</td>
      <td>0.988701</td>
      <td>0.996162</td>
      <td>0.754630</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Yorkshire and The Humber</td>
      <td>0.979378</td>
      <td>0.996405</td>
      <td>1.738566</td>
    </tr>
  </tbody>
</table>
</div>




```python
def extrapolate_population_for_region_v2(region):
    #Filtering to handle each region seperately
    Population_by_Region_and_Year_for_Extrapoloation2 = Population_by_Region_and_Year[(Population_by_Region_and_Year['Region']==region) &
                                                                            (Population_by_Region_and_Year['Year']!='2020') & 
                                                                            (Population_by_Region_and_Year['Year']!='2021') &
                                                                            (Population_by_Region_and_Year['Year']!='2022')]


    X = Population_by_Region_and_Year_for_Extrapoloation2['Year'].values.reshape(-1,1)
    y = Population_by_Region_and_Year_for_Extrapoloation2['Population']

    #Fitting the linear regession model
    Population_by_Region_and_Year_Model_v2 = LinearRegression()
    Population_by_Region_and_Year_Model_v2.fit(X,y)
    
    #Extrapolating for the years where data is missing
    years_to_extrapolate = np.array([2005, 2006, 2007, 2008, 2009, 2010]).reshape(-1, 1)

    population_predictions = Population_by_Region_and_Year_Model_v2.predict(years_to_extrapolate)
    population_predictions = population_predictions.astype(int)
    
    #Bring the predictions into a dataframe
    Predictions_df = pd.DataFrame({'Year':years_to_extrapolate.flatten(),'Population':Population_by_Region_and_Year_Model_v2.predict(years_to_extrapolate),'Region':region,'Data_Type': 'Extrapolated'})
    
    return Predictions_df
```


```python
#Create a list of regions for which to loop the extrapolate function
Regions = list(Population_by_Region_and_Year['Region'].unique())

#Concatating all the extrapolated values into one dataframe
Population_by_Region_and_Year_Extrapolated_v2 = pd.concat((extrapolate_population_for_region_v2(region) for region in Regions), ignore_index=True)

#Combining the Original data with the extrapolated data
Population_by_Region_and_Year_Combined_v2 = pd.concat([Population_by_Region_and_Year,Population_by_Region_and_Year_Extrapolated_v2], ignore_index=True)

```


```python
# Change the datatype of the Year column to be integer (required step to fix later graph)
Population_by_Region_and_Year_Combined_v2['Year'] = Population_by_Region_and_Year_Combined_v2['Year'].astype(int)

Population_by_Region_and_Year_Combined_v2 = Population_by_Region_and_Year_Combined_v2.sort_values(by='Year')
```


```python
Population_by_Region_and_Year_Combined_v2_Line = px.line(Population_by_Region_and_Year_Combined_v2, x= 'Year', y='Population', color='Region')

Population_by_Region_and_Year_Combined_v2_Line
Population_by_Region_and_Year_Combined_v2_Line.update_layout(
        margin={"r": 10, "t": 170, "l": 10, "b": 10},
    title="Population by Region and Year (Extrapolation Model 2)",
    xaxis_title="Year",
    yaxis_title="Population",
    title_x=0.5,
    title_y=0.95, 
    legend=dict(
        orientation="h", 
        yanchor="bottom", 
        y=1, 
        xanchor="center",  
        x=0.5,
        title = None
    ,font=dict(
        size=26 
    )),
    height = 1000,
    font=dict(
        size=26 
    ))
#Include commentry about the 2010 prediction for London this seems unlikely
```




```python
Population_by_Region_and_Year_Combined_v2[Population_by_Region_and_Year_Combined_v2['Region']=='London']
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
      <th>Region</th>
      <th>Year</th>
      <th>Population</th>
      <th>Data_Type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>120</th>
      <td>London</td>
      <td>2005</td>
      <td>7.743208e+06</td>
      <td>Extrapolated</td>
    </tr>
    <tr>
      <th>121</th>
      <td>London</td>
      <td>2006</td>
      <td>7.829041e+06</td>
      <td>Extrapolated</td>
    </tr>
    <tr>
      <th>122</th>
      <td>London</td>
      <td>2007</td>
      <td>7.914874e+06</td>
      <td>Extrapolated</td>
    </tr>
    <tr>
      <th>123</th>
      <td>London</td>
      <td>2008</td>
      <td>8.000707e+06</td>
      <td>Extrapolated</td>
    </tr>
    <tr>
      <th>124</th>
      <td>London</td>
      <td>2009</td>
      <td>8.086540e+06</td>
      <td>Extrapolated</td>
    </tr>
    <tr>
      <th>125</th>
      <td>London</td>
      <td>2010</td>
      <td>8.172374e+06</td>
      <td>Extrapolated</td>
    </tr>
    <tr>
      <th>2</th>
      <td>London</td>
      <td>2011</td>
      <td>8.204407e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>11</th>
      <td>London</td>
      <td>2012</td>
      <td>8.320767e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>20</th>
      <td>London</td>
      <td>2013</td>
      <td>8.438987e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>29</th>
      <td>London</td>
      <td>2014</td>
      <td>8.547192e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>38</th>
      <td>London</td>
      <td>2015</td>
      <td>8.659545e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>47</th>
      <td>London</td>
      <td>2016</td>
      <td>8.743651e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>56</th>
      <td>London</td>
      <td>2017</td>
      <td>8.776229e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>65</th>
      <td>London</td>
      <td>2018</td>
      <td>8.833335e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>74</th>
      <td>London</td>
      <td>2019</td>
      <td>8.889743e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>83</th>
      <td>London</td>
      <td>2020</td>
      <td>8.867008e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>92</th>
      <td>London</td>
      <td>2021</td>
      <td>8.804769e+06</td>
      <td>Original</td>
    </tr>
    <tr>
      <th>101</th>
      <td>London</td>
      <td>2022</td>
      <td>8.869043e+06</td>
      <td>Original</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Data taken from here:
#https://en.wikipedia.org/wiki/Regions_of_England See citation [28]

Regional_Areas = pd.DataFrame({'Region':['North East','North West','Yorkshire and The Humber', 'East Midlands', 'West Midlands', 'East', 'London', 'South East', 'South West'],'Area km^2':['8581','14108','15404','15624','12998','19116','1572','1972','23836']})

Regional_Areas['Area km^2'] = Regional_Areas['Area km^2'].astype(float)
```


```python
Population_Density_by_Region = Population_by_Region_and_Year_Combined_v2.merge(Regional_Areas, on='Region', how='left')
Population_Density_by_Region['Population_Density(Persons.km^-2)'] = Population_Density_by_Region['Population'] / Population_Density_by_Region['Area km^2']
Population_Density_by_Region
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
      <th>Region</th>
      <th>Year</th>
      <th>Population</th>
      <th>Data_Type</th>
      <th>Area km^2</th>
      <th>Population_Density(Persons.km^-2)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>East</td>
      <td>2005</td>
      <td>5.551186e+06</td>
      <td>Extrapolated</td>
      <td>19116.0</td>
      <td>290.394745</td>
    </tr>
    <tr>
      <th>1</th>
      <td>North East</td>
      <td>2005</td>
      <td>2.564187e+06</td>
      <td>Extrapolated</td>
      <td>8581.0</td>
      <td>298.821453</td>
    </tr>
    <tr>
      <th>2</th>
      <td>London</td>
      <td>2005</td>
      <td>7.743208e+06</td>
      <td>Extrapolated</td>
      <td>1572.0</td>
      <td>4925.704658</td>
    </tr>
    <tr>
      <th>3</th>
      <td>East Midlands</td>
      <td>2005</td>
      <td>3.663689e+06</td>
      <td>Extrapolated</td>
      <td>15624.0</td>
      <td>234.491086</td>
    </tr>
    <tr>
      <th>4</th>
      <td>South West</td>
      <td>2005</td>
      <td>4.519226e+06</td>
      <td>Extrapolated</td>
      <td>23836.0</td>
      <td>189.596679</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>West Midlands</td>
      <td>2022</td>
      <td>6.017026e+06</td>
      <td>Original</td>
      <td>12998.0</td>
      <td>462.919372</td>
    </tr>
    <tr>
      <th>158</th>
      <td>South West</td>
      <td>2022</td>
      <td>5.189846e+06</td>
      <td>Original</td>
      <td>23836.0</td>
      <td>217.731415</td>
    </tr>
    <tr>
      <th>159</th>
      <td>East</td>
      <td>2022</td>
      <td>6.401418e+06</td>
      <td>Original</td>
      <td>19116.0</td>
      <td>334.872254</td>
    </tr>
    <tr>
      <th>160</th>
      <td>North West</td>
      <td>2022</td>
      <td>7.012739e+06</td>
      <td>Original</td>
      <td>14108.0</td>
      <td>497.075347</td>
    </tr>
    <tr>
      <th>161</th>
      <td>North East</td>
      <td>2022</td>
      <td>2.682069e+06</td>
      <td>Original</td>
      <td>8581.0</td>
      <td>312.559026</td>
    </tr>
  </tbody>
</table>
<p>162 rows × 6 columns</p>
</div>



Making Chloropleths

Cite this user:
https://github.com/martinjc/UK-GeoJSON/tree/master
Cite this GeoJSON Specifically:
https://raw.githubusercontent.com/martinjc/UK-GeoJSON/refs/heads/master/json/eurostat/ew/nuts1.json


```python
#Importing the GeoJSON
with urlopen('https://raw.githubusercontent.com/martinjc/UK-GeoJSON/refs/heads/master/json/eurostat/ew/nuts1.json') as response:
    Regions = json.load(response)
    
```


```python
#Printing a list of all Regions in the GeoJSON along with their Location Code
for ref in range(len(Regions['features'])):
    print(f"{Regions['features'][ref]['properties']['NUTS112CD']} / {Regions['features'][ref]['properties']['NUTS112NM']}")
```

    UKC / North East (England)
    UKD / North West (England)
    UKE / Yorkshire and The Humber
    UKF / East Midlands (England)
    UKG / West Midlands (England)
    UKH / East of England
    UKI / London
    UKJ / South East (England)
    UKK / South West (England
    UKL / Wales
    


```python
#Printing a list of all the Location Codes in the Gas_Consumption dataframe
Gas_Consumption['Country or region'].unique()
```




    array(['North East', 'North West', 'Yorkshire and The Humber',
           'East Midlands', 'West Midlands', 'East', 'London', 'South East',
           'South West', 'Yorkshire and the Humber', 'Inner London',
           'Outer London'], dtype=object)




```python
#Defining the Location_Code mapping table
Location_Code_List = [
    ('UKC', 'North East', 'North East'),
    ('UKD', 'North West', 'North West'),
    ('UKE', 'Yorkshire and The Humber', 'Yorkshire and The Humber'),
    ('UKE', 'Yorkshire and the Humber', 'Yorkshire and The Humber'),
    ('UKF', 'East Midlands', 'East Midlands'),
    ('UKG', 'West Midlands', 'West Midlands'),
    ('UKH', 'East', 'East'),
    ('UKI', 'Inner London', 'London'),
    ('UKI', 'Outer London', 'London'),
    ('UKI', 'London', 'London'),
    ('UKJ', 'South East', 'South East'),
    ('UKK', 'South West', 'South West')
]

Location_Code = pd.DataFrame(Location_Code_List, columns=['Location_Code', 'In_Region', 'Region'])
Location_Code
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
      <th>Location_Code</th>
      <th>In_Region</th>
      <th>Region</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>UKC</td>
      <td>North East</td>
      <td>North East</td>
    </tr>
    <tr>
      <th>1</th>
      <td>UKD</td>
      <td>North West</td>
      <td>North West</td>
    </tr>
    <tr>
      <th>2</th>
      <td>UKE</td>
      <td>Yorkshire and The Humber</td>
      <td>Yorkshire and The Humber</td>
    </tr>
    <tr>
      <th>3</th>
      <td>UKE</td>
      <td>Yorkshire and the Humber</td>
      <td>Yorkshire and The Humber</td>
    </tr>
    <tr>
      <th>4</th>
      <td>UKF</td>
      <td>East Midlands</td>
      <td>East Midlands</td>
    </tr>
    <tr>
      <th>5</th>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>West Midlands</td>
    </tr>
    <tr>
      <th>6</th>
      <td>UKH</td>
      <td>East</td>
      <td>East</td>
    </tr>
    <tr>
      <th>7</th>
      <td>UKI</td>
      <td>Inner London</td>
      <td>London</td>
    </tr>
    <tr>
      <th>8</th>
      <td>UKI</td>
      <td>Outer London</td>
      <td>London</td>
    </tr>
    <tr>
      <th>9</th>
      <td>UKI</td>
      <td>London</td>
      <td>London</td>
    </tr>
    <tr>
      <th>10</th>
      <td>UKJ</td>
      <td>South East</td>
      <td>South East</td>
    </tr>
    <tr>
      <th>11</th>
      <td>UKK</td>
      <td>South West</td>
      <td>South West</td>
    </tr>
  </tbody>
</table>
</div>




```python
#This step reclassifies regions into the desired regions found in the Geo-JSON
Gas_Consumption_Cleaned = Gas_Consumption.merge(Location_Code, right_on='In_Region', left_on='Country or region', how='left')

#This step combines values for Outer and inner london, which have both been classified as London
Gas_Consumption_Cleaned = Gas_Consumption_Cleaned.groupby(['Year','Location_Code'],as_index=False).agg({
    'Region': 'max',
    'Number of meters\n(thousands):\nDomestic\n' : 'sum',
     'Number of meters\n(thousands):\nNon-Domestic': 'sum',
     'Number of meters\n(thousands):\nAll meters': 'sum',
     'Total consumption\n(GWh):\nDomestic\n': 'sum',
     'Total consumption\n(GWh):\nNon-Domestic': 'sum',
     'Total consumption\n(GWh):\nAll meters': 'sum',
     'Mean consumption\n(kWh per meter):\nDomestic\n': 'sum',
     'Mean consumption\n(kWh per meter):\nNon-Domestic': 'sum',
     'Mean consumption\n(kWh per meter):\nAll meters' : 'sum'
})
Gas_Consumption_Cleaned
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
      <th>Year</th>
      <th>Location_Code</th>
      <th>Region</th>
      <th>Number of meters\n(thousands):\nDomestic\n</th>
      <th>Number of meters\n(thousands):\nNon-Domestic</th>
      <th>Number of meters\n(thousands):\nAll meters</th>
      <th>Total consumption\n(GWh):\nDomestic\n</th>
      <th>Total consumption\n(GWh):\nNon-Domestic</th>
      <th>Total consumption\n(GWh):\nAll meters</th>
      <th>Mean consumption\n(kWh per meter):\nDomestic\n</th>
      <th>Mean consumption\n(kWh per meter):\nNon-Domestic</th>
      <th>Mean consumption\n(kWh per meter):\nAll meters</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005</td>
      <td>UKC</td>
      <td>North East</td>
      <td>1037.403</td>
      <td>16.219</td>
      <td>1053.622</td>
      <td>20710.658157</td>
      <td>13952.158200</td>
      <td>34662.816357</td>
      <td>19963.946660</td>
      <td>860235.415254</td>
      <td>32898.721132</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2005</td>
      <td>UKD</td>
      <td>North West</td>
      <td>2747.967</td>
      <td>48.871</td>
      <td>2796.838</td>
      <td>53390.873953</td>
      <td>35926.287793</td>
      <td>89317.161746</td>
      <td>19429.226753</td>
      <td>735124.875550</td>
      <td>31935.050134</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005</td>
      <td>UKE</td>
      <td>Yorkshire and The Humber</td>
      <td>1989.970</td>
      <td>36.997</td>
      <td>2026.967</td>
      <td>39024.143235</td>
      <td>30648.965991</td>
      <td>69673.109226</td>
      <td>19610.417863</td>
      <td>828417.601184</td>
      <td>34373.085120</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005</td>
      <td>UKF</td>
      <td>East Midlands</td>
      <td>1620.786</td>
      <td>28.502</td>
      <td>1649.288</td>
      <td>31469.021304</td>
      <td>18936.399391</td>
      <td>50405.420695</td>
      <td>19415.901485</td>
      <td>664388.442601</td>
      <td>30561.927750</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2005</td>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>1984.918</td>
      <td>36.397</td>
      <td>2021.315</td>
      <td>37726.189511</td>
      <td>22962.792215</td>
      <td>60688.981726</td>
      <td>19006.422185</td>
      <td>630897.937055</td>
      <td>30024.504704</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>2022</td>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>2221.428</td>
      <td>21.914</td>
      <td>2243.342</td>
      <td>25247.105176</td>
      <td>15406.577700</td>
      <td>40653.682876</td>
      <td>11365.259273</td>
      <td>703047.262016</td>
      <td>18121.928300</td>
    </tr>
    <tr>
      <th>158</th>
      <td>2022</td>
      <td>UKH</td>
      <td>East</td>
      <td>2217.992</td>
      <td>20.366</td>
      <td>2238.358</td>
      <td>25143.154335</td>
      <td>13711.245874</td>
      <td>38854.400209</td>
      <td>11335.998658</td>
      <td>673241.965715</td>
      <td>17358.438734</td>
    </tr>
    <tr>
      <th>159</th>
      <td>2022</td>
      <td>UKI</td>
      <td>London</td>
      <td>3001.866</td>
      <td>39.524</td>
      <td>3041.390</td>
      <td>35792.967053</td>
      <td>19576.692142</td>
      <td>55369.659195</td>
      <td>11923.572555</td>
      <td>495311.510530</td>
      <td>18205.379512</td>
    </tr>
    <tr>
      <th>160</th>
      <td>2022</td>
      <td>UKJ</td>
      <td>South East</td>
      <td>3428.042</td>
      <td>35.662</td>
      <td>3463.704</td>
      <td>39604.299045</td>
      <td>16541.969255</td>
      <td>56146.268301</td>
      <td>11553.037870</td>
      <td>463854.221730</td>
      <td>16209.892156</td>
    </tr>
    <tr>
      <th>161</th>
      <td>2022</td>
      <td>UKK</td>
      <td>South West</td>
      <td>1994.866</td>
      <td>18.140</td>
      <td>2013.006</td>
      <td>19620.178140</td>
      <td>10846.932730</td>
      <td>30467.110870</td>
      <td>9835.336379</td>
      <td>597956.600307</td>
      <td>15135.131674</td>
    </tr>
  </tbody>
</table>
<p>162 rows × 12 columns</p>
</div>




```python
#Checking that all regions have been successfully allocated a Location_Code
Gas_Consumption_Cleaned[Gas_Consumption_Cleaned['Location_Code'].isnull()]
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
      <th>Year</th>
      <th>Location_Code</th>
      <th>Region</th>
      <th>Number of meters\n(thousands):\nDomestic\n</th>
      <th>Number of meters\n(thousands):\nNon-Domestic</th>
      <th>Number of meters\n(thousands):\nAll meters</th>
      <th>Total consumption\n(GWh):\nDomestic\n</th>
      <th>Total consumption\n(GWh):\nNon-Domestic</th>
      <th>Total consumption\n(GWh):\nAll meters</th>
      <th>Mean consumption\n(kWh per meter):\nDomestic\n</th>
      <th>Mean consumption\n(kWh per meter):\nNon-Domestic</th>
      <th>Mean consumption\n(kWh per meter):\nAll meters</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
#Converts the field 'Year' to an integer to allow following join
Gas_Consumption_Cleaned['Year'] = Gas_Consumption_Cleaned['Year'].astype(int)
```


```python
#Here I am merging the Population density data onto the Gas energy consumption data
Gas_Consumption_Cleaned_PopD = Gas_Consumption_Cleaned.merge(Population_Density_by_Region, on=['Region','Year'], how='left')

Gas_Consumption_Cleaned_PopD[Gas_Consumption_Cleaned_PopD['Year'] == 2010]
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
      <th>Year</th>
      <th>Location_Code</th>
      <th>Region</th>
      <th>Number of meters\n(thousands):\nDomestic\n</th>
      <th>Number of meters\n(thousands):\nNon-Domestic</th>
      <th>Number of meters\n(thousands):\nAll meters</th>
      <th>Total consumption\n(GWh):\nDomestic\n</th>
      <th>Total consumption\n(GWh):\nNon-Domestic</th>
      <th>Total consumption\n(GWh):\nAll meters</th>
      <th>Mean consumption\n(kWh per meter):\nDomestic\n</th>
      <th>Mean consumption\n(kWh per meter):\nNon-Domestic</th>
      <th>Mean consumption\n(kWh per meter):\nAll meters</th>
      <th>Population</th>
      <th>Data_Type</th>
      <th>Area km^2</th>
      <th>Population_Density(Persons.km^-2)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>45</th>
      <td>2010</td>
      <td>UKC</td>
      <td>North East</td>
      <td>1080.768</td>
      <td>11.923</td>
      <td>1092.691</td>
      <td>16620.453603</td>
      <td>9478.474684</td>
      <td>26098.928287</td>
      <td>15378.373160</td>
      <td>794973.973329</td>
      <td>23885.003434</td>
      <td>2.589266e+06</td>
      <td>Extrapolated</td>
      <td>8581.0</td>
      <td>301.744092</td>
    </tr>
    <tr>
      <th>46</th>
      <td>2010</td>
      <td>UKD</td>
      <td>North West</td>
      <td>2833.501</td>
      <td>34.101</td>
      <td>2867.602</td>
      <td>43159.855574</td>
      <td>26673.463523</td>
      <td>69833.319097</td>
      <td>15231.988827</td>
      <td>782190.068414</td>
      <td>24352.514434</td>
      <td>6.498146e+06</td>
      <td>Extrapolated</td>
      <td>14108.0</td>
      <td>460.600117</td>
    </tr>
    <tr>
      <th>47</th>
      <td>2010</td>
      <td>UKE</td>
      <td>Yorkshire and The Humber</td>
      <td>2083.114</td>
      <td>25.578</td>
      <td>2108.692</td>
      <td>32486.133640</td>
      <td>22541.149332</td>
      <td>55027.282972</td>
      <td>15594.985987</td>
      <td>881270.988037</td>
      <td>26095.457740</td>
      <td>4.667718e+06</td>
      <td>Extrapolated</td>
      <td>15404.0</td>
      <td>303.019842</td>
    </tr>
    <tr>
      <th>48</th>
      <td>2010</td>
      <td>UKF</td>
      <td>East Midlands</td>
      <td>1714.820</td>
      <td>19.904</td>
      <td>1734.724</td>
      <td>26448.515431</td>
      <td>14815.583187</td>
      <td>41264.098618</td>
      <td>15423.493679</td>
      <td>744352.049186</td>
      <td>23787.126147</td>
      <td>3.809082e+06</td>
      <td>Extrapolated</td>
      <td>15624.0</td>
      <td>243.796871</td>
    </tr>
    <tr>
      <th>49</th>
      <td>2010</td>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>2068.149</td>
      <td>24.960</td>
      <td>2093.109</td>
      <td>31161.449497</td>
      <td>16743.357037</td>
      <td>47904.806534</td>
      <td>15067.313572</td>
      <td>670807.573598</td>
      <td>22886.914410</td>
      <td>5.560093e+06</td>
      <td>Extrapolated</td>
      <td>12998.0</td>
      <td>427.765291</td>
    </tr>
    <tr>
      <th>50</th>
      <td>2010</td>
      <td>UKH</td>
      <td>East</td>
      <td>2004.529</td>
      <td>23.978</td>
      <td>2028.507</td>
      <td>30751.229491</td>
      <td>17361.387992</td>
      <td>48112.617483</td>
      <td>15340.875333</td>
      <td>724054.883310</td>
      <td>23718.240796</td>
      <td>5.817881e+06</td>
      <td>Extrapolated</td>
      <td>19116.0</td>
      <td>304.346169</td>
    </tr>
    <tr>
      <th>51</th>
      <td>2010</td>
      <td>UKI</td>
      <td>London</td>
      <td>2987.734</td>
      <td>44.259</td>
      <td>3031.993</td>
      <td>44701.219243</td>
      <td>22721.974812</td>
      <td>67423.194055</td>
      <td>14961.579325</td>
      <td>513386.538602</td>
      <td>22237.252545</td>
      <td>8.172374e+06</td>
      <td>Extrapolated</td>
      <td>1572.0</td>
      <td>5198.710966</td>
    </tr>
    <tr>
      <th>52</th>
      <td>2010</td>
      <td>UKJ</td>
      <td>South East</td>
      <td>3130.822</td>
      <td>41.931</td>
      <td>3172.753</td>
      <td>48158.250292</td>
      <td>21256.898786</td>
      <td>69415.149078</td>
      <td>15381.982844</td>
      <td>506949.483342</td>
      <td>21878.522872</td>
      <td>1.091100e+07</td>
      <td>Extrapolated</td>
      <td>1972.0</td>
      <td>5532.959150</td>
    </tr>
    <tr>
      <th>53</th>
      <td>2010</td>
      <td>UKK</td>
      <td>South West</td>
      <td>1779.916</td>
      <td>20.640</td>
      <td>1800.556</td>
      <td>23919.620625</td>
      <td>11367.803472</td>
      <td>35287.424097</td>
      <td>13438.623297</td>
      <td>550765.672093</td>
      <td>19598.070872</td>
      <td>4.721775e+06</td>
      <td>Extrapolated</td>
      <td>23836.0</td>
      <td>198.094253</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Calculated Gas consumption per person for each region and year
Gas_Consumption_Cleaned_PopD['Domestic_Gas_Consumption(GWh/person)'] = Gas_Consumption_Cleaned_PopD['Total consumption\n(GWh):\nDomestic\n']/Gas_Consumption_Cleaned_PopD['Population']
Gas_Consumption_Cleaned_PopD['Commercial_Gas_Consumption(GWh/person)'] = Gas_Consumption_Cleaned_PopD['Total consumption\n(GWh):\nNon-Domestic']/Gas_Consumption_Cleaned_PopD['Population']
Gas_Consumption_Cleaned_PopD['Total_Gas_Consumption(GWh/person)'] = Gas_Consumption_Cleaned_PopD['Total consumption\n(GWh):\nAll meters']/Gas_Consumption_Cleaned_PopD['Population']

#Please note only domestic values where used in visualisation as the scope of this project centres around the domestic cost of energy
Gas_Consumption_Cleaned_PopD
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
      <th>Year</th>
      <th>Location_Code</th>
      <th>Region</th>
      <th>Number of meters\n(thousands):\nDomestic\n</th>
      <th>Number of meters\n(thousands):\nNon-Domestic</th>
      <th>Number of meters\n(thousands):\nAll meters</th>
      <th>Total consumption\n(GWh):\nDomestic\n</th>
      <th>Total consumption\n(GWh):\nNon-Domestic</th>
      <th>Total consumption\n(GWh):\nAll meters</th>
      <th>Mean consumption\n(kWh per meter):\nDomestic\n</th>
      <th>Mean consumption\n(kWh per meter):\nNon-Domestic</th>
      <th>Mean consumption\n(kWh per meter):\nAll meters</th>
      <th>Population</th>
      <th>Data_Type</th>
      <th>Area km^2</th>
      <th>Population_Density(Persons.km^-2)</th>
      <th>Domestic_Gas_Consumption(GWh/person)</th>
      <th>Commercial_Gas_Consumption(GWh/person)</th>
      <th>Total_Gas_Consumption(GWh/person)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005</td>
      <td>UKC</td>
      <td>North East</td>
      <td>1037.403</td>
      <td>16.219</td>
      <td>1053.622</td>
      <td>20710.658157</td>
      <td>13952.158200</td>
      <td>34662.816357</td>
      <td>19963.946660</td>
      <td>860235.415254</td>
      <td>32898.721132</td>
      <td>2.564187e+06</td>
      <td>Extrapolated</td>
      <td>8581.0</td>
      <td>298.821453</td>
      <td>0.008077</td>
      <td>0.005441</td>
      <td>0.013518</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2005</td>
      <td>UKD</td>
      <td>North West</td>
      <td>2747.967</td>
      <td>48.871</td>
      <td>2796.838</td>
      <td>53390.873953</td>
      <td>35926.287793</td>
      <td>89317.161746</td>
      <td>19429.226753</td>
      <td>735124.875550</td>
      <td>31935.050134</td>
      <td>6.300295e+06</td>
      <td>Extrapolated</td>
      <td>14108.0</td>
      <td>446.576040</td>
      <td>0.008474</td>
      <td>0.005702</td>
      <td>0.014177</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005</td>
      <td>UKE</td>
      <td>Yorkshire and The Humber</td>
      <td>1989.970</td>
      <td>36.997</td>
      <td>2026.967</td>
      <td>39024.143235</td>
      <td>30648.965991</td>
      <td>69673.109226</td>
      <td>19610.417863</td>
      <td>828417.601184</td>
      <td>34373.085120</td>
      <td>4.561347e+06</td>
      <td>Extrapolated</td>
      <td>15404.0</td>
      <td>296.114433</td>
      <td>0.008555</td>
      <td>0.006719</td>
      <td>0.015275</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005</td>
      <td>UKF</td>
      <td>East Midlands</td>
      <td>1620.786</td>
      <td>28.502</td>
      <td>1649.288</td>
      <td>31469.021304</td>
      <td>18936.399391</td>
      <td>50405.420695</td>
      <td>19415.901485</td>
      <td>664388.442601</td>
      <td>30561.927750</td>
      <td>3.663689e+06</td>
      <td>Extrapolated</td>
      <td>15624.0</td>
      <td>234.491086</td>
      <td>0.008589</td>
      <td>0.005169</td>
      <td>0.013758</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2005</td>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>1984.918</td>
      <td>36.397</td>
      <td>2021.315</td>
      <td>37726.189511</td>
      <td>22962.792215</td>
      <td>60688.981726</td>
      <td>19006.422185</td>
      <td>630897.937055</td>
      <td>30024.504704</td>
      <td>5.356108e+06</td>
      <td>Extrapolated</td>
      <td>12998.0</td>
      <td>412.071665</td>
      <td>0.007044</td>
      <td>0.004287</td>
      <td>0.011331</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>2022</td>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>2221.428</td>
      <td>21.914</td>
      <td>2243.342</td>
      <td>25247.105176</td>
      <td>15406.577700</td>
      <td>40653.682876</td>
      <td>11365.259273</td>
      <td>703047.262016</td>
      <td>18121.928300</td>
      <td>6.017026e+06</td>
      <td>Original</td>
      <td>12998.0</td>
      <td>462.919372</td>
      <td>0.004196</td>
      <td>0.002560</td>
      <td>0.006756</td>
    </tr>
    <tr>
      <th>158</th>
      <td>2022</td>
      <td>UKH</td>
      <td>East</td>
      <td>2217.992</td>
      <td>20.366</td>
      <td>2238.358</td>
      <td>25143.154335</td>
      <td>13711.245874</td>
      <td>38854.400209</td>
      <td>11335.998658</td>
      <td>673241.965715</td>
      <td>17358.438734</td>
      <td>6.401418e+06</td>
      <td>Original</td>
      <td>19116.0</td>
      <td>334.872254</td>
      <td>0.003928</td>
      <td>0.002142</td>
      <td>0.006070</td>
    </tr>
    <tr>
      <th>159</th>
      <td>2022</td>
      <td>UKI</td>
      <td>London</td>
      <td>3001.866</td>
      <td>39.524</td>
      <td>3041.390</td>
      <td>35792.967053</td>
      <td>19576.692142</td>
      <td>55369.659195</td>
      <td>11923.572555</td>
      <td>495311.510530</td>
      <td>18205.379512</td>
      <td>8.869043e+06</td>
      <td>Original</td>
      <td>1572.0</td>
      <td>5641.884860</td>
      <td>0.004036</td>
      <td>0.002207</td>
      <td>0.006243</td>
    </tr>
    <tr>
      <th>160</th>
      <td>2022</td>
      <td>UKJ</td>
      <td>South East</td>
      <td>3428.042</td>
      <td>35.662</td>
      <td>3463.704</td>
      <td>39604.299045</td>
      <td>16541.969255</td>
      <td>56146.268301</td>
      <td>11553.037870</td>
      <td>463854.221730</td>
      <td>16209.892156</td>
      <td>1.188401e+07</td>
      <td>Original</td>
      <td>1972.0</td>
      <td>6026.372718</td>
      <td>0.003333</td>
      <td>0.001392</td>
      <td>0.004725</td>
    </tr>
    <tr>
      <th>161</th>
      <td>2022</td>
      <td>UKK</td>
      <td>South West</td>
      <td>1994.866</td>
      <td>18.140</td>
      <td>2013.006</td>
      <td>19620.178140</td>
      <td>10846.932730</td>
      <td>30467.110870</td>
      <td>9835.336379</td>
      <td>597956.600307</td>
      <td>15135.131674</td>
      <td>5.189846e+06</td>
      <td>Original</td>
      <td>23836.0</td>
      <td>217.731415</td>
      <td>0.003780</td>
      <td>0.002090</td>
      <td>0.005871</td>
    </tr>
  </tbody>
</table>
<p>162 rows × 19 columns</p>
</div>




```python
#Setting the maximum and minimum values as variables
Max_Domestic_Gas_Consumption = Gas_Consumption_Cleaned_PopD['Domestic_Gas_Consumption(GWh/person)'].max()
Min_Domestic_Gas_Consumption = Gas_Consumption_Cleaned_PopD['Domestic_Gas_Consumption(GWh/person)'].min()
```


```python
#defining the graphing function
def create_domestic_choropleth(year):
    fig = px.choropleth(Gas_Consumption_Cleaned_PopD.loc[Gas_Consumption_Cleaned_PopD['Year'] == year]
                        ,locations='Location_Code'
                        ,geojson=Regions
                        ,color ='Domestic_Gas_Consumption(GWh/person)'
                        ,featureidkey="properties.NUTS112CD"
                        ,hover_name='Region'
                        ,range_color = (Min_Domestic_Gas_Consumption, Max_Domestic_Gas_Consumption))
    fig.update_geos(
        fitbounds="geojson",
       # projection_scale = 100,
        visible=False,
        projection_type="orthographic" 
        )
    fig.update_layout(
    margin={"r": 0, "t": 50, "l": 0, "b": 0}, 
    title={
        'text': f"{year}",  
        'y': 0.95, 
        'x': 0.5,  
        'xanchor': 'center',
        'yanchor': 'top'},
    coloraxis_colorbar_title=None,
    coloraxis_colorbar=dict(
            len=0.8,
            thickness=15 
        ),
        width=800,
        height=650,
        font=dict(
        size=26 
    ))

    return fig

#Showing "Domestic Gas Consumption in {year} (GWh/person)"

```


```python
#Defining the years to graph as a variable
Years_to_Graph = list(range(2005,2023))
Years_to_Graph
```




    [2005,
     2006,
     2007,
     2008,
     2009,
     2010,
     2011,
     2012,
     2013,
     2014,
     2015,
     2016,
     2017,
     2018,
     2019,
     2020,
     2021,
     2022]




```python
#Looping the create a gragh function for each year
#Choropleth hidden here to support GitHub upload
figures = [create_domestic_choropleth(year) for year in Years_to_Graph]

'''for fig in figures:
    fig.show()'''
```




    'for fig in figures:\n    fig.show()'



This is the Importation of Energy Bills data


```python
#The importation of Energy price data

#Importing the Historical data sheet (Pre-2020 data)
Energy_Price_Raw_S1 = pd.read_excel(io=Domestic_Energy_Bills_5, sheet_name=r'2.2.3 (Historic Consumption)', header=16, usecols='A:F')
#Filtering to only include the average value for each region for each year
Energy_Price_Raw_S1 = Energy_Price_Raw_S1[Energy_Price_Raw_S1['Bill range\n[Note 1]'] == 'Average']
#Selecting only the needed columns
Energy_Price_Raw_S1 = Energy_Price_Raw_S1[['Year', 'PES area', 'Credit: Unit cost (Pence per kWh)']]
#Renaming columns to fit with the same naming convention as the recent dataset
Energy_Price_Raw_S1.rename(columns={'PES area':'Region','Credit: Unit cost (Pence per kWh)':'Average Unit Cost (Pence per kWh)'}, inplace = True)
#Removing Yearly totals. These will be calculated later using an aggregate function
Energy_Price_Raw_S1 = Energy_Price_Raw_S1[Energy_Price_Raw_S1['Region'] != 'United Kingdom']
#(!!)Removing years later than 2017 as these are included in both datasets. Include dialog around the difference in these years. The latest dataset was chosen under the assumption that the more recent data would be better. justify this
Energy_Price_Raw_S1 = Energy_Price_Raw_S1[Energy_Price_Raw_S1['Year'] < 2017]

#Importing the Historical data sheet (Post-2018 data)
Energy_Price_Raw_S2 = pd.read_excel(io=Domestic_Energy_Bills_5, sheet_name=r'2.2.3', header=11,usecols='A:C')
#Removing Yearly totals. These will be calculated later using an aggregate function
Energy_Price_Raw_S2 = Energy_Price_Raw_S2[Energy_Price_Raw_S2['Region'] != 'United Kingdom']
Energy_Price_Raw_S2.rename(columns={'Credit: Unit cost (Pence per kWh)':'Average Unit Cost (Pence per kWh)'}, inplace = True)

Energy_Price = pd.concat([Energy_Price_Raw_S1,Energy_Price_Raw_S2])

```


```python
Energy_Price[Energy_Price['Year'] == 2000]
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
      <th>Year</th>
      <th>Region</th>
      <th>Average Unit Cost (Pence per kWh)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>63</th>
      <td>2000</td>
      <td>Northern Scotland</td>
      <td>8.25</td>
    </tr>
    <tr>
      <th>65</th>
      <td>2000</td>
      <td>Northern Ireland</td>
      <td>9.35</td>
    </tr>
    <tr>
      <th>67</th>
      <td>2000</td>
      <td>West Midlands</td>
      <td>7.44</td>
    </tr>
    <tr>
      <th>70</th>
      <td>2000</td>
      <td>South East</td>
      <td>7.38</td>
    </tr>
    <tr>
      <th>73</th>
      <td>2000</td>
      <td>South Wales</td>
      <td>8.64</td>
    </tr>
    <tr>
      <th>76</th>
      <td>2000</td>
      <td>Southern Scotland</td>
      <td>8.11</td>
    </tr>
    <tr>
      <th>79</th>
      <td>2000</td>
      <td>Eastern</td>
      <td>7.31</td>
    </tr>
    <tr>
      <th>82</th>
      <td>2000</td>
      <td>Yorkshire</td>
      <td>7.57</td>
    </tr>
    <tr>
      <th>85</th>
      <td>2000</td>
      <td>Merseyside &amp; North Wales</td>
      <td>8.21</td>
    </tr>
    <tr>
      <th>88</th>
      <td>2000</td>
      <td>London</td>
      <td>7.64</td>
    </tr>
    <tr>
      <th>91</th>
      <td>2000</td>
      <td>North West</td>
      <td>7.54</td>
    </tr>
    <tr>
      <th>94</th>
      <td>2000</td>
      <td>North East</td>
      <td>8.00</td>
    </tr>
    <tr>
      <th>97</th>
      <td>2000</td>
      <td>East Midlands</td>
      <td>7.26</td>
    </tr>
    <tr>
      <th>100</th>
      <td>2000</td>
      <td>South West</td>
      <td>8.22</td>
    </tr>
    <tr>
      <th>103</th>
      <td>2000</td>
      <td>Southern</td>
      <td>7.70</td>
    </tr>
  </tbody>
</table>
</div>




```python
#ChatGPT Generated code to replace 'South West' with the average of 'Southern' and 'South West'
# Group by Year to calculate averages
for year in Energy_Price['Year'].unique():
    # Filter the current year's data for 'Southern' and 'South West'
    southern_value = Energy_Price.loc[(Energy_Price['Year'] == year) & (Energy_Price['Region'] == 'Southern'), 'Average Unit Cost (Pence per kWh)'].values
    south_west_value = Energy_Price.loc[(Energy_Price['Year'] == year) & (Energy_Price['Region'] == 'South West'), 'Average Unit Cost (Pence per kWh)'].values

    if southern_value.size > 0 and south_west_value.size > 0:
        # Calculate the average
        avg_value = (southern_value[0] + south_west_value[0]) / 2

        # Update 'South West' value
        Energy_Price.loc[(Energy_Price['Year'] == year) & (Energy_Price['Region'] == 'South West'), 'Average Unit Cost (Pence per kWh)'] = avg_value
```


```python
#ChatGPT Generated code to replace 'South East' with the average of 'Southern' and 'South East'
# Group by Year to calculate averages
for year in Energy_Price['Year'].unique():
    # Filter the current year's data for 'Southern' and 'South East'
    southern_value = Energy_Price.loc[(Energy_Price['Year'] == year) & (Energy_Price['Region'] == 'Southern'), 'Average Unit Cost (Pence per kWh)'].values
    south_east_value = Energy_Price.loc[(Energy_Price['Year'] == year) & (Energy_Price['Region'] == 'South East'), 'Average Unit Cost (Pence per kWh)'].values

    if southern_value.size > 0 and south_east_value.size > 0:
        # Calculate the average
        avg_value = (southern_value[0] + south_east_value[0]) / 2

        # Update 'South East' value
        Energy_Price.loc[(Energy_Price['Year'] == year) & (Energy_Price['Region'] == 'South East'), 'Average Unit Cost (Pence per kWh)'] = avg_value
```


```python
#This process above assumes that each region is of roughly the same magnitude. Interpolation was used
```


```python
Energy_Price.rename(columns={'Region':'In_Region'}, inplace = True)
```


```python
#Standardisation of naming convention for Region
Energy_Price['In_Region'].unique()
Energy_Price_Region_Correction = pd.DataFrame({'In_Region':['West Midlands',
       'South East', 'Eastern',
       'Yorkshire', 'London', 'North West',
       'North East', 'East Midlands', 'South West'], 'Region':['West Midlands','South East','East','Yorkshire and The Humber','London', 'North West',
       'North East', 'East Midlands', 'South West']})
Energy_Price_Region_Correction
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
      <th>In_Region</th>
      <th>Region</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>West Midlands</td>
      <td>West Midlands</td>
    </tr>
    <tr>
      <th>1</th>
      <td>South East</td>
      <td>South East</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Eastern</td>
      <td>East</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Yorkshire</td>
      <td>Yorkshire and The Humber</td>
    </tr>
    <tr>
      <th>4</th>
      <td>London</td>
      <td>London</td>
    </tr>
    <tr>
      <th>5</th>
      <td>North West</td>
      <td>North West</td>
    </tr>
    <tr>
      <th>6</th>
      <td>North East</td>
      <td>North East</td>
    </tr>
    <tr>
      <th>7</th>
      <td>East Midlands</td>
      <td>East Midlands</td>
    </tr>
    <tr>
      <th>8</th>
      <td>South West</td>
      <td>South West</td>
    </tr>
  </tbody>
</table>
</div>




```python
Energy_Price_Region = pd.merge(Energy_Price, Energy_Price_Region_Correction, on='In_Region', how='left')
Energy_Price_Region = Energy_Price_Region[Energy_Price_Region['Region'].notnull()]
```


```python
#Data processing to allow for the line graph of energy price by year
Energy_Price_Aggregate = Energy_Price.groupby('Year',as_index=False).agg({'Average Unit Cost (Pence per kWh)' : 'mean'})
Energy_Price_Aggregate
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
      <th>Year</th>
      <th>Average Unit Cost (Pence per kWh)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1998</td>
      <td>8.258000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1999</td>
      <td>8.143000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2000</td>
      <td>7.901333</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2001</td>
      <td>7.737667</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2002</td>
      <td>7.725000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2003</td>
      <td>7.757576</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2004</td>
      <td>8.028283</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2005</td>
      <td>8.797980</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2006</td>
      <td>10.333333</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2007</td>
      <td>11.501813</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2008</td>
      <td>13.246108</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2009</td>
      <td>13.750564</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2010</td>
      <td>13.354118</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2011</td>
      <td>14.415625</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2012</td>
      <td>15.329657</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2013</td>
      <td>15.949624</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2014</td>
      <td>16.490626</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2015</td>
      <td>16.346747</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2016</td>
      <td>16.509944</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2017</td>
      <td>17.800095</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2018</td>
      <td>19.247067</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2019</td>
      <td>20.853823</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2020</td>
      <td>20.916758</td>
    </tr>
    <tr>
      <th>23</th>
      <td>2021</td>
      <td>22.718208</td>
    </tr>
    <tr>
      <th>24</th>
      <td>2022</td>
      <td>33.564966</td>
    </tr>
    <tr>
      <th>25</th>
      <td>2023</td>
      <td>36.695593</td>
    </tr>
  </tbody>
</table>
</div>



Now to bring in the Gas inmport cost data


```python
#Importation of the Price of Fuel Imports data
Energy_Generation_Imports_Raw = pd.read_excel(io=Energy_Import_Costs_6, sheet_name=r'3.2.1 (Annual real)', header=12, usecols='A:J')
Energy_Generation_Imports_Raw.rename(columns={'Major power producers: Coal (pence per kWh)\n[Note 1]':'Coal (pence per kWh)','Major power producers: Oil (pence per kWh)\n[Note 2, 3]':'Oil (pence per kWh)','Major power producers: Natural gas (pence per kWh)\n[Note 4]':'Natural Gas (pence per kWh)'}, inplace = True)
Energy_Generation_Imports_Subset = Energy_Generation_Imports_Raw[['Year','Coal (pence per kWh)', 'Oil (pence per kWh)','Natural Gas (pence per kWh)']]
Energy_Generation_Imports_Subset = Energy_Generation_Imports_Subset[Energy_Generation_Imports_Raw['Year'] >= 1998]

Energy_Generation_Imports_Subset
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
      <th>Year</th>
      <th>Coal (pence per kWh)</th>
      <th>Oil (pence per kWh)</th>
      <th>Natural Gas (pence per kWh)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>8</th>
      <td>1998</td>
      <td>0.544375</td>
      <td>0.774538</td>
      <td>0.848242</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1999</td>
      <td>0.513870</td>
      <td>0.911555</td>
      <td>0.782009</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2000</td>
      <td>0.512963</td>
      <td>1.273934</td>
      <td>0.750103</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2001</td>
      <td>0.551299</td>
      <td>1.220321</td>
      <td>0.824940</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2002</td>
      <td>0.498163</td>
      <td>1.289745</td>
      <td>0.740489</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2003</td>
      <td>0.463550</td>
      <td>1.555741</td>
      <td>0.810822</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2004</td>
      <td>0.521652</td>
      <td>1.395152</td>
      <td>0.881483</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2005</td>
      <td>0.559014</td>
      <td>2.172904</td>
      <td>1.141708</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2006</td>
      <td>0.572821</td>
      <td>2.314728</td>
      <td>1.405394</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2007</td>
      <td>0.604027</td>
      <td>2.120755</td>
      <td>1.322306</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2008</td>
      <td>0.930995</td>
      <td>2.456412</td>
      <td>1.701349</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2009</td>
      <td>0.764055</td>
      <td>2.256922</td>
      <td>1.424737</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2010</td>
      <td>0.869646</td>
      <td>3.485674</td>
      <td>1.461134</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2011</td>
      <td>1.083788</td>
      <td>4.321027</td>
      <td>1.872670</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2012</td>
      <td>0.879180</td>
      <td>4.623866</td>
      <td>2.058533</td>
    </tr>
    <tr>
      <th>23</th>
      <td>2013</td>
      <td>0.794566</td>
      <td>4.236485</td>
      <td>2.169998</td>
    </tr>
    <tr>
      <th>24</th>
      <td>2014</td>
      <td>0.724438</td>
      <td>3.779470</td>
      <td>1.760189</td>
    </tr>
    <tr>
      <th>25</th>
      <td>2015</td>
      <td>0.619805</td>
      <td>2.503178</td>
      <td>1.467249</td>
    </tr>
    <tr>
      <th>26</th>
      <td>2016</td>
      <td>0.669809</td>
      <td>2.171225</td>
      <td>1.158163</td>
    </tr>
    <tr>
      <th>27</th>
      <td>2017</td>
      <td>0.894947</td>
      <td>2.749002</td>
      <td>1.357619</td>
    </tr>
    <tr>
      <th>28</th>
      <td>2018</td>
      <td>0.913079</td>
      <td>3.370556</td>
      <td>1.683321</td>
    </tr>
    <tr>
      <th>29</th>
      <td>2019</td>
      <td>0.706770</td>
      <td>3.433348</td>
      <td>1.215103</td>
    </tr>
    <tr>
      <th>30</th>
      <td>2020</td>
      <td>0.685702</td>
      <td>2.770059</td>
      <td>0.969033</td>
    </tr>
    <tr>
      <th>31</th>
      <td>2021</td>
      <td>1.474959</td>
      <td>3.720295</td>
      <td>2.377165</td>
    </tr>
    <tr>
      <th>32</th>
      <td>2022</td>
      <td>2.830942</td>
      <td>5.878026</td>
      <td>4.862319</td>
    </tr>
    <tr>
      <th>33</th>
      <td>2023</td>
      <td>1.775227</td>
      <td>4.273539</td>
      <td>4.904535</td>
    </tr>
  </tbody>
</table>
</div>




```python
Energy_Vs_Import_Price = Energy_Generation_Imports_Subset.merge(Energy_Price_Aggregate, on='Year')
#Energy_Vs_Import_Price['Log Unit Cost'] = np.log(Energy_Vs_Import_Price['Credit: Unit cost (Pence per kWh)'])
#Energy_Vs_Import_Price['Log Import Cost'] = np.log(Energy_Vs_Import_Price['Major power producers: Oil (pence per kWh)\n[Note 2, 3]'])
Energy_Vs_Import_Price
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
      <th>Year</th>
      <th>Coal (pence per kWh)</th>
      <th>Oil (pence per kWh)</th>
      <th>Natural Gas (pence per kWh)</th>
      <th>Average Unit Cost (Pence per kWh)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1998</td>
      <td>0.544375</td>
      <td>0.774538</td>
      <td>0.848242</td>
      <td>8.258000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1999</td>
      <td>0.513870</td>
      <td>0.911555</td>
      <td>0.782009</td>
      <td>8.143000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2000</td>
      <td>0.512963</td>
      <td>1.273934</td>
      <td>0.750103</td>
      <td>7.901333</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2001</td>
      <td>0.551299</td>
      <td>1.220321</td>
      <td>0.824940</td>
      <td>7.737667</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2002</td>
      <td>0.498163</td>
      <td>1.289745</td>
      <td>0.740489</td>
      <td>7.725000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2003</td>
      <td>0.463550</td>
      <td>1.555741</td>
      <td>0.810822</td>
      <td>7.757576</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2004</td>
      <td>0.521652</td>
      <td>1.395152</td>
      <td>0.881483</td>
      <td>8.028283</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2005</td>
      <td>0.559014</td>
      <td>2.172904</td>
      <td>1.141708</td>
      <td>8.797980</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2006</td>
      <td>0.572821</td>
      <td>2.314728</td>
      <td>1.405394</td>
      <td>10.333333</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2007</td>
      <td>0.604027</td>
      <td>2.120755</td>
      <td>1.322306</td>
      <td>11.501813</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2008</td>
      <td>0.930995</td>
      <td>2.456412</td>
      <td>1.701349</td>
      <td>13.246108</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2009</td>
      <td>0.764055</td>
      <td>2.256922</td>
      <td>1.424737</td>
      <td>13.750564</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2010</td>
      <td>0.869646</td>
      <td>3.485674</td>
      <td>1.461134</td>
      <td>13.354118</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2011</td>
      <td>1.083788</td>
      <td>4.321027</td>
      <td>1.872670</td>
      <td>14.415625</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2012</td>
      <td>0.879180</td>
      <td>4.623866</td>
      <td>2.058533</td>
      <td>15.329657</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2013</td>
      <td>0.794566</td>
      <td>4.236485</td>
      <td>2.169998</td>
      <td>15.949624</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2014</td>
      <td>0.724438</td>
      <td>3.779470</td>
      <td>1.760189</td>
      <td>16.490626</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2015</td>
      <td>0.619805</td>
      <td>2.503178</td>
      <td>1.467249</td>
      <td>16.346747</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2016</td>
      <td>0.669809</td>
      <td>2.171225</td>
      <td>1.158163</td>
      <td>16.509944</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2017</td>
      <td>0.894947</td>
      <td>2.749002</td>
      <td>1.357619</td>
      <td>17.800095</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2018</td>
      <td>0.913079</td>
      <td>3.370556</td>
      <td>1.683321</td>
      <td>19.247067</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2019</td>
      <td>0.706770</td>
      <td>3.433348</td>
      <td>1.215103</td>
      <td>20.853823</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2020</td>
      <td>0.685702</td>
      <td>2.770059</td>
      <td>0.969033</td>
      <td>20.916758</td>
    </tr>
    <tr>
      <th>23</th>
      <td>2021</td>
      <td>1.474959</td>
      <td>3.720295</td>
      <td>2.377165</td>
      <td>22.718208</td>
    </tr>
    <tr>
      <th>24</th>
      <td>2022</td>
      <td>2.830942</td>
      <td>5.878026</td>
      <td>4.862319</td>
      <td>33.564966</td>
    </tr>
    <tr>
      <th>25</th>
      <td>2023</td>
      <td>1.775227</td>
      <td>4.273539</td>
      <td>4.904535</td>
      <td>36.695593</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Median Income by region and region Adjusted for inflation
Median_Income_Raw = pd.read_excel(io=Average_Earning_7, sheet_name='FT weekly pay by home region', header=4, nrows=9, skiprows=[5,6,7], usecols='B:E,G:H, J:N, P:Y, AA:AC')
Median_Income_Raw.rename(columns={'Unnamed: 1': 'Region'}, inplace=True)
Median_Income_Raw
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
      <th>Region</th>
      <th>2002</th>
      <th>2003</th>
      <th>2004</th>
      <th>2005</th>
      <th>2006</th>
      <th>2007</th>
      <th>2008</th>
      <th>2009</th>
      <th>2010</th>
      <th>...</th>
      <th>2015</th>
      <th>2016</th>
      <th>2017</th>
      <th>2018</th>
      <th>2019</th>
      <th>2020</th>
      <th>2021</th>
      <th>2022</th>
      <th>2023</th>
      <th>2024</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>North East</td>
      <td>343.2</td>
      <td>350.5</td>
      <td>372.7</td>
      <td>383.3</td>
      <td>392.9</td>
      <td>401.0</td>
      <td>421.7</td>
      <td>438.5</td>
      <td>443.4</td>
      <td>...</td>
      <td>485.6</td>
      <td>492.4</td>
      <td>504.1</td>
      <td>511.1</td>
      <td>531.4</td>
      <td>525.2</td>
      <td>546.8</td>
      <td>581.4</td>
      <td>617.4</td>
      <td>661.2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>North West</td>
      <td>370.2</td>
      <td>383.2</td>
      <td>399.7</td>
      <td>409.5</td>
      <td>422.0</td>
      <td>433.7</td>
      <td>451.3</td>
      <td>460.0</td>
      <td>471.0</td>
      <td>...</td>
      <td>491.5</td>
      <td>502.5</td>
      <td>514.5</td>
      <td>529.8</td>
      <td>555.8</td>
      <td>558.1</td>
      <td>578.0</td>
      <td>604.4</td>
      <td>653.3</td>
      <td>696.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Yorkshire and The Humber</td>
      <td>360.0</td>
      <td>375.5</td>
      <td>394.9</td>
      <td>400.0</td>
      <td>414.9</td>
      <td>425.6</td>
      <td>444.3</td>
      <td>452.6</td>
      <td>462.5</td>
      <td>...</td>
      <td>480.6</td>
      <td>498.3</td>
      <td>502.3</td>
      <td>520.4</td>
      <td>540.8</td>
      <td>539.7</td>
      <td>568.5</td>
      <td>594.5</td>
      <td>634.7</td>
      <td>674.8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>East Midlands</td>
      <td>369.6</td>
      <td>385.7</td>
      <td>398.2</td>
      <td>412.2</td>
      <td>425.3</td>
      <td>430.0</td>
      <td>450.2</td>
      <td>460.2</td>
      <td>469.8</td>
      <td>...</td>
      <td>491.0</td>
      <td>501.5</td>
      <td>516.7</td>
      <td>529.9</td>
      <td>547.5</td>
      <td>562.5</td>
      <td>573.4</td>
      <td>604.3</td>
      <td>644.1</td>
      <td>684.1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>West Midlands</td>
      <td>366.0</td>
      <td>378.9</td>
      <td>397.2</td>
      <td>404.7</td>
      <td>419.9</td>
      <td>431.1</td>
      <td>449.8</td>
      <td>456.8</td>
      <td>469.2</td>
      <td>...</td>
      <td>492.1</td>
      <td>507.6</td>
      <td>517.1</td>
      <td>535.5</td>
      <td>550.8</td>
      <td>551.7</td>
      <td>581.8</td>
      <td>615.0</td>
      <td>657.9</td>
      <td>689.9</td>
    </tr>
    <tr>
      <th>5</th>
      <td>East</td>
      <td>415.9</td>
      <td>431.7</td>
      <td>453.0</td>
      <td>456.7</td>
      <td>469.4</td>
      <td>479.9</td>
      <td>499.0</td>
      <td>509.5</td>
      <td>523.3</td>
      <td>...</td>
      <td>550.6</td>
      <td>569.5</td>
      <td>574.9</td>
      <td>589.4</td>
      <td>610.2</td>
      <td>607.6</td>
      <td>573.4</td>
      <td>670.0</td>
      <td>709.5</td>
      <td>763.5</td>
    </tr>
    <tr>
      <th>6</th>
      <td>London</td>
      <td>479.9</td>
      <td>496.3</td>
      <td>514.7</td>
      <td>526.7</td>
      <td>538.9</td>
      <td>555.9</td>
      <td>581.5</td>
      <td>598.2</td>
      <td>606.4</td>
      <td>...</td>
      <td>620.8</td>
      <td>631.8</td>
      <td>654.1</td>
      <td>670.8</td>
      <td>699.3</td>
      <td>714.3</td>
      <td>728.4</td>
      <td>766.6</td>
      <td>804.9</td>
      <td>853.4</td>
    </tr>
    <tr>
      <th>7</th>
      <td>South East</td>
      <td>435.1</td>
      <td>451.0</td>
      <td>468.7</td>
      <td>468.9</td>
      <td>488.0</td>
      <td>502.3</td>
      <td>524.8</td>
      <td>536.6</td>
      <td>547.8</td>
      <td>...</td>
      <td>574.9</td>
      <td>581.8</td>
      <td>595.9</td>
      <td>614.9</td>
      <td>636.3</td>
      <td>629.0</td>
      <td>660.1</td>
      <td>689.0</td>
      <td>728.3</td>
      <td>779.2</td>
    </tr>
    <tr>
      <th>8</th>
      <td>South West</td>
      <td>367.1</td>
      <td>383.9</td>
      <td>400.7</td>
      <td>406.0</td>
      <td>422.6</td>
      <td>432.6</td>
      <td>451.9</td>
      <td>460.0</td>
      <td>468.3</td>
      <td>...</td>
      <td>498.3</td>
      <td>513.4</td>
      <td>527.0</td>
      <td>537.6</td>
      <td>560.9</td>
      <td>558.9</td>
      <td>577.3</td>
      <td>622.0</td>
      <td>667.5</td>
      <td>700.8</td>
    </tr>
  </tbody>
</table>
<p>9 rows × 24 columns</p>
</div>




```python
# Unpivot the Median_Income_Raw df
Median_Income = pd.melt(
    Median_Income_Raw, 
    id_vars=['Region'], 
    var_name='Year',   
    value_name='Median_Income' )

#Convert the weekly income into salary
Median_Income['Salary'] = Median_Income['Median_Income']*52
```


```python
Median_Income_by_Year = Median_Income.groupby('Year', as_index=False)['Salary'].mean()
Median_Income_by_Year
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
      <th>Year</th>
      <th>Salary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2002</td>
      <td>20262.666667</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2003</td>
      <td>21012.044444</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2004</td>
      <td>21954.400000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005</td>
      <td>22348.444444</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2006</td>
      <td>23075.866667</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2007</td>
      <td>23643.244444</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2008</td>
      <td>24697.111111</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2009</td>
      <td>25262.755556</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2010</td>
      <td>25778.711111</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2011</td>
      <td>25916.222222</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2012</td>
      <td>26091.866667</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2013</td>
      <td>26578.355556</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2014</td>
      <td>26722.800000</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2015</td>
      <td>27071.200000</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2016</td>
      <td>27726.400000</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2017</td>
      <td>28349.244444</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2018</td>
      <td>29116.533333</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2019</td>
      <td>30235.111111</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2020</td>
      <td>30316.000000</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2021</td>
      <td>31128.933333</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2022</td>
      <td>33206.044444</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2023</td>
      <td>35346.133333</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2024</td>
      <td>37572.311111</td>
    </tr>
  </tbody>
</table>
</div>




```python
Energy_Vs_Import_Price_Sal = Energy_Vs_Import_Price.merge(Median_Income_by_Year, on=['Year'], how='left')
Energy_Vs_Import_Price_Sal = Energy_Vs_Import_Price_Sal[Energy_Vs_Import_Price_Sal['Salary'].notnull()]
Energy_Vs_Import_Price_Sal = Energy_Vs_Import_Price_Sal.reset_index()
Energy_Vs_Import_Price_Sal = Energy_Vs_Import_Price_Sal.drop(columns=['index'])
Energy_Vs_Import_Price_Sal["Salary (£'000)"] = Energy_Vs_Import_Price_Sal['Salary']/1000
Energy_Vs_Import_Price_Sal
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
      <th>Year</th>
      <th>Coal (pence per kWh)</th>
      <th>Oil (pence per kWh)</th>
      <th>Natural Gas (pence per kWh)</th>
      <th>Average Unit Cost (Pence per kWh)</th>
      <th>Salary</th>
      <th>Salary (£'000)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2002</td>
      <td>0.498163</td>
      <td>1.289745</td>
      <td>0.740489</td>
      <td>7.725000</td>
      <td>20262.666667</td>
      <td>20.262667</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2003</td>
      <td>0.463550</td>
      <td>1.555741</td>
      <td>0.810822</td>
      <td>7.757576</td>
      <td>21012.044444</td>
      <td>21.012044</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2004</td>
      <td>0.521652</td>
      <td>1.395152</td>
      <td>0.881483</td>
      <td>8.028283</td>
      <td>21954.400000</td>
      <td>21.954400</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005</td>
      <td>0.559014</td>
      <td>2.172904</td>
      <td>1.141708</td>
      <td>8.797980</td>
      <td>22348.444444</td>
      <td>22.348444</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2006</td>
      <td>0.572821</td>
      <td>2.314728</td>
      <td>1.405394</td>
      <td>10.333333</td>
      <td>23075.866667</td>
      <td>23.075867</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2007</td>
      <td>0.604027</td>
      <td>2.120755</td>
      <td>1.322306</td>
      <td>11.501813</td>
      <td>23643.244444</td>
      <td>23.643244</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2008</td>
      <td>0.930995</td>
      <td>2.456412</td>
      <td>1.701349</td>
      <td>13.246108</td>
      <td>24697.111111</td>
      <td>24.697111</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2009</td>
      <td>0.764055</td>
      <td>2.256922</td>
      <td>1.424737</td>
      <td>13.750564</td>
      <td>25262.755556</td>
      <td>25.262756</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2010</td>
      <td>0.869646</td>
      <td>3.485674</td>
      <td>1.461134</td>
      <td>13.354118</td>
      <td>25778.711111</td>
      <td>25.778711</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2011</td>
      <td>1.083788</td>
      <td>4.321027</td>
      <td>1.872670</td>
      <td>14.415625</td>
      <td>25916.222222</td>
      <td>25.916222</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2012</td>
      <td>0.879180</td>
      <td>4.623866</td>
      <td>2.058533</td>
      <td>15.329657</td>
      <td>26091.866667</td>
      <td>26.091867</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2013</td>
      <td>0.794566</td>
      <td>4.236485</td>
      <td>2.169998</td>
      <td>15.949624</td>
      <td>26578.355556</td>
      <td>26.578356</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2014</td>
      <td>0.724438</td>
      <td>3.779470</td>
      <td>1.760189</td>
      <td>16.490626</td>
      <td>26722.800000</td>
      <td>26.722800</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2015</td>
      <td>0.619805</td>
      <td>2.503178</td>
      <td>1.467249</td>
      <td>16.346747</td>
      <td>27071.200000</td>
      <td>27.071200</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2016</td>
      <td>0.669809</td>
      <td>2.171225</td>
      <td>1.158163</td>
      <td>16.509944</td>
      <td>27726.400000</td>
      <td>27.726400</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2017</td>
      <td>0.894947</td>
      <td>2.749002</td>
      <td>1.357619</td>
      <td>17.800095</td>
      <td>28349.244444</td>
      <td>28.349244</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2018</td>
      <td>0.913079</td>
      <td>3.370556</td>
      <td>1.683321</td>
      <td>19.247067</td>
      <td>29116.533333</td>
      <td>29.116533</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2019</td>
      <td>0.706770</td>
      <td>3.433348</td>
      <td>1.215103</td>
      <td>20.853823</td>
      <td>30235.111111</td>
      <td>30.235111</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2020</td>
      <td>0.685702</td>
      <td>2.770059</td>
      <td>0.969033</td>
      <td>20.916758</td>
      <td>30316.000000</td>
      <td>30.316000</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2021</td>
      <td>1.474959</td>
      <td>3.720295</td>
      <td>2.377165</td>
      <td>22.718208</td>
      <td>31128.933333</td>
      <td>31.128933</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2022</td>
      <td>2.830942</td>
      <td>5.878026</td>
      <td>4.862319</td>
      <td>33.564966</td>
      <td>33206.044444</td>
      <td>33.206044</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2023</td>
      <td>1.775227</td>
      <td>4.273539</td>
      <td>4.904535</td>
      <td>36.695593</td>
      <td>35346.133333</td>
      <td>35.346133</td>
    </tr>
  </tbody>
</table>
</div>




```python
Energy_Price_Vs_Salary_Fig = px.line(Energy_Vs_Import_Price_Sal
                                     , x='Year'
                                     , y=['Average Unit Cost (Pence per kWh)','Oil (pence per kWh)', 'Natural Gas (pence per kWh)', 'Coal (pence per kWh)' ]
                                     ,color_discrete_map= { 
                                                             
                                                                
                                                                'Average Unit Cost (Pence per kWh)': 'green',
                                                                'Oil (pence per kWh)': 'grey',
                                                                'Coal (pence per kWh)': 'lightgrey',
                                                                'Natural Gas (pence per kWh)': 'purple'})
Energy_Price_Vs_Salary_Fig.update_layout(
        margin={"r": 10, "t": 130, "l": 10, "b": 10},
    title="Unit Cost of Energy and Import Cost of Fuel (2002-2023)",
    xaxis_title="Year",
    yaxis_title="",
    title_x=0.5,
    title_y=0.95, 
    legend=dict(
        orientation="h", 
        yanchor="bottom", 
        y=1, 
        xanchor="center",  
        x=0.5,
        title = None
    ,font=dict(
        size=26 
    )),
    height = 1000,
    font=dict(
        size=26 
    ))
#Energy_Vs_Import_Price_Fig = px.line(Energy_Vs_Import_Price, x='Year', y=['Oil (pence per kWh)','Average Unit Cost (Pence per kWh)'])
Energy_Price_Vs_Salary_Fig
```




```python
Energy_Price_Vs_Salary_Fig = px.line(Energy_Vs_Import_Price_Sal
                                     , x='Year'
                                     , y=["Salary (£'000)",'Average Unit Cost (Pence per kWh)','Oil (pence per kWh)', 'Natural Gas (pence per kWh)', 'Coal (pence per kWh)' ]
                                     ,color_discrete_map= { 
                                                                "Salary (£'000)":'Orange',
                                                                'Average Unit Cost (Pence per kWh)': 'green',
                                                                'Oil (pence per kWh)': 'grey',
                                                                'Coal (pence per kWh)': 'lightgrey',
                                                                'Natural Gas (pence per kWh)': 'purple'})
Energy_Price_Vs_Salary_Fig.update_layout(
        margin={"r": 10, "t": 170, "l": 10, "b": 10},
    title="Salary Against Unit Cost of Energy and Import Cost of Fuel (2002-2023)",
    xaxis_title="Year",
    yaxis_title="",
    title_x=0.5,
    title_y=0.95, 
    legend=dict(
        orientation="h", 
        yanchor="bottom", 
        y=1, 
        xanchor="center",  
        x=0.5,
        title = None
    ,font=dict(
        size=26 
    )),
    height = 1000,
    font=dict(
        size=26 
    ))
#Energy_Vs_Import_Price_Fig = px.line(Energy_Vs_Import_Price, x='Year', y=['Oil (pence per kWh)','Average Unit Cost (Pence per kWh)'])
Energy_Price_Vs_Salary_Fig
```



Up next do corrolation matrixs on raw data
do a time series corrolation on the graph values



```python
#Energy_Generation_Imports_Subset_Matrix = Energy_Generation_Imports_Subset.corr()
#plt.imshow(Energy_Generation_Imports_Subset_Matrix, cmap = 'Blues')
Energy_Vs_Import_Price_Matrix = Energy_Vs_Import_Price.corr()

sns.heatmap(Energy_Vs_Import_Price_Matrix, cmap="Reds", annot=True)

plt.title("Correlation Between Import Price of Fuel and Average Unit Cost of Energy (1998-2023)", pad=20)

plt.show()

```


    
![png](Gas%20and%20Energy%20DS%20Project%20with%20Outputs_files/Gas%20and%20Energy%20DS%20Project%20with%20Outputs_75_0.png)
    



```python
Years_to_Graph = list(range(2005,2023))
Years_to_Graph
```




    [2005,
     2006,
     2007,
     2008,
     2009,
     2010,
     2011,
     2012,
     2013,
     2014,
     2015,
     2016,
     2017,
     2018,
     2019,
     2020,
     2021,
     2022]




```python
Gas_Consumption_Cleaned_PopD_MedIncome = Gas_Consumption_Cleaned_PopD.merge(Median_Income, on=['Region','Year'], how='left')
Gas_Consumption_Cleaned_PopD_MedIncome['Domestic_Gas_Consumption_to_Median_Salary(GWh/person/£)'] = Gas_Consumption_Cleaned_PopD_MedIncome['Domestic_Gas_Consumption(GWh/person)']/Gas_Consumption_Cleaned_PopD_MedIncome['Salary']
Gas_Consumption_Cleaned_PopD_MedIncome
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
      <th>Year</th>
      <th>Location_Code</th>
      <th>Region</th>
      <th>Number of meters\n(thousands):\nDomestic\n</th>
      <th>Number of meters\n(thousands):\nNon-Domestic</th>
      <th>Number of meters\n(thousands):\nAll meters</th>
      <th>Total consumption\n(GWh):\nDomestic\n</th>
      <th>Total consumption\n(GWh):\nNon-Domestic</th>
      <th>Total consumption\n(GWh):\nAll meters</th>
      <th>Mean consumption\n(kWh per meter):\nDomestic\n</th>
      <th>...</th>
      <th>Population</th>
      <th>Data_Type</th>
      <th>Area km^2</th>
      <th>Population_Density(Persons.km^-2)</th>
      <th>Domestic_Gas_Consumption(GWh/person)</th>
      <th>Commercial_Gas_Consumption(GWh/person)</th>
      <th>Total_Gas_Consumption(GWh/person)</th>
      <th>Median_Income</th>
      <th>Salary</th>
      <th>Domestic_Gas_Consumption_to_Median_Salary(GWh/person/£)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005</td>
      <td>UKC</td>
      <td>North East</td>
      <td>1037.403</td>
      <td>16.219</td>
      <td>1053.622</td>
      <td>20710.658157</td>
      <td>13952.158200</td>
      <td>34662.816357</td>
      <td>19963.946660</td>
      <td>...</td>
      <td>2.564187e+06</td>
      <td>Extrapolated</td>
      <td>8581.0</td>
      <td>298.821453</td>
      <td>0.008077</td>
      <td>0.005441</td>
      <td>0.013518</td>
      <td>383.3</td>
      <td>19931.6</td>
      <td>4.052304e-07</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2005</td>
      <td>UKD</td>
      <td>North West</td>
      <td>2747.967</td>
      <td>48.871</td>
      <td>2796.838</td>
      <td>53390.873953</td>
      <td>35926.287793</td>
      <td>89317.161746</td>
      <td>19429.226753</td>
      <td>...</td>
      <td>6.300295e+06</td>
      <td>Extrapolated</td>
      <td>14108.0</td>
      <td>446.576040</td>
      <td>0.008474</td>
      <td>0.005702</td>
      <td>0.014177</td>
      <td>409.5</td>
      <td>21294.0</td>
      <td>3.979687e-07</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005</td>
      <td>UKE</td>
      <td>Yorkshire and The Humber</td>
      <td>1989.970</td>
      <td>36.997</td>
      <td>2026.967</td>
      <td>39024.143235</td>
      <td>30648.965991</td>
      <td>69673.109226</td>
      <td>19610.417863</td>
      <td>...</td>
      <td>4.561347e+06</td>
      <td>Extrapolated</td>
      <td>15404.0</td>
      <td>296.114433</td>
      <td>0.008555</td>
      <td>0.006719</td>
      <td>0.015275</td>
      <td>400.0</td>
      <td>20800.0</td>
      <td>4.113173e-07</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005</td>
      <td>UKF</td>
      <td>East Midlands</td>
      <td>1620.786</td>
      <td>28.502</td>
      <td>1649.288</td>
      <td>31469.021304</td>
      <td>18936.399391</td>
      <td>50405.420695</td>
      <td>19415.901485</td>
      <td>...</td>
      <td>3.663689e+06</td>
      <td>Extrapolated</td>
      <td>15624.0</td>
      <td>234.491086</td>
      <td>0.008589</td>
      <td>0.005169</td>
      <td>0.013758</td>
      <td>412.2</td>
      <td>21434.4</td>
      <td>4.007314e-07</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2005</td>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>1984.918</td>
      <td>36.397</td>
      <td>2021.315</td>
      <td>37726.189511</td>
      <td>22962.792215</td>
      <td>60688.981726</td>
      <td>19006.422185</td>
      <td>...</td>
      <td>5.356108e+06</td>
      <td>Extrapolated</td>
      <td>12998.0</td>
      <td>412.071665</td>
      <td>0.007044</td>
      <td>0.004287</td>
      <td>0.011331</td>
      <td>404.7</td>
      <td>21044.4</td>
      <td>3.347011e-07</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>2022</td>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>2221.428</td>
      <td>21.914</td>
      <td>2243.342</td>
      <td>25247.105176</td>
      <td>15406.577700</td>
      <td>40653.682876</td>
      <td>11365.259273</td>
      <td>...</td>
      <td>6.017026e+06</td>
      <td>Original</td>
      <td>12998.0</td>
      <td>462.919372</td>
      <td>0.004196</td>
      <td>0.002560</td>
      <td>0.006756</td>
      <td>615.0</td>
      <td>31980.0</td>
      <td>1.312053e-07</td>
    </tr>
    <tr>
      <th>158</th>
      <td>2022</td>
      <td>UKH</td>
      <td>East</td>
      <td>2217.992</td>
      <td>20.366</td>
      <td>2238.358</td>
      <td>25143.154335</td>
      <td>13711.245874</td>
      <td>38854.400209</td>
      <td>11335.998658</td>
      <td>...</td>
      <td>6.401418e+06</td>
      <td>Original</td>
      <td>19116.0</td>
      <td>334.872254</td>
      <td>0.003928</td>
      <td>0.002142</td>
      <td>0.006070</td>
      <td>670.0</td>
      <td>34840.0</td>
      <td>1.127367e-07</td>
    </tr>
    <tr>
      <th>159</th>
      <td>2022</td>
      <td>UKI</td>
      <td>London</td>
      <td>3001.866</td>
      <td>39.524</td>
      <td>3041.390</td>
      <td>35792.967053</td>
      <td>19576.692142</td>
      <td>55369.659195</td>
      <td>11923.572555</td>
      <td>...</td>
      <td>8.869043e+06</td>
      <td>Original</td>
      <td>1572.0</td>
      <td>5641.884860</td>
      <td>0.004036</td>
      <td>0.002207</td>
      <td>0.006243</td>
      <td>766.6</td>
      <td>39863.2</td>
      <td>1.012392e-07</td>
    </tr>
    <tr>
      <th>160</th>
      <td>2022</td>
      <td>UKJ</td>
      <td>South East</td>
      <td>3428.042</td>
      <td>35.662</td>
      <td>3463.704</td>
      <td>39604.299045</td>
      <td>16541.969255</td>
      <td>56146.268301</td>
      <td>11553.037870</td>
      <td>...</td>
      <td>1.188401e+07</td>
      <td>Original</td>
      <td>1972.0</td>
      <td>6026.372718</td>
      <td>0.003333</td>
      <td>0.001392</td>
      <td>0.004725</td>
      <td>689.0</td>
      <td>35828.0</td>
      <td>9.301583e-08</td>
    </tr>
    <tr>
      <th>161</th>
      <td>2022</td>
      <td>UKK</td>
      <td>South West</td>
      <td>1994.866</td>
      <td>18.140</td>
      <td>2013.006</td>
      <td>19620.178140</td>
      <td>10846.932730</td>
      <td>30467.110870</td>
      <td>9835.336379</td>
      <td>...</td>
      <td>5.189846e+06</td>
      <td>Original</td>
      <td>23836.0</td>
      <td>217.731415</td>
      <td>0.003780</td>
      <td>0.002090</td>
      <td>0.005871</td>
      <td>622.0</td>
      <td>32344.0</td>
      <td>1.168839e-07</td>
    </tr>
  </tbody>
</table>
<p>162 rows × 22 columns</p>
</div>




```python
Energy_Vs_Import_Price_Sal = Energy_Vs_Import_Price.merge(Median_Income, on=['Year'], how='left')
Energy_Vs_Import_Price_Sal
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
      <th>Year</th>
      <th>Coal (pence per kWh)</th>
      <th>Oil (pence per kWh)</th>
      <th>Natural Gas (pence per kWh)</th>
      <th>Average Unit Cost (Pence per kWh)</th>
      <th>Region</th>
      <th>Median_Income</th>
      <th>Salary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1998</td>
      <td>0.544375</td>
      <td>0.774538</td>
      <td>0.848242</td>
      <td>8.258000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1999</td>
      <td>0.513870</td>
      <td>0.911555</td>
      <td>0.782009</td>
      <td>8.143000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2000</td>
      <td>0.512963</td>
      <td>1.273934</td>
      <td>0.750103</td>
      <td>7.901333</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2001</td>
      <td>0.551299</td>
      <td>1.220321</td>
      <td>0.824940</td>
      <td>7.737667</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2002</td>
      <td>0.498163</td>
      <td>1.289745</td>
      <td>0.740489</td>
      <td>7.725000</td>
      <td>North East</td>
      <td>343.2</td>
      <td>17846.4</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>197</th>
      <td>2023</td>
      <td>1.775227</td>
      <td>4.273539</td>
      <td>4.904535</td>
      <td>36.695593</td>
      <td>West Midlands</td>
      <td>657.9</td>
      <td>34210.8</td>
    </tr>
    <tr>
      <th>198</th>
      <td>2023</td>
      <td>1.775227</td>
      <td>4.273539</td>
      <td>4.904535</td>
      <td>36.695593</td>
      <td>East</td>
      <td>709.5</td>
      <td>36894.0</td>
    </tr>
    <tr>
      <th>199</th>
      <td>2023</td>
      <td>1.775227</td>
      <td>4.273539</td>
      <td>4.904535</td>
      <td>36.695593</td>
      <td>London</td>
      <td>804.9</td>
      <td>41854.8</td>
    </tr>
    <tr>
      <th>200</th>
      <td>2023</td>
      <td>1.775227</td>
      <td>4.273539</td>
      <td>4.904535</td>
      <td>36.695593</td>
      <td>South East</td>
      <td>728.3</td>
      <td>37871.6</td>
    </tr>
    <tr>
      <th>201</th>
      <td>2023</td>
      <td>1.775227</td>
      <td>4.273539</td>
      <td>4.904535</td>
      <td>36.695593</td>
      <td>South West</td>
      <td>667.5</td>
      <td>34710.0</td>
    </tr>
  </tbody>
</table>
<p>202 rows × 8 columns</p>
</div>



EDITS MADE HERE


```python
#retrieving the yearly minimum of 'Domestic_Gas_Consumption_to_Median_Salary(GWh/person/£)'
Gas_Consumption_Cleaned_PopD_MedIncome_Min_Values = Gas_Consumption_Cleaned_PopD_MedIncome.groupby('Year',as_index=False).agg({
    'Domestic_Gas_Consumption_to_Median_Salary(GWh/person/£)' : 'min'
})
Gas_Consumption_Cleaned_PopD_MedIncome_Min_Values.rename(columns= {'Domestic_Gas_Consumption_to_Median_Salary(GWh/person/£)': 'Domestic_Gas_Consumption_to_Median_Salary_Annual_min(GWh/person/£)'},inplace=True)
Gas_Consumption_Cleaned_PopD_MedIncome_Min_Values
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
      <th>Year</th>
      <th>Domestic_Gas_Consumption_to_Median_Salary_Annual_min(GWh/person/£)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005</td>
      <td>2.202689e-07</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2006</td>
      <td>2.055350e-07</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2007</td>
      <td>1.951975e-07</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2008</td>
      <td>1.787076e-07</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2009</td>
      <td>1.599383e-07</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2010</td>
      <td>1.549462e-07</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2011</td>
      <td>1.432596e-07</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2012</td>
      <td>1.416845e-07</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2013</td>
      <td>1.374620e-07</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2014</td>
      <td>1.330323e-07</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2015</td>
      <td>1.294064e-07</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2016</td>
      <td>1.273451e-07</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2017</td>
      <td>1.264038e-07</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2018</td>
      <td>1.225793e-07</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2019</td>
      <td>1.190629e-07</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2020</td>
      <td>1.222902e-07</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2021</td>
      <td>1.108826e-07</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2022</td>
      <td>9.301583e-08</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Joining the Yearly minimum onto the main table and expressing Domestic_Gas_Consumption_to_Median_Salary(GWh/person/£) as a percentage of the annual minimum
Gas_Consumption_Cleaned_PopD_MedIncome_Percent = Gas_Consumption_Cleaned_PopD_MedIncome.merge(Gas_Consumption_Cleaned_PopD_MedIncome_Min_Values, on=['Year'], how='left')
Gas_Consumption_Cleaned_PopD_MedIncome_Percent['Domestic_Gas_Consumption_to_Median_Salary_as_a_Percentage_of_Annual_min'] = Gas_Consumption_Cleaned_PopD_MedIncome_Percent['Domestic_Gas_Consumption_to_Median_Salary(GWh/person/£)']/Gas_Consumption_Cleaned_PopD_MedIncome_Percent['Domestic_Gas_Consumption_to_Median_Salary_Annual_min(GWh/person/£)']
Gas_Consumption_Cleaned_PopD_MedIncome_Percent
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
      <th>Year</th>
      <th>Location_Code</th>
      <th>Region</th>
      <th>Number of meters\n(thousands):\nDomestic\n</th>
      <th>Number of meters\n(thousands):\nNon-Domestic</th>
      <th>Number of meters\n(thousands):\nAll meters</th>
      <th>Total consumption\n(GWh):\nDomestic\n</th>
      <th>Total consumption\n(GWh):\nNon-Domestic</th>
      <th>Total consumption\n(GWh):\nAll meters</th>
      <th>Mean consumption\n(kWh per meter):\nDomestic\n</th>
      <th>...</th>
      <th>Area km^2</th>
      <th>Population_Density(Persons.km^-2)</th>
      <th>Domestic_Gas_Consumption(GWh/person)</th>
      <th>Commercial_Gas_Consumption(GWh/person)</th>
      <th>Total_Gas_Consumption(GWh/person)</th>
      <th>Median_Income</th>
      <th>Salary</th>
      <th>Domestic_Gas_Consumption_to_Median_Salary(GWh/person/£)</th>
      <th>Domestic_Gas_Consumption_to_Median_Salary_Annual_min(GWh/person/£)</th>
      <th>Domestic_Gas_Consumption_to_Median_Salary_as_a_Percentage_of_Annual_min</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005</td>
      <td>UKC</td>
      <td>North East</td>
      <td>1037.403</td>
      <td>16.219</td>
      <td>1053.622</td>
      <td>20710.658157</td>
      <td>13952.158200</td>
      <td>34662.816357</td>
      <td>19963.946660</td>
      <td>...</td>
      <td>8581.0</td>
      <td>298.821453</td>
      <td>0.008077</td>
      <td>0.005441</td>
      <td>0.013518</td>
      <td>383.3</td>
      <td>19931.6</td>
      <td>4.052304e-07</td>
      <td>2.202689e-07</td>
      <td>1.839708</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2005</td>
      <td>UKD</td>
      <td>North West</td>
      <td>2747.967</td>
      <td>48.871</td>
      <td>2796.838</td>
      <td>53390.873953</td>
      <td>35926.287793</td>
      <td>89317.161746</td>
      <td>19429.226753</td>
      <td>...</td>
      <td>14108.0</td>
      <td>446.576040</td>
      <td>0.008474</td>
      <td>0.005702</td>
      <td>0.014177</td>
      <td>409.5</td>
      <td>21294.0</td>
      <td>3.979687e-07</td>
      <td>2.202689e-07</td>
      <td>1.806740</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005</td>
      <td>UKE</td>
      <td>Yorkshire and The Humber</td>
      <td>1989.970</td>
      <td>36.997</td>
      <td>2026.967</td>
      <td>39024.143235</td>
      <td>30648.965991</td>
      <td>69673.109226</td>
      <td>19610.417863</td>
      <td>...</td>
      <td>15404.0</td>
      <td>296.114433</td>
      <td>0.008555</td>
      <td>0.006719</td>
      <td>0.015275</td>
      <td>400.0</td>
      <td>20800.0</td>
      <td>4.113173e-07</td>
      <td>2.202689e-07</td>
      <td>1.867342</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005</td>
      <td>UKF</td>
      <td>East Midlands</td>
      <td>1620.786</td>
      <td>28.502</td>
      <td>1649.288</td>
      <td>31469.021304</td>
      <td>18936.399391</td>
      <td>50405.420695</td>
      <td>19415.901485</td>
      <td>...</td>
      <td>15624.0</td>
      <td>234.491086</td>
      <td>0.008589</td>
      <td>0.005169</td>
      <td>0.013758</td>
      <td>412.2</td>
      <td>21434.4</td>
      <td>4.007314e-07</td>
      <td>2.202689e-07</td>
      <td>1.819283</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2005</td>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>1984.918</td>
      <td>36.397</td>
      <td>2021.315</td>
      <td>37726.189511</td>
      <td>22962.792215</td>
      <td>60688.981726</td>
      <td>19006.422185</td>
      <td>...</td>
      <td>12998.0</td>
      <td>412.071665</td>
      <td>0.007044</td>
      <td>0.004287</td>
      <td>0.011331</td>
      <td>404.7</td>
      <td>21044.4</td>
      <td>3.347011e-07</td>
      <td>2.202689e-07</td>
      <td>1.519511</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>2022</td>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>2221.428</td>
      <td>21.914</td>
      <td>2243.342</td>
      <td>25247.105176</td>
      <td>15406.577700</td>
      <td>40653.682876</td>
      <td>11365.259273</td>
      <td>...</td>
      <td>12998.0</td>
      <td>462.919372</td>
      <td>0.004196</td>
      <td>0.002560</td>
      <td>0.006756</td>
      <td>615.0</td>
      <td>31980.0</td>
      <td>1.312053e-07</td>
      <td>9.301583e-08</td>
      <td>1.410569</td>
    </tr>
    <tr>
      <th>158</th>
      <td>2022</td>
      <td>UKH</td>
      <td>East</td>
      <td>2217.992</td>
      <td>20.366</td>
      <td>2238.358</td>
      <td>25143.154335</td>
      <td>13711.245874</td>
      <td>38854.400209</td>
      <td>11335.998658</td>
      <td>...</td>
      <td>19116.0</td>
      <td>334.872254</td>
      <td>0.003928</td>
      <td>0.002142</td>
      <td>0.006070</td>
      <td>670.0</td>
      <td>34840.0</td>
      <td>1.127367e-07</td>
      <td>9.301583e-08</td>
      <td>1.212017</td>
    </tr>
    <tr>
      <th>159</th>
      <td>2022</td>
      <td>UKI</td>
      <td>London</td>
      <td>3001.866</td>
      <td>39.524</td>
      <td>3041.390</td>
      <td>35792.967053</td>
      <td>19576.692142</td>
      <td>55369.659195</td>
      <td>11923.572555</td>
      <td>...</td>
      <td>1572.0</td>
      <td>5641.884860</td>
      <td>0.004036</td>
      <td>0.002207</td>
      <td>0.006243</td>
      <td>766.6</td>
      <td>39863.2</td>
      <td>1.012392e-07</td>
      <td>9.301583e-08</td>
      <td>1.088408</td>
    </tr>
    <tr>
      <th>160</th>
      <td>2022</td>
      <td>UKJ</td>
      <td>South East</td>
      <td>3428.042</td>
      <td>35.662</td>
      <td>3463.704</td>
      <td>39604.299045</td>
      <td>16541.969255</td>
      <td>56146.268301</td>
      <td>11553.037870</td>
      <td>...</td>
      <td>1972.0</td>
      <td>6026.372718</td>
      <td>0.003333</td>
      <td>0.001392</td>
      <td>0.004725</td>
      <td>689.0</td>
      <td>35828.0</td>
      <td>9.301583e-08</td>
      <td>9.301583e-08</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>161</th>
      <td>2022</td>
      <td>UKK</td>
      <td>South West</td>
      <td>1994.866</td>
      <td>18.140</td>
      <td>2013.006</td>
      <td>19620.178140</td>
      <td>10846.932730</td>
      <td>30467.110870</td>
      <td>9835.336379</td>
      <td>...</td>
      <td>23836.0</td>
      <td>217.731415</td>
      <td>0.003780</td>
      <td>0.002090</td>
      <td>0.005871</td>
      <td>622.0</td>
      <td>32344.0</td>
      <td>1.168839e-07</td>
      <td>9.301583e-08</td>
      <td>1.256602</td>
    </tr>
  </tbody>
</table>
<p>162 rows × 24 columns</p>
</div>




```python
#Choropleth standardised to see if north and south are getting more or less equal
#Choropleth hidden here to support GitHub upload
def create_salaryration_choropleth(year):
    fig = px.choropleth(Gas_Consumption_Cleaned_PopD_MedIncome.loc[Gas_Consumption_Cleaned_PopD_MedIncome['Year'] == year]
                        ,locations='Location_Code'
                        ,geojson=Regions
                        ,color ='Domestic_Gas_Consumption_to_Median_Salary(GWh/person/£)'
                        ,featureidkey="properties.NUTS112CD"
                        ,hover_name='Region'
                        ,color_continuous_scale=px.colors.sequential.Viridis
                        )
    fig.update_geos(
        fitbounds="geojson",
       # projection_scale = 100,
        visible=False,
        projection_type="orthographic" 
        )
    fig.update_layout(
    margin={"r": 0, "t": 50, "l": 0, "b": 0}, 
    title={
        'text': f"{year}",  
        'y': 0.95, 
        'x': 0.5,  
        'xanchor': 'center',
        'yanchor': 'top'},
    coloraxis_colorbar_title=None,
    coloraxis_colorbar=dict(
            len=0.8,
            thickness=15 
        ),
        width=800,
        height=650,
        font=dict(
        size=26 
    ))

    return fig

Years_to_Graph = list(range(2005,2023))
Years_to_Graph

figures = [create_salaryration_choropleth(year) for year in Years_to_Graph]

'''for fig in figures:
    fig.show()
    '''
    #Showing "Domestic Gas Consumption as a Portion of Salary {year} (GWh/person/£)"
```




    'for fig in figures:\n    fig.show()\n    '




```python
#defining the max and min values for all years
Max_Percent = Gas_Consumption_Cleaned_PopD_MedIncome_Percent['Domestic_Gas_Consumption_to_Median_Salary_as_a_Percentage_of_Annual_min'].max()
Min_Percent = Gas_Consumption_Cleaned_PopD_MedIncome_Percent['Domestic_Gas_Consumption_to_Median_Salary_as_a_Percentage_of_Annual_min'].min()
```


```python
#Choropleth standardised to see if north and south are getting more or less equal
#Choropleth hidden here to support GitHub upload
def create_salaryration_choropleth(year):
    fig = px.choropleth(Gas_Consumption_Cleaned_PopD_MedIncome_Percent.loc[Gas_Consumption_Cleaned_PopD_MedIncome_Percent['Year'] == year]
                        ,locations='Location_Code'
                        ,geojson=Regions
                        ,color ='Domestic_Gas_Consumption_to_Median_Salary_as_a_Percentage_of_Annual_min'
                        ,featureidkey="properties.NUTS112CD"
                        ,hover_name='Region'
                        ,color_continuous_scale=px.colors.sequential.Viridis
                        ,range_color = (Min_Percent, Max_Percent)
                        )
    fig.update_geos(
        fitbounds="geojson",
       # projection_scale = 100,
        visible=False,
        projection_type="orthographic" 
        )
    fig.update_layout(
    margin={"r": 0, "t": 50, "l": 0, "b": 0}, 
    title={
        'text': f"{year}",  
        'y': 0.95, 
        'x': 0.5,  
        'xanchor': 'center',
        'yanchor': 'top'},
    coloraxis_colorbar_title=None,
    coloraxis_colorbar=dict(
            len=0.8,
            thickness=15 
        ),
        width=800,
        height=650,
        font=dict(
        size=26 
    ))

    return fig

Years_to_Graph = list(range(2005,2023))
Years_to_Graph

figures = [create_salaryration_choropleth(year) for year in Years_to_Graph]

'''for fig in figures:
    fig.show()'''
    
    #Showing "Domestic Gas Consumption as a Portion of Salary {year} % difference From Annual Minimum (%)"
```




    'for fig in figures:\n    fig.show()'




```python
Gas_Consumption_Cleaned_PopD_MedIncome
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
      <th>Year</th>
      <th>Location_Code</th>
      <th>Region</th>
      <th>Number of meters\n(thousands):\nDomestic\n</th>
      <th>Number of meters\n(thousands):\nNon-Domestic</th>
      <th>Number of meters\n(thousands):\nAll meters</th>
      <th>Total consumption\n(GWh):\nDomestic\n</th>
      <th>Total consumption\n(GWh):\nNon-Domestic</th>
      <th>Total consumption\n(GWh):\nAll meters</th>
      <th>Mean consumption\n(kWh per meter):\nDomestic\n</th>
      <th>...</th>
      <th>Population</th>
      <th>Data_Type</th>
      <th>Area km^2</th>
      <th>Population_Density(Persons.km^-2)</th>
      <th>Domestic_Gas_Consumption(GWh/person)</th>
      <th>Commercial_Gas_Consumption(GWh/person)</th>
      <th>Total_Gas_Consumption(GWh/person)</th>
      <th>Median_Income</th>
      <th>Salary</th>
      <th>Domestic_Gas_Consumption_to_Median_Salary(GWh/person/£)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005</td>
      <td>UKC</td>
      <td>North East</td>
      <td>1037.403</td>
      <td>16.219</td>
      <td>1053.622</td>
      <td>20710.658157</td>
      <td>13952.158200</td>
      <td>34662.816357</td>
      <td>19963.946660</td>
      <td>...</td>
      <td>2.564187e+06</td>
      <td>Extrapolated</td>
      <td>8581.0</td>
      <td>298.821453</td>
      <td>0.008077</td>
      <td>0.005441</td>
      <td>0.013518</td>
      <td>383.3</td>
      <td>19931.6</td>
      <td>4.052304e-07</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2005</td>
      <td>UKD</td>
      <td>North West</td>
      <td>2747.967</td>
      <td>48.871</td>
      <td>2796.838</td>
      <td>53390.873953</td>
      <td>35926.287793</td>
      <td>89317.161746</td>
      <td>19429.226753</td>
      <td>...</td>
      <td>6.300295e+06</td>
      <td>Extrapolated</td>
      <td>14108.0</td>
      <td>446.576040</td>
      <td>0.008474</td>
      <td>0.005702</td>
      <td>0.014177</td>
      <td>409.5</td>
      <td>21294.0</td>
      <td>3.979687e-07</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005</td>
      <td>UKE</td>
      <td>Yorkshire and The Humber</td>
      <td>1989.970</td>
      <td>36.997</td>
      <td>2026.967</td>
      <td>39024.143235</td>
      <td>30648.965991</td>
      <td>69673.109226</td>
      <td>19610.417863</td>
      <td>...</td>
      <td>4.561347e+06</td>
      <td>Extrapolated</td>
      <td>15404.0</td>
      <td>296.114433</td>
      <td>0.008555</td>
      <td>0.006719</td>
      <td>0.015275</td>
      <td>400.0</td>
      <td>20800.0</td>
      <td>4.113173e-07</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005</td>
      <td>UKF</td>
      <td>East Midlands</td>
      <td>1620.786</td>
      <td>28.502</td>
      <td>1649.288</td>
      <td>31469.021304</td>
      <td>18936.399391</td>
      <td>50405.420695</td>
      <td>19415.901485</td>
      <td>...</td>
      <td>3.663689e+06</td>
      <td>Extrapolated</td>
      <td>15624.0</td>
      <td>234.491086</td>
      <td>0.008589</td>
      <td>0.005169</td>
      <td>0.013758</td>
      <td>412.2</td>
      <td>21434.4</td>
      <td>4.007314e-07</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2005</td>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>1984.918</td>
      <td>36.397</td>
      <td>2021.315</td>
      <td>37726.189511</td>
      <td>22962.792215</td>
      <td>60688.981726</td>
      <td>19006.422185</td>
      <td>...</td>
      <td>5.356108e+06</td>
      <td>Extrapolated</td>
      <td>12998.0</td>
      <td>412.071665</td>
      <td>0.007044</td>
      <td>0.004287</td>
      <td>0.011331</td>
      <td>404.7</td>
      <td>21044.4</td>
      <td>3.347011e-07</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>2022</td>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>2221.428</td>
      <td>21.914</td>
      <td>2243.342</td>
      <td>25247.105176</td>
      <td>15406.577700</td>
      <td>40653.682876</td>
      <td>11365.259273</td>
      <td>...</td>
      <td>6.017026e+06</td>
      <td>Original</td>
      <td>12998.0</td>
      <td>462.919372</td>
      <td>0.004196</td>
      <td>0.002560</td>
      <td>0.006756</td>
      <td>615.0</td>
      <td>31980.0</td>
      <td>1.312053e-07</td>
    </tr>
    <tr>
      <th>158</th>
      <td>2022</td>
      <td>UKH</td>
      <td>East</td>
      <td>2217.992</td>
      <td>20.366</td>
      <td>2238.358</td>
      <td>25143.154335</td>
      <td>13711.245874</td>
      <td>38854.400209</td>
      <td>11335.998658</td>
      <td>...</td>
      <td>6.401418e+06</td>
      <td>Original</td>
      <td>19116.0</td>
      <td>334.872254</td>
      <td>0.003928</td>
      <td>0.002142</td>
      <td>0.006070</td>
      <td>670.0</td>
      <td>34840.0</td>
      <td>1.127367e-07</td>
    </tr>
    <tr>
      <th>159</th>
      <td>2022</td>
      <td>UKI</td>
      <td>London</td>
      <td>3001.866</td>
      <td>39.524</td>
      <td>3041.390</td>
      <td>35792.967053</td>
      <td>19576.692142</td>
      <td>55369.659195</td>
      <td>11923.572555</td>
      <td>...</td>
      <td>8.869043e+06</td>
      <td>Original</td>
      <td>1572.0</td>
      <td>5641.884860</td>
      <td>0.004036</td>
      <td>0.002207</td>
      <td>0.006243</td>
      <td>766.6</td>
      <td>39863.2</td>
      <td>1.012392e-07</td>
    </tr>
    <tr>
      <th>160</th>
      <td>2022</td>
      <td>UKJ</td>
      <td>South East</td>
      <td>3428.042</td>
      <td>35.662</td>
      <td>3463.704</td>
      <td>39604.299045</td>
      <td>16541.969255</td>
      <td>56146.268301</td>
      <td>11553.037870</td>
      <td>...</td>
      <td>1.188401e+07</td>
      <td>Original</td>
      <td>1972.0</td>
      <td>6026.372718</td>
      <td>0.003333</td>
      <td>0.001392</td>
      <td>0.004725</td>
      <td>689.0</td>
      <td>35828.0</td>
      <td>9.301583e-08</td>
    </tr>
    <tr>
      <th>161</th>
      <td>2022</td>
      <td>UKK</td>
      <td>South West</td>
      <td>1994.866</td>
      <td>18.140</td>
      <td>2013.006</td>
      <td>19620.178140</td>
      <td>10846.932730</td>
      <td>30467.110870</td>
      <td>9835.336379</td>
      <td>...</td>
      <td>5.189846e+06</td>
      <td>Original</td>
      <td>23836.0</td>
      <td>217.731415</td>
      <td>0.003780</td>
      <td>0.002090</td>
      <td>0.005871</td>
      <td>622.0</td>
      <td>32344.0</td>
      <td>1.168839e-07</td>
    </tr>
  </tbody>
</table>
<p>162 rows × 22 columns</p>
</div>




```python
#Join to allow generation of choropleths displaying Energy cost againt salary by region and year
Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice = pd.merge(Gas_Consumption_Cleaned_PopD_MedIncome, Energy_Price_Region, on=['Region','Year'], how='left')
Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice["Salary (£'000)"] = Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice["Salary"]/1000
Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice["EngPrice / Salary (Pence/kWh/£'000)"] = Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice['Average Unit Cost (Pence per kWh)']/Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice["Salary (£'000)"]
```


```python
Max_Unit_Cost = Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice["Average Unit Cost (Pence per kWh)"].max()
Min_Unit_Cost = Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice["Average Unit Cost (Pence per kWh)"].min()
```


```python
#Choropleth showing avg energy price agianst alary by region and year (This one for comparing change over time)
#Choropleth hidden here to support GitHub upload
def create_salary_to_engprice_choropleth_time(year):
    fig = px.choropleth(Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice.loc[Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice['Year'] == year]
                        ,locations='Location_Code'
                        ,geojson=Regions
                        ,color ="Average Unit Cost (Pence per kWh)"
                        ,featureidkey="properties.NUTS112CD"
                        ,hover_name='Region'
                        ,range_color = (Min_Unit_Cost, Max_Unit_Cost)
                        ,color_continuous_scale = px.colors.diverging.RdYlBu[::-1]
                        )
    fig.update_geos(
        fitbounds="geojson",
       # projection_scale = 100,
        visible=False,
        projection_type="orthographic" 
        )
    fig.update_layout(
    margin={"r": 0, "t": 50, "l": 0, "b": 0}, 
    title={
        'text': f"{year}",  
        'y': 0.95, 
        'x': 0.5,  
        'xanchor': 'center',
        'yanchor': 'top'},
    coloraxis_colorbar_title=None,
    coloraxis_colorbar=dict(
            len=0.8,
            thickness=15 
        ),
        width=800,
        height=650,
        font=dict(
        size=26 
    ))

    return fig

Years_to_Graph = list(range(2005,2023))
Years_to_Graph

figures = [create_salary_to_engprice_choropleth_time(year) for year in Years_to_Graph]

'''for fig in figures:
    fig.show()'''
    
    #Showing "Average Energy Price {year} (Pence/kWh)"
    
    ##Do not include
```




    'for fig in figures:\n    fig.show()'




```python
Max_Price_Sal_Ratio = Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice["EngPrice / Salary (Pence/kWh/£'000)"].max()
Min_Price_Sal_Ratio = Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice["EngPrice / Salary (Pence/kWh/£'000)"].min()
```


```python
Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice[Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice['Year']==2022]
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
      <th>Year</th>
      <th>Location_Code</th>
      <th>Region</th>
      <th>Number of meters\n(thousands):\nDomestic\n</th>
      <th>Number of meters\n(thousands):\nNon-Domestic</th>
      <th>Number of meters\n(thousands):\nAll meters</th>
      <th>Total consumption\n(GWh):\nDomestic\n</th>
      <th>Total consumption\n(GWh):\nNon-Domestic</th>
      <th>Total consumption\n(GWh):\nAll meters</th>
      <th>Mean consumption\n(kWh per meter):\nDomestic\n</th>
      <th>...</th>
      <th>Domestic_Gas_Consumption(GWh/person)</th>
      <th>Commercial_Gas_Consumption(GWh/person)</th>
      <th>Total_Gas_Consumption(GWh/person)</th>
      <th>Median_Income</th>
      <th>Salary</th>
      <th>Domestic_Gas_Consumption_to_Median_Salary(GWh/person/£)</th>
      <th>In_Region</th>
      <th>Average Unit Cost (Pence per kWh)</th>
      <th>Salary (£'000)</th>
      <th>EngPrice / Salary (Pence/kWh/£'000)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>154</th>
      <td>2022</td>
      <td>UKC</td>
      <td>North East</td>
      <td>1158.606</td>
      <td>10.530</td>
      <td>1169.136</td>
      <td>12999.923010</td>
      <td>8117.429798</td>
      <td>21117.352808</td>
      <td>11220.313903</td>
      <td>...</td>
      <td>0.004847</td>
      <td>0.003027</td>
      <td>0.007874</td>
      <td>581.4</td>
      <td>30232.8</td>
      <td>1.603218e-07</td>
      <td>North East</td>
      <td>33.430249</td>
      <td>30.2328</td>
      <td>1.105761</td>
    </tr>
    <tr>
      <th>155</th>
      <td>2022</td>
      <td>UKD</td>
      <td>North West</td>
      <td>3016.545</td>
      <td>28.398</td>
      <td>3044.943</td>
      <td>33039.479334</td>
      <td>24060.254689</td>
      <td>57099.734023</td>
      <td>10952.755332</td>
      <td>...</td>
      <td>0.004711</td>
      <td>0.003431</td>
      <td>0.008142</td>
      <td>604.4</td>
      <td>31428.8</td>
      <td>1.499056e-07</td>
      <td>North West</td>
      <td>34.211569</td>
      <td>31.4288</td>
      <td>1.088542</td>
    </tr>
    <tr>
      <th>156</th>
      <td>2022</td>
      <td>UKE</td>
      <td>Yorkshire and The Humber</td>
      <td>2212.441</td>
      <td>21.635</td>
      <td>2234.076</td>
      <td>25289.699190</td>
      <td>18934.778220</td>
      <td>44224.477410</td>
      <td>11430.677333</td>
      <td>...</td>
      <td>0.005146</td>
      <td>0.003853</td>
      <td>0.008999</td>
      <td>594.5</td>
      <td>30914.0</td>
      <td>1.664659e-07</td>
      <td>Yorkshire</td>
      <td>34.061956</td>
      <td>30.9140</td>
      <td>1.101829</td>
    </tr>
    <tr>
      <th>157</th>
      <td>2022</td>
      <td>UKF</td>
      <td>East Midlands</td>
      <td>1901.834</td>
      <td>17.598</td>
      <td>1919.432</td>
      <td>21536.954679</td>
      <td>15105.203633</td>
      <td>36642.158311</td>
      <td>11324.308367</td>
      <td>...</td>
      <td>0.005200</td>
      <td>0.003647</td>
      <td>0.008846</td>
      <td>604.3</td>
      <td>31423.6</td>
      <td>1.654665e-07</td>
      <td>East Midlands</td>
      <td>33.826934</td>
      <td>31.4236</td>
      <td>1.076482</td>
    </tr>
    <tr>
      <th>158</th>
      <td>2022</td>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>2221.428</td>
      <td>21.914</td>
      <td>2243.342</td>
      <td>25247.105176</td>
      <td>15406.577700</td>
      <td>40653.682876</td>
      <td>11365.259273</td>
      <td>...</td>
      <td>0.004196</td>
      <td>0.002560</td>
      <td>0.006756</td>
      <td>615.0</td>
      <td>31980.0</td>
      <td>1.312053e-07</td>
      <td>West Midlands</td>
      <td>34.459508</td>
      <td>31.9800</td>
      <td>1.077533</td>
    </tr>
    <tr>
      <th>159</th>
      <td>2022</td>
      <td>UKH</td>
      <td>East</td>
      <td>2217.992</td>
      <td>20.366</td>
      <td>2238.358</td>
      <td>25143.154335</td>
      <td>13711.245874</td>
      <td>38854.400209</td>
      <td>11335.998658</td>
      <td>...</td>
      <td>0.003928</td>
      <td>0.002142</td>
      <td>0.006070</td>
      <td>670.0</td>
      <td>34840.0</td>
      <td>1.127367e-07</td>
      <td>Eastern</td>
      <td>34.766436</td>
      <td>34.8400</td>
      <td>0.997889</td>
    </tr>
    <tr>
      <th>160</th>
      <td>2022</td>
      <td>UKI</td>
      <td>London</td>
      <td>3001.866</td>
      <td>39.524</td>
      <td>3041.390</td>
      <td>35792.967053</td>
      <td>19576.692142</td>
      <td>55369.659195</td>
      <td>11923.572555</td>
      <td>...</td>
      <td>0.004036</td>
      <td>0.002207</td>
      <td>0.006243</td>
      <td>766.6</td>
      <td>39863.2</td>
      <td>1.012392e-07</td>
      <td>London</td>
      <td>34.592071</td>
      <td>39.8632</td>
      <td>0.867770</td>
    </tr>
    <tr>
      <th>161</th>
      <td>2022</td>
      <td>UKJ</td>
      <td>South East</td>
      <td>3428.042</td>
      <td>35.662</td>
      <td>3463.704</td>
      <td>39604.299045</td>
      <td>16541.969255</td>
      <td>56146.268301</td>
      <td>11553.037870</td>
      <td>...</td>
      <td>0.003333</td>
      <td>0.001392</td>
      <td>0.004725</td>
      <td>689.0</td>
      <td>35828.0</td>
      <td>9.301583e-08</td>
      <td>South East</td>
      <td>34.309975</td>
      <td>35.8280</td>
      <td>0.957630</td>
    </tr>
    <tr>
      <th>162</th>
      <td>2022</td>
      <td>UKK</td>
      <td>South West</td>
      <td>1994.866</td>
      <td>18.140</td>
      <td>2013.006</td>
      <td>19620.178140</td>
      <td>10846.932730</td>
      <td>30467.110870</td>
      <td>9835.336379</td>
      <td>...</td>
      <td>0.003780</td>
      <td>0.002090</td>
      <td>0.005871</td>
      <td>622.0</td>
      <td>32344.0</td>
      <td>1.168839e-07</td>
      <td>South West</td>
      <td>34.269061</td>
      <td>32.3440</td>
      <td>1.059518</td>
    </tr>
  </tbody>
</table>
<p>9 rows × 26 columns</p>
</div>




```python
#retrieving the yearly minimum of "EngPrice / Salary (Pence/kWh/£'000)"
Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice_Min_Values = Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice.groupby('Year',as_index=False).agg({
    "EngPrice / Salary (Pence/kWh/£'000)" : 'min'
})
Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice_Min_Values.rename(columns= {"EngPrice / Salary (Pence/kWh/£'000)": "EngPrice / Salary of Annual Minimum(Pence/kWh/£'000)"},inplace=True)
Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice_Min_Values
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
      <th>Year</th>
      <th>EngPrice / Salary of Annual Minimum(Pence/kWh/£'000)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005</td>
      <td>0.318649</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2006</td>
      <td>0.366585</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2007</td>
      <td>0.382096</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2008</td>
      <td>0.422873</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2009</td>
      <td>0.430430</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2010</td>
      <td>0.417600</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2011</td>
      <td>0.443498</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2012</td>
      <td>0.464709</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2013</td>
      <td>0.500817</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2014</td>
      <td>0.498517</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2015</td>
      <td>0.494880</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2016</td>
      <td>0.495815</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2017</td>
      <td>0.526229</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2018</td>
      <td>0.558080</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2019</td>
      <td>0.568789</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2020</td>
      <td>0.562296</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2021</td>
      <td>0.602011</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2022</td>
      <td>0.867770</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Joining the Yearly minimum onto the main table and expressing "EngPrice / Salary (Pence/kWh/£'000)" as a percentage of the annual minimum
Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice_Percent = Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice.merge(Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice_Min_Values, on=['Year'], how='left')
Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice_Percent["EngPrice / Salary as a Percent of Annual Minimum(%)"] = Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice_Percent["EngPrice / Salary (Pence/kWh/£'000)"]/Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice_Percent["EngPrice / Salary of Annual Minimum(Pence/kWh/£'000)"]
Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice_Percent
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
      <th>Year</th>
      <th>Location_Code</th>
      <th>Region</th>
      <th>Number of meters\n(thousands):\nDomestic\n</th>
      <th>Number of meters\n(thousands):\nNon-Domestic</th>
      <th>Number of meters\n(thousands):\nAll meters</th>
      <th>Total consumption\n(GWh):\nDomestic\n</th>
      <th>Total consumption\n(GWh):\nNon-Domestic</th>
      <th>Total consumption\n(GWh):\nAll meters</th>
      <th>Mean consumption\n(kWh per meter):\nDomestic\n</th>
      <th>...</th>
      <th>Total_Gas_Consumption(GWh/person)</th>
      <th>Median_Income</th>
      <th>Salary</th>
      <th>Domestic_Gas_Consumption_to_Median_Salary(GWh/person/£)</th>
      <th>In_Region</th>
      <th>Average Unit Cost (Pence per kWh)</th>
      <th>Salary (£'000)</th>
      <th>EngPrice / Salary (Pence/kWh/£'000)</th>
      <th>EngPrice / Salary of Annual Minimum(Pence/kWh/£'000)</th>
      <th>EngPrice / Salary as a Percent of Annual Minimum(%)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2005</td>
      <td>UKC</td>
      <td>North East</td>
      <td>1037.403</td>
      <td>16.219</td>
      <td>1053.622</td>
      <td>20710.658157</td>
      <td>13952.158200</td>
      <td>34662.816357</td>
      <td>19963.946660</td>
      <td>...</td>
      <td>0.013518</td>
      <td>383.3</td>
      <td>19931.6</td>
      <td>4.052304e-07</td>
      <td>North East</td>
      <td>8.727273</td>
      <td>19.9316</td>
      <td>0.437861</td>
      <td>0.318649</td>
      <td>1.374119</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2005</td>
      <td>UKD</td>
      <td>North West</td>
      <td>2747.967</td>
      <td>48.871</td>
      <td>2796.838</td>
      <td>53390.873953</td>
      <td>35926.287793</td>
      <td>89317.161746</td>
      <td>19429.226753</td>
      <td>...</td>
      <td>0.014177</td>
      <td>409.5</td>
      <td>21294.0</td>
      <td>3.979687e-07</td>
      <td>North West</td>
      <td>8.181818</td>
      <td>21.2940</td>
      <td>0.384231</td>
      <td>0.318649</td>
      <td>1.205815</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2005</td>
      <td>UKE</td>
      <td>Yorkshire and The Humber</td>
      <td>1989.970</td>
      <td>36.997</td>
      <td>2026.967</td>
      <td>39024.143235</td>
      <td>30648.965991</td>
      <td>69673.109226</td>
      <td>19610.417863</td>
      <td>...</td>
      <td>0.015275</td>
      <td>400.0</td>
      <td>20800.0</td>
      <td>4.113173e-07</td>
      <td>Yorkshire</td>
      <td>8.454545</td>
      <td>20.8000</td>
      <td>0.406469</td>
      <td>0.318649</td>
      <td>1.275602</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2005</td>
      <td>UKF</td>
      <td>East Midlands</td>
      <td>1620.786</td>
      <td>28.502</td>
      <td>1649.288</td>
      <td>31469.021304</td>
      <td>18936.399391</td>
      <td>50405.420695</td>
      <td>19415.901485</td>
      <td>...</td>
      <td>0.013758</td>
      <td>412.2</td>
      <td>21434.4</td>
      <td>4.007314e-07</td>
      <td>East Midlands</td>
      <td>8.060606</td>
      <td>21.4344</td>
      <td>0.376059</td>
      <td>0.318649</td>
      <td>1.180170</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2005</td>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>1984.918</td>
      <td>36.397</td>
      <td>2021.315</td>
      <td>37726.189511</td>
      <td>22962.792215</td>
      <td>60688.981726</td>
      <td>19006.422185</td>
      <td>...</td>
      <td>0.011331</td>
      <td>404.7</td>
      <td>21044.4</td>
      <td>3.347011e-07</td>
      <td>West Midlands</td>
      <td>8.454545</td>
      <td>21.0444</td>
      <td>0.401748</td>
      <td>0.318649</td>
      <td>1.260787</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>158</th>
      <td>2022</td>
      <td>UKG</td>
      <td>West Midlands</td>
      <td>2221.428</td>
      <td>21.914</td>
      <td>2243.342</td>
      <td>25247.105176</td>
      <td>15406.577700</td>
      <td>40653.682876</td>
      <td>11365.259273</td>
      <td>...</td>
      <td>0.006756</td>
      <td>615.0</td>
      <td>31980.0</td>
      <td>1.312053e-07</td>
      <td>West Midlands</td>
      <td>34.459508</td>
      <td>31.9800</td>
      <td>1.077533</td>
      <td>0.867770</td>
      <td>1.241727</td>
    </tr>
    <tr>
      <th>159</th>
      <td>2022</td>
      <td>UKH</td>
      <td>East</td>
      <td>2217.992</td>
      <td>20.366</td>
      <td>2238.358</td>
      <td>25143.154335</td>
      <td>13711.245874</td>
      <td>38854.400209</td>
      <td>11335.998658</td>
      <td>...</td>
      <td>0.006070</td>
      <td>670.0</td>
      <td>34840.0</td>
      <td>1.127367e-07</td>
      <td>Eastern</td>
      <td>34.766436</td>
      <td>34.8400</td>
      <td>0.997889</td>
      <td>0.867770</td>
      <td>1.149946</td>
    </tr>
    <tr>
      <th>160</th>
      <td>2022</td>
      <td>UKI</td>
      <td>London</td>
      <td>3001.866</td>
      <td>39.524</td>
      <td>3041.390</td>
      <td>35792.967053</td>
      <td>19576.692142</td>
      <td>55369.659195</td>
      <td>11923.572555</td>
      <td>...</td>
      <td>0.006243</td>
      <td>766.6</td>
      <td>39863.2</td>
      <td>1.012392e-07</td>
      <td>London</td>
      <td>34.592071</td>
      <td>39.8632</td>
      <td>0.867770</td>
      <td>0.867770</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>161</th>
      <td>2022</td>
      <td>UKJ</td>
      <td>South East</td>
      <td>3428.042</td>
      <td>35.662</td>
      <td>3463.704</td>
      <td>39604.299045</td>
      <td>16541.969255</td>
      <td>56146.268301</td>
      <td>11553.037870</td>
      <td>...</td>
      <td>0.004725</td>
      <td>689.0</td>
      <td>35828.0</td>
      <td>9.301583e-08</td>
      <td>South East</td>
      <td>34.309975</td>
      <td>35.8280</td>
      <td>0.957630</td>
      <td>0.867770</td>
      <td>1.103554</td>
    </tr>
    <tr>
      <th>162</th>
      <td>2022</td>
      <td>UKK</td>
      <td>South West</td>
      <td>1994.866</td>
      <td>18.140</td>
      <td>2013.006</td>
      <td>19620.178140</td>
      <td>10846.932730</td>
      <td>30467.110870</td>
      <td>9835.336379</td>
      <td>...</td>
      <td>0.005871</td>
      <td>622.0</td>
      <td>32344.0</td>
      <td>1.168839e-07</td>
      <td>South West</td>
      <td>34.269061</td>
      <td>32.3440</td>
      <td>1.059518</td>
      <td>0.867770</td>
      <td>1.220967</td>
    </tr>
  </tbody>
</table>
<p>163 rows × 28 columns</p>
</div>




```python
Max_EngPrice_Sal_Perc = Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice_Percent["EngPrice / Salary as a Percent of Annual Minimum(%)"].max()
Min_EngPrice_Sal_Perc = Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice_Percent["EngPrice / Salary as a Percent of Annual Minimum(%)"].min()
```


```python

#Choropleth hidden here to support GitHub upload
def create_salary_to_engprice_choropleth_time(year):
    fig = px.choropleth(Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice_Percent.loc[Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice_Percent['Year'] == year]
                        ,locations='Location_Code'
                        ,geojson=Regions
                        ,color ="EngPrice / Salary as a Percent of Annual Minimum(%)"
                        ,featureidkey="properties.NUTS112CD"
                        ,hover_name='Region'
                        ,range_color = (Min_EngPrice_Sal_Perc, Max_EngPrice_Sal_Perc)
                        ,color_continuous_scale = px.colors.diverging.RdYlBu[::-1]
                        )
    fig.update_geos(
        fitbounds="geojson",
       # projection_scale = 100,
        visible=False,
        projection_type="orthographic" 
        )
    fig.update_layout(
    margin={"r": 0, "t": 50, "l": 0, "b": 0}, 
    title={
        'text': f"{year}",  
        'y': 0.95, 
        'x': 0.5,  
        'xanchor': 'center',
        'yanchor': 'top'},
    coloraxis_colorbar_title=None,
    coloraxis_colorbar=dict(
            len=0.8,
            thickness=15 
        ),
        width=800,
        height=650,
        font=dict(
        size=26 
    ))

    return fig

Years_to_Graph = list(range(2005,2023))
Years_to_Graph

figures = [create_salary_to_engprice_choropleth_time(year) for year in Years_to_Graph]

'''for fig in figures:
    fig.show()'''
    
    #showing "Average Energy Price as a Portion of Salary as a Percentage of the annual minimum (%)"
```




    'for fig in figures:\n    fig.show()'




```python
#Choropleth showing avg energy price against salary by region and year (This one for comparing equality of north vs south)
#Choropleth hidden here to support GitHub upload
def create_salary_to_engprice_choropleth(year):
    fig = px.choropleth(Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice.loc[Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice['Year'] == year]
                        ,locations='Location_Code'
                        ,geojson=Regions
                        ,color ="EngPrice / Salary (Pence/kWh/£'000)"
                        ,featureidkey="properties.NUTS112CD"
                        ,hover_name='Region'
                        ,color_continuous_scale=px.colors.diverging.RdYlBu[::-1]
                        )
    fig.update_geos(
        fitbounds="geojson",
       # projection_scale = 100,
        visible=False,
        projection_type="orthographic" 
        )
    fig.update_layout(
    margin={"r": 0, "t": 50, "l": 0, "b": 0}, 
    title={
        'text': f"{year}",  
        'y': 0.95, 
        'x': 0.5,  
        'xanchor': 'center',
        'yanchor': 'top'},
    coloraxis_colorbar_title=None,
    coloraxis_colorbar=dict(
            len=0.8,
            thickness=15 
        ),
        width=800,
        height=650,
        font=dict(
        size=26 
    ))
    
    return fig

Years_to_Graph = list(range(2005,2023))
Years_to_Graph

figures = [create_salary_to_engprice_choropleth(year) for year in Years_to_Graph]

'''for fig in figures:
    fig.show()'''
    
    #Showsing "Average Energy Price as a Portion of Salary {year} (Pence/kWh/£'000)" (Standardised)
```




    'for fig in figures:\n    fig.show()'




```python
#Choropleth showing avg energy price agianst alary by region and year (This one for comparing change over time)
#Choropleth hidden here to support GitHub upload
def create_salary_to_engprice_choropleth_time(year):
    fig = px.choropleth(Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice.loc[Gas_Consumption_Cleaned_PopD_MedIncome_EngPrice['Year'] == year]
                        ,locations='Location_Code'
                        ,geojson=Regions
                        ,color ="EngPrice / Salary (Pence/kWh/£'000)"
                        ,featureidkey="properties.NUTS112CD"
                        ,hover_name='Region'
                        ,range_color = (Min_Price_Sal_Ratio, Max_Price_Sal_Ratio)
                        ,color_continuous_scale = px.colors.diverging.RdYlBu[::-1]
                        )
    fig.update_geos(
        fitbounds="geojson",
       # projection_scale = 100,
        visible=False,
        projection_type="orthographic" 
        )
    fig.update_layout(
    margin={"r": 0, "t": 50, "l": 0, "b": 0}, 
    title={
        'text': f"{year}",  
        'y': 0.95, 
        'x': 0.5,  
        'xanchor': 'center',
        'yanchor': 'top'},
    coloraxis_colorbar_title=None,
    coloraxis_colorbar=dict(
            len=0.8,
            thickness=15 
        ),
        width=800,
        height=650,
        font=dict(
        size=26 
    ))

    return fig

Years_to_Graph = list(range(2005,2023))
Years_to_Graph

figures = [create_salary_to_engprice_choropleth_time(year) for year in Years_to_Graph]

'''for fig in figures:
    fig.show()'''
    
    #showing "Average Energy Price as a Portion of Salary {year} (Pence/kWh/£'000)" (on a fixed scale)
```




    'for fig in figures:\n    fig.show()'


