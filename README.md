# The Cost-of-Living Crisis, the Russian Invasion and What can be Done to Help – A Data Science Project

![Cover Photo](assets/Pictures/Cover%20Photo.jpg)

(Signify, 2023)

‌
## Introduction

In 2021 the United Kingdom started to see an increase in the cost of day-to-day essentials. This phenomenon has been dubbed ‘the cost-of-living crisis’. One of the most shocking increases in price was seen in the energy market. This disproportionately affected working class families, and leading to an estimated 14.3 million individuals, including 4.3 million children, to be living in poverty in the UK in 2022‌ (Francis-Devine, 2024).

At the start of 2022 Russia invaded Ukraine. Many headlines at the time linked the conflict to rising energy prices in the UK, claiming the severing of our Russian gas imports led to heightened fuel import costs.

![Breaking News](assets/Pictures/Breaking%20news.jpg)

Borrett (2022), Jones (2022), Clark (2022), News (2022).

## Data Science Project Goals

In this project a hypothesis will be addressed, and further analysis will be carried out to give speculative insight into how future policy may affect the cost of living.

### Hypothesis

The hypothesis investigated in the first half of this project is:

“The Russian invasion of Ukraine caused an increase in the cost of energy bills in England”

### Future Policy

The effects of a change of pricing policy from marginal to regional will be investigated. Further explanation on the differences between these two policies will be given in the second half of this project.

## Data Collection and Cleaning

Multiple datasets were leveraged to gain insight into this research question. Timeseries data was combined to observe the relationship between the average unit cost of energy, the import price of fuel and the average salary. Regional data was combined to observe geographic trends in fuel use, average unit cost of energy, and average salary.

Pandas was used to import, clean and manipulate data; SKLearn was used to fit linear regressions and to extrapolate values; Plotly and Seaborn were used to bring the data to life by creating visuals.

To limit scope, only data for England was analysed as part of this project.

Please see the pdf available in assets to view the full data pipeline.

