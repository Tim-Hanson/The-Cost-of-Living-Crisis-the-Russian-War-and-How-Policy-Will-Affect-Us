# DSProject
![histogram](assets/images/histogram.png)
# Heading 1
## Heading 2
### Heading 3
```python
#Defining the file to read from
File2 = --my file location--

#Defining the sheet names to read from the excel
Years1 = [str(n) for n in list(range(2005,2014))]

#Defining the dataframe which will hold the Gas consumption data as the concatination of sheets in the excel with the Year appended
#Treating the years differently as they have different formats in teh ecxel file
Gas_Consumption1 = pd.concat(((pd.read_excel(io=File2, sheet_name=Year, header=5, nrows=9, skiprows=[6,7]).assign(Year = Year))for Year in Years1), ignore_index=True)
Gas_Consumption1['Country or region'].unique()

#In this one Inner and outerlondon are differinciated
Years2 = [str(n) for n in list(range(2014,2015))]
Gas_Consumption2 = pd.concat(((pd.read_excel(io=File2, sheet_name=Year, header=5, nrows=10, skiprows=[6,7]).assign(Year = Year))for Year in Years2), ignore_index=True)

#Removing the sub regions of Inner and outer London as as sum of these 'London is already included' this is to avoid double counting
Years3 = [str(n) for n in list(range(2015,2023))]
Gas_Consumption3 = pd.concat(((pd.read_excel(io=File2, sheet_name=Year, header=5, nrows=11, skiprows=[6,7,8,16,17]).assign(Year = Year))for Year in Years3), ignore_index=True)

Gas_Consumption = pd.concat([Gas_Consumption1, Gas_Consumption2, Gas_Consumption3])

{
 "cells": [
  {
   "cell_type": "code",
   "execution_count": 1,
   "metadata": {},
   "outputs": [],