**\[1\] Subnational total final energy consumption, UK, 2005 to 2022** - Department of Energy Security and Net Zero - [_https://view.officeapps.live.com/op/view.aspx?src=https%3A%2F%2Fassets.publishing.service.gov.uk%2Fmedia%2F66f2c25686ba7c46c40c6eae%2FSubnational_total_final_consumption_2005_2022.xlsx&wdOrigin=BROWSELINK_](https://view.officeapps.live.com/op/view.aspx?src=https%3A%2F%2Fassets.publishing.service.gov.uk%2Fmedia%2F66f2c25686ba7c46c40c6eae%2FSubnational_total_final_consumption_2005_2022.xlsx&wdOrigin=BROWSELINK)

**\[2\] Subnational gas consumption, Great Britain, 2005 – 2022** \- Department of Energy Security and Net Zero - _<https://view.officeapps.live.com/op/view.aspx?src=https%3A%2F%2Fassets.publishing.service.gov.uk%2Fmedia%2F65b02faf1702b1000dcb111a%2FSubnational_gas_consumption_statistics_2005-2022.xlsx&wdOrigin=BROWSELINK>_

**\[3\] Estimates of the population for the UK, England, Wales, Scotland, and Northern Ireland 2011 to 2022** – Office of National Statistics - [_https://www.ons.gov.uk/peoplepopulationandcommunity/populationandmigration/populationestimates/datasets/populationestimatesforukenglandandwalesscotlandandnorthernireland_](https://www.ons.gov.uk/peoplepopulationandcommunity/populationandmigration/populationestimates/datasets/populationestimatesforukenglandandwalesscotlandandnorthernireland)

**\[4\] Local Authority District to Region (April 2020) Lookup in EN -** Office of National Statistics - [_https://geoportal.statistics.gov.uk/datasets/ons::local-authority-district-to-region-april-2020-lookup-in-en/explore_](https://geoportal.statistics.gov.uk/datasets/ons::local-authority-district-to-region-april-2020-lookup-in-en/explore)

**\[5\] Average annual domestic standard electricity bills and unit costs for regions in the United Kingdom** – Department of Energy Security and Net Zero - _<https://view.officeapps.live.com/op/view.aspx?src=https%3A%2F%2Fassets.publishing.service.gov.uk%2Fmedia%2F667c0b71aec8650b1009006d%2Ftable_223.xlsx&wdOrigin=BROWSELINK>_

**\[6\] Average prices of fuels purchased by the major UK power producers 1990 - 2023** - Department of Energy Security and Net Zero - _<https://view.officeapps.live.com/op/view.aspx?src=https%3A%2F%2Fassets.publishing.service.gov.uk%2Fmedia%2F66f3d0e19d78f6f2d75bf81a%2Ftable_321.xlsx&wdOrigin=BROWSELINK>_

**\[7\] Average earnings by age and region 2002 – 2024** – House of Commons Library - [_https://commonslibrary.parliament.uk/research-briefings/cbp-8456/_](https://commonslibrary.parliament.uk/research-briefings/cbp-8456/)

**\[8\] Geo-JSON of UK Regions** _-_ [_https://raw.githubusercontent.com/martinjc/UK-GeoJSON/refs/heads/master/json/eurostat/ew/nuts1.json_](https://raw.githubusercontent.com/martinjc/UK-GeoJSON/refs/heads/master/json/eurostat/ew/nuts1.json)

**\[9\] Area of each of the regions of England -** Wikipedia - <https://en.wikipedia.org/wiki/Regions_of_England>

_\-Please note dataset \[9\] was used in the code but the data was not in the end used for analysis-_

### Standardising the Regional Naming Conventions

#### Converting Local Authority Districts to Region Names

To be able to plot the regional data onto choropleths it was necessary to ensure that the regional naming convention for all data match that of the Geo-JSON. The chosen Geo-JSON (dataset \[8\]) used 3 letter location codes, which can be mapped from RGN20NM (regional codes), whereas the population dataset (dataset \[3\]) used LAD20CD (Local authority district code).

HERE SHOW conversion Table

A conversion table was used (see above) (dataset \[4\]) to join the corresponding LAD20CD values onto the population dataset.

CoDE HERE

Panda’s _.isna_ function was used to check for any failed matches. Some results were returned. To rectify this the missing values were added to the conversion table and the code reran.

#### Accounting for varying regional structures in data

The sub national gas consumption dataset \[2\] captured data at different granularity and with different naming conventions depending on the year. Some years London was grouped as a single row, others it was split over two as Inner London and Outer London, and have 3 rows London, Inner London and Outer London.

In the first case no cleaning was needed, and data was imported directly. In the second case both rows where imported. In the third case, only the row with a sum value for London was imported.

HERE CODE

To enable the creation of Choropleths it is needed to allocate a 3-letter location code to each region. This is done by completing a join using the Location_Code mapping table.

This mapping table also allowed me to clean inconsistencies in the naming of regions found in the data; mapping both ‘Inner London’ and ‘Outer London’ as ‘London’; and mapping inconsistencies such as 'Yorkshire and the Humber' to 'Yorkshire and The Humber'.

HERE CODE

After the join a group by was performed to sum the split values for London.

HERE CODE

The field \[Location Code\] was checked for NULL values

#### Grouping Overly Granular Data

The domestic energy bills dataset \[5\] had an additional regional grouping of south. I only had South East and South West I decided to split the South region by averaging its value first with the south east and then with the South West and replaced the east and west values with the averaged ones. This method was selected as the differences in values between South West and Southern, and South East and Southern where small, ~7% from Southern either way.

This could be more appropriately split using a 3<sup>rd</sup> variable to calculate a weighted split based for example population or regional area. This method was however chosen for simplicity and is acceptable variance between the regions is small.

### Extrapolation of Data

Dataset \[3\] was used to bring population for each region into the dataset. This source only covered values from the years 2011 to 2022, however, the other regional data covered 2005 to 2022.

Values had to be extrapolated for the years 2005 to 2010.

HERE POP GRAPH

Firstly, the data was plotted to visually determine the data’s trend. The lines produced appear to be mostly straight suggesting a linear relationship between population and time (Accounting Insights, 2024). Although a few outliers can be determined, such as the reduction in population in London in 2020 and 2021, likely caused by the emigration of individuals out of major population centres which occurred in the Covid-19 pandemic (Centre for Cities, 2024). Even with this outlier from the visual we can observe that the variance year to year is small; therefore, the error of extrapolated values is also likely to be small.

HERE CORR CODE TABVLE

Secondly, Panda’s _.corr_ function was used to calculate Pearson’s correlation values between population(for each region) and year (See the right most column). All values were 0.9 or greater indicating a strong linear correlation between population and year.

HERE POP EXTRAP GRAPH 1

Values for 2005 to 2010 were extrapolated using a linear regression model fit to all years of data using Sklearn. When these values are plotted there is an obvious mismatch across the years 2010 to 2011 where the extrapolated data meets the actual data, especially for London. This suggests the model has not been very successful in predicting datapoints outside the initial data range. This is likely due to the outliers created in the years 2020 to 2022 due to the Covid-19 pandemic.

HERE POP EXTRAP GRAPH 2

Another model was fit with datapoints from 2020 to 2022 removed from the training data. Values for 2005 to 2010 were extrapolated and plotted. The transition from extrapolated to real data here is much smoother.

HERE R2 TABLE FROM CODE

R^2 values were compared for each model to give an objective measure of predictive accuracy. model 2 outperformed model 1 across all regions. The points extrapolated from model 2 were used in the final dataset.  
HERE EXTRAP TABLE RESULTS

The final population dataset shown for London.

## Data Analysis

### The Relationship Between Fuel Import Price and Cost of Energy

Let us first focus on the claim that the Russia – Ukraine conflict was the cause for the large increase in energy prices. The claim was based on the logic that due to European embargoes and santions placed on Russia, the demand for gas in Europe would have to be met through alternative sources from the market, in turn increasing the market value of gas. The additional cost is then handed down to the consumer as increased energy bills (Jones2022).

The below gragh shows the average unit cost of energy (pence per kWh) to the consumer (dataset \[5\]), and the average unit cost of fuels in terms of their energy generating potential (pence per kWh) from varying imported fuels (dataset \[6\]).

HERE GRAPH

This graph largely supports the logic set out above. In 2022 the price of gas reached 4.86 pence per kWh which coincided with a steep rise in the cost to consumer up to 33.61 pence per kWh. It is worth noting however that prices were already on the rise in 2021, and the 2022 spike was only an exacerbation to this trend.

The relationship between import price of gas and cost to consumer is less obvious before this spike. The correlation matrix below allows for evaluation of the correlation between factors less subjectively.

HERE CORR MATRIX

The correlation matrix shows that the cost of gas imports is the best predictor (of the fuel types shown here) of the unit cost of energy, with a p-value of 0.88.

This should not be surprising as UK energy pricing policy dictates that there is a direct relationship between the two. We will cover this policy in the next section.

What is surprising is that the year was the best predictor of average unit cost of energy with a p-value of 0.9. This is not caused by the effects of inflation as the data has been deflated (see the cover sheet of data source \[6\]) (Francis-Devine, 2022). This very strong positive correlation suggests that the price of energy inevitably increases with time irrespective of external factors. Although this cannot be stated with confidence as the p-vales of 0.88 and 0.9 are so similar.

In reality, this is a complicated and multifactor issue, the scope of which is not fully explored in this project. The effects of energy company operating costs; the lack of energy storage; the Ofgen price cap; the energy price guarantee; the availability and demand of freight and energy industry workers have not been considered. A large and thorough investigation would be needed to determine the true causes of the 2022 energy crisis.

#### Hypothesis Conclusion

The trend indicated in the line chart and the strength of the correlation between the import price of gas and the unit cost of energy, both support the hypothesis. From this analysis we can conclude that the rise in the price of importing gas due to the Russian Ukrainian conflict has contributed to the increase of energy prices in the UK in 2022.

### Future Energy Policy

In the following section UK energy policy will be covered. Data science methods will be employed to explore regional trends in energy consumption and energy affordability as well as the potential affects which changes in energy policy may have.

### The UK Energy Market

In 2022 the Review of Electricity Market Arrangements (REMA) consultation was launched to tackle the root cause of high energy prices and to support the pivot to decarbonate the UK energy market. The consultation concluded that Locational Marginal Pricing (LMP), also called ‘Zonal Pricing’, may be beneficial to implement in place of a national marginal pricing (Mullane, 2024).

In this section the current policy which sets energy prices (Marginal Pricing) will be explored. Data science methods will then be used to contextualise what the Zonal Pricing policy change, as suggested by REMA, may mean.

#### Marginal Pricing

‘Marginal cost pricing is where units of electricity are sold at the price of the most expensive unit needed to meet demand at a particular moment in time.’ – Stewart (2023)

HERE MARG PRICE PICTURE

Stewart (2023)

Please note the above visual does not use actual values and it merely to demonstrate how marginal pricing works. Gas is very often the most expensive fuel source required to meet demand, and therefore sets the price of a unit of energy (Stewart, 2024).

HERE DOM ENERGY MARKET PIC

(Stewart, 2024)

The above diagram shows the role of marginal pricing in defining the wholesale energy price and how this has a knock on to consumer energy pricing.

What marginal pricing means in practice is that a unit cost to generate energy, which is higher than what it cost, is assigned. Due to this system, there is little incentive provided to energy generating companies to reduce the price of the most expensive way of generating energy.

Marginal pricing has also been criticised as running opposed to the UK government’s own targets to fully decarbonise the power sector by 2035 (IEA, 2024).

HERE GRAGH

HERE BAR GRAGH

The consumption of energy from gas has fallen consistently with time in the UK. However, very little change in the proportion of total energy consumed via gas has been seen. Which has consistently stayed at ~34% since 2010, omitting 2020 and 2021. This supports the claim that marginal pricing does not provide enough incentive to reduce reliance on gas.

#### Zonal Pricing

Zonal Pricing, also referred to as Locational Marginal Pricing (LMP), is the pricing policy substitute which has been suggested by REMA.

Zonal Pricing is an alternative way to set unit prices in the wholesale energy market. LMP is calculated similarly with regards to setting the unit price based on the most expensive type of supply, but this unit cost is allocated per zone rather than nationally. This is a credible way to restructure energy pricing and would encourage local competition between energy companies to drive costs down. In turn, encouraging energy companies to focus on producing more renewable energy to stay locally competitive (Farah, 2024).

Critics of Zonal pricing argue that the new structure would not address root causes of energy inefficiencies stemming from inadequate grid infrastructure (Renewableuk.com, 2019).

#### Nodal Pricing

Nodal Pricing is the most granular pricing strategy currently being proposed. In this form of pricing, the unit cost of energy is set based several factors which are taken as readings from over a thousand nodes across the UK. It accounts for local patterns of load, generation, and the physical limits of transmission system. Using a nodal approach would help to drive infrastructure efficiency and ensure price transparency by reflecting real-time supply and demand conditions at each node (Diversegy, 2024).

### The Choropleth Cheat Sheet

The following section is heavily reliant on the use of choropleths (map graphs) to interpret regional data. To make the meanings of these easier to interpret a cheat sheet table has been included below.

HERE CHET SHEET TABLE

### The Effects of Zonal Pricing

As Zonal Pricing is the most likely energy price policy reform to be implemented it is worth investigating further.

Should Zonal Pricing be implemented it would undoubtably be beneficial as it can only decrease the unit cost of energy or at worst keep it the same. This should be kept in mind throughout the following section where the effects of Zonal Pricing on regional inequality are examined. Even if the threat of increasing inequality due to regional pricing is observed, it should be noted that this is against a backdrop of reduced energy costs. In theory, everyone will pay less, it is just that some will pay less than others. The risk of Zonal Pricing increasing regional inequality is more concerning once you consider that there will be pressure placed on Ofgen by energy companies who will look to redress their loss in profits via another mechanism.

Please note the regions shown in the choropleth visualisation below do not align with the zones which would determine zonal pricing. Zonal Pricing has not currently been implemented and the regions it may use have not been published.

#### Who Uses Gas

The figure below shows the energy generated from gas consumed per person by region between 2005 and 2022. In this figure the scale is set to plot the colour of the choropleth between the maxima and minima of the whole set not just for each year. This means we can use this figure to compare regional trends as well as trends over time.

The figure shows a consistent trend towards less gas usage, as we would expect from figure (THIS WILL BE THE RED AND BLUE LINE ONE). It also shows a regional trend of greater gas consumption in the north of England. The largest consumers in 2022 where the East Midlands and Yorkshire and The Humber, with a value of 5.2 MWh/person, the lowest consumer was the South East, with a value of 3.3 MWh/person.

Please note this figure only plots domestic energy consumption so will not be skewed by industry energy use.

HERE C1

#### Who Would be Affected the Most by Zonal Pricing

The figure below shows the energy generated from gas consumed per person as a portion of the median salary by region between 2005 and 2022. This can be thought of as a measure to demonstrate who would be affected most in the case that regions are penalised for gas consumption. In this figure the scale functions independently for each year. The colour scale is stretched between the maxima and minima of the for each year. This means we can use this figure to compare regions proportional to one another but cannot observe trends over time.

You can see that consistently Yorkshire and The Humber is the most vulnerable to a regional increase in the price of gas, with a 2022 value of 1.66e-07 (GWh/person/£), and the South East being the least vulnerable to a regional increase in the price of gas, with a 2022 value of 0.93e-07 (GWh/person/£).

HERE C2

#### Has the Proportional Vulnerability to Regional Pricing Changed Over Time

The figure below also shows the energy generated from gas consumed per person as a portion of the median salary by region between 2005 and 2022. This time each year is plotted as a percentage difference from the minimum that year. The scale is also fixed between the minimum and maximum for the dataset, allowing us to compare the trend over time as well the difference between regions.

We can see that there is a persistent trend of still inequality in the vulnerability to change in gas price, with the North being more vulnerable and the South less vulnerable. This gap in inequality is shrinking, this can be seen as the colour difference between the North and South is decreasing.

HERE C3

#### Who is The Energy Crisis Affecting

Below is the figure from before showing the trends of Average Unit Cost of energy (pence per kWh) and Average Unit Cost to Generate Energy by Different Fuels (pence per kWh). Median salary (£’000) has now been added to demonstrate the fast decline in affordability.

Although median salaries have risen over time, they have not kept up with the rise in the unit cost of energy.

HERE Gragh with sal

The below figure shows the unit cost of energy as a proportion of median salary of each region and year. The minima and maxima of the colour scale have been fixed such that we can compare both regional trends and trends over time.

The trend regionally shows that energy is consistently more affordable in the south of England, particularly in London. However, over time all regions are finding energy less affordable as salaries fail to keep up with energy prices.

HERE c4

#### Inequality in the Affordability of Energy

The figure below plots the affordability of energy (unit energy price as a proportion of salary) as a percentage difference from the annual minimum. We can use it to explore trends of inequality in energy affordability by region and year.

2007 saw the greatest inequality in energy affordability with energy in the North East being 50% less affordable than in London. Over time, the affordability of energy has got more equal. In 2022 all regions were less than 30% less affordable than London.

HERE c5

### Future Energy Policy Conclusion

The introduction of Zonal energy pricing holds promises as a way to reduce the burden of energy prices as felt by the consumer, and to encourage energy sector competition to decarbonate the energy market. Legislators should however be aware of the implicit risk to regional inequalities.

The gap in energy inequality between the north and south of England has been reducing with time (see choropleth 5). The north south differences in the vulnerability to Zonal pricing have also diminished with time (see choropleth 3). However, with time energy has been getting less affordable with time (see choropleth 4 and figure ONE WITH ORANGE LINE).

The move to Zonal pricing has potential to penalise regions which are more reliant on gas as a source of energy. These regions overlap conspicuously with those which find energy least affordable (compare choropleths 1 and 5). Regions where the median salary as a proportion of gas consumption would be most vulnerable to any additional cost under Zonal energy pricing (see choropleth 2), this also overlaps conspicuous with the regions which already find energy least affordable.

One benefit which marginal pricing has had is to reduce the energy inequality between regions (see choropleth 5). Unless energy companies are willing to take a cut to their profit should Zonal pricing be introduced, then north south energy inequality may be exacerbated greatly. Particularly as energy companies look for alternative mechanisms with which to maintain their profitability, which may impact regions unequally; and as businesses relocate to make the most of reduced energy prices, in other regions.

In conclusion Zonal energy pricing has great potential to assist the energy crisis in England but does carry some risks of increasing regional inequality. The details of how it is implemented will determine its success in tackling fuel poverty in England.

# References

‌Accounting Insights. (2024). _Extrapolation Techniques in Financial Forecasting: Methods and Applications_. \[online\] Available at: <https://accountinginsights.org/extrapolation-techniques-in-financial-forecasting-methods-and-applications/>. \[Accessed 29 Dec. 2024\].

Borrett, A. (2022). _Cost of living crisis: How the war in Ukraine is eroding living standards in the UK_. \[online\] Sky News. Available at: <https://news.sky.com/story/cost-of-living-crisis-how-the-war-in-ukraine-is-eroding-living-standards-in-the-uk-12568365>. \[Accessed 18 Dec. 2024\].

‌Centre for Cities. (2024). _Population decline was driven by people moving out of London - Centre for Cities_. \[online\] Available at: <https://www.centreforcities.org/reader/escape-to-the-country/population-decline-was-driven-by-people-moving-out-of-london/>. \[Accessed 29 Dec. 2024\].

Clark, J. (2022). _Five things you need to know about rising energy and petrol bills due to Russia and Ukraine war..._ \[online\] The Sun. Available at: <https://www.thesun.co.uk/money/17778294/rising-energy-petrol-bills-russia-ukraine-war/> \[Accessed 18 Dec. 2024\].

‌Diversegy (2024). _Nodal Pricing vs. Zonal Pricing: Understanding Different Electricity Pricing Models_. \[online\] Diversegy. Available at: <https://diversegy.com/nodal-pricing-vs-zonal-pricing/> \[Accessed 24 Dec. 2024\].

Farah, S. (2024). _How zonal pricing could make bills cheaper_. \[online\] Octopus Energy. Available at: <https://octopus.energy/blog/regional-pricing-explained/>. \[Accessed 24 Dec. 2024\].

Francis-Devine, B. (2022). Average earnings by age and region. _commonslibrary.parliament.uk_. \[online\] Available at: <https://commonslibrary.parliament.uk/research-briefings/cbp-8456/>.

Francis-Devine, B. (2024). Poverty in the UK: Statistics. _commonslibrary.parliament.uk_, \[online\] 7096(7096). Available at: <https://commonslibrary.parliament.uk/research-briefings/sn07096/>.

IEA (2024). _Executive summary – United Kingdom 2024 – Analysis - IEA_. \[online\] IEA. Available at: <https://www.iea.org/reports/united-kingdom-2024/executive-summary>. \[Accessed 24 Dec. 2024\].

Jones, L. (2022). Five ways the Ukraine war could push up prices. _BBC News_. \[online\] 4 Mar. Available at: <https://www.bbc.co.uk/news/business-60509453>. \[Accessed 18 Dec. 2024\].

Mullane, J. (2024). _How your energy prices could soon depend on where you live in the UK_. \[online\] Homebuilding & Renovating. Available at: <https://www.homebuilding.co.uk/news/how-your-energy-prices-could-soon-depend-on-where-you-live-in-the-uk>. \[Accessed 24 Dec. 2024\].

News, BBC. (2022). _Ukraine war fuels new surge in energy prices - BBC News_. _YouTube_. Available at: <https://www.youtube.com/watch?v=23q6eNufxZQ>. \[Accessed 18 Dec. 2024\].

Renewableuk.com. (2019). _RenewableUK_. \[online\] Available at: <https://www.renewableuk.com/news-and-resources/blog/why-it-s-time-to-move-on-from-zonal-pricing/>. \[Accessed 24 Dec. 2024\].

Signify. (2023). _Five approaches to combat the UK’s energy crisis | Signify_. \[online\] Available at: <https://www.signify.com/en-gb/blog/sustainability/how-to-combat-the-uk-energy-crisis> \[Accessed 29 Dec. 2024\].

Stewart, I. (2023). _Why is cheap renewable electricity so expensive on the wholesale market?_ \[online\] House of Commons Library. Available at: <https://commonslibrary.parliament.uk/why-is-cheap-renewable-electricity-so-expensive/>. \[Accessed 24 Dec. 2024\].

Stewart, I. (2024). _Introduction to the domestic energy market_. \[online\] House of Commons Library. Available at: <https://commonslibrary.parliament.uk/research-briefings/cbp-9768/>. \[Accessed 24 Dec. 2024\].
