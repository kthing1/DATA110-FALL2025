# Project 1 Report
Kavya Thing
>[Link to Slides](https://slides.com/kavyathing/copy-of-kvthing-project-1-excess-deaths/) \
>[Link to GitHub Colab File](https://github.com/kthing1/DATA110-FALL2025/blob/data110-fall2025/Project1/KTHING_project1_colab.ipynb)

## Introduction

During the COVID-19 pandemic, the news would often show a heatmap of COVID cases by state (left). It used *total number of cases*. Nearly all of the time, the map would show states like CA, TX, FL, NY in the darkest (highest number of cases) color, and states like WY, AK, ND, SD, in the lightest (lowest number of cases) color. Seeing the largest and smallest states grouped like this made me wonder if the map was any different from a heatmap of population by state, so I pulled 2020 Census data and created a population heatmap (right). It seems there were only a handful of differences.

<img
        src="https://github.com/kthing1/DATA110-FALL2025/blob/data110-fall2025/Project1/img_covid-2020.jpg" 
        width=49%
        title="Image of MSNBC COVID-19 Cases Heatmap"
        alt="Image of MSNBC COVID-19 Cases Heatmap"
    />  &nbsp; <img
        src="https://github.com/kthing1/DATA110-FALL2025/blob/data110-fall2025/Project1/img_pop-2020.jpg" 
        width=49%
        title="Image of MSNBC COVID-19 Cases Heatmap"
        alt="Image of MSNBC COVID-19 Cases Heatmap"
    />

It isn't surprising to find a correlation between the number of COVID cases and the population. It does, however, invite the question of whether the COVID cases heatmap is a "bad data visualization." That depends on what question the visualization is addressing as well as *how the audience would naturally interpret the visualization*. Most people would expect a COVID cases heatmap on the news to inform them of the risk level in their state.

As *How to Lie with Statistics* notes, "More people were killed by airplanes last year than in 1910. Therefore modern planes are more dangerous? Nonsense. There are hundreds of times more people flying now, that's all." Instead, the book advises people trying to understand their risk to "*Get the rate*... That will come closest to telling you where your greatest risk lies."

For this project, I chose to use another public health dataset to explore the importance of understanding what variables are influencing each other and how those influences impact a chosen data visualization. How can it change the story the data visualization *appears* to tell? What can we do to make sure visualizations we produce are meaningfully addressing the question we're interested in answering? And how do we effectively communicate the story of the data to our audience without (intentionally or unintentionally) misleading them?

My chosen dataset is *Potentially Excess Deaths from the Five Leading Causes of Death* produced by the National Center for Health Statistics (NCHS). Per the [NCHS](https://www.cdc.gov/nchs/nvss/potentially_excess_deaths.htm), "Potentially excess deaths are calculated by subtracting the expected number of deaths from the observed number of deaths. The expected number refers to the number of deaths that we would see if that state’s death rate was equal to the best-performing states (benchmark states)."

## Exploratory Data Analysis (EDA)
I began my exploratory data analysis (EDA) by performing the `info` function to identify the columns in the dataset, then viewing the `head` to understand the application of these columns.

### Variables
Unordered Categories:
- **5** Leading Causes of Death
  - Cancer
  - Chronic Lower Respiratory Disease
  - Heart Disease
  - Stroke
  - Unintentional Injury
- **51** States & DC + U.S. Total
  - Columns for both full names and FIPS codes
- **10** HHS Regions + U.S. Total
  - Numbered but not considered ordered because the number assignment is not tied to an ordered value
  - Groups states together, but doesn't create additional rows like the rest of the categories
- **3** Benchmarks
  - 2005 Fixed
  - 2010 Fixed
  - Floating
- **2** Locality types + total for both
  - Metropolitan
  - Nonmetropolitian

Ordered Categories:
- **11** Years (2005 through 2015)
- **8** Cumulative Age Ranges (0-49 through 0-84, increasing 5 years with each expansion)

Quantitative Values:
- Population
- Observed Deaths
- Expected Deaths
- Potentially Excess Deaths
- Percent Potentially Excess Deaths (ratio of Potentially Excess Deaths to Observed Deaths)

### Null Values
```
RangeIndex: 205920 entries, 0 to 205919
Data columns (total 13 columns):
 #   Column                             Non-Null Count   Dtype  
---  ------                             --------------   -----  
 0   Year                               205920 non-null  int64  
 1   Cause of Death                     205920 non-null  object 
 2   State                              205920 non-null  object 
 3   State FIPS Code                    205920 non-null  object 
 4   HHS Region                         205920 non-null  int64  
 5   Age Range                          205920 non-null  object 
 6   Benchmark                          205920 non-null  object 
 7   Locality                           205920 non-null  object 
 8   Observed Deaths                    195708 non-null  float64
 9   Population                         200640 non-null  float64
 10  Expected Deaths                    195708 non-null  float64
 11  Potentially Excess Deaths          195708 non-null  float64
 12  Percent Potentially Excess Deaths  195708 non-null  float64
dtypes: float64(5), int64(2), object(6)
```

While there's an overall count of 205920 rows of data, there are several null values among the quantitative data. Based on a sampling of these null values, they appear to be largely due to (a) physically smaller states, such as Delaware and Rhode Island, that do not have non-metropolitan localities and (b) younger age ranges that have no expected or observed deaths for some causes of death in certain states and locality types.

### Distribution
While I did perform `describe` on the overall dataset, the distribution information is not particularly enlightening because it includes both state and U.S. total values. The `pop_focus` and `age_focus` data frames provide more insight with `describe`, particularly for appreciating the range of Percent Potentially Excess Deaths across the states and confirming that the age ranges do not expand with equal populations.

### Setting Parameters
I created two major data frames: `pop_focus` to explore the influence of state populations on other quantitative values; and `age_focus` to explore the influence of age range populations on the number of observed deaths. With both data frames, I set the following parameters:
```
['Year'] == 2015
['Benchmark'] == 'Floating'
['Locality'] == 'All'
```
For the Population Focus data frame, I removed the U.S. totals values and selected a single Age Range and Cause of Death so that trends would be clear.
```
['State'] != 'United States'
['Age Range'] == '0-84'
['Cause of Death'] == 'Stroke'
```
For the Age Focus data frame, I only used U.S. totals, and I created subframes for each Cause of Death. Most critically, I created additional calculated columns to identify the Population and Observed Deaths within each non-cumulative age range ("Age Bin"), then found the Death Rate by Age Bin.
```
age_focus['DeltaObDe'] = age_focus.groupby('Cause of Death')['Observed Deaths'].diff()
age_focus['DeltaPop'] = age_focus.groupby('Cause of Death')['Population'].diff()
age_focus['Age Bin'] = age_focus.apply(label_ab, axis=1)
age_focus['DeltaDeathRate'] = age_focus['DeltaObDe'] / age_focus['DeltaPop'] * 100000
```
### Correlation Matrix
![Image of Correlation Matrices](https://github.com/kjdatamc/Data110/blob/c58660956d6ac3963ac42a9003e67ad2b11da14f/Project1/img_correlation_matrix.png)

I created Correlation Matrices for both the overall dataset and the Population Focus (`pop_focus`) data frame. There's a noticeable difference, specifically how comparatively low the correlation between population and other absolute values was for the overall dataset. That's likely because the overall dataset contains multiple causes of death for the same population size, as well as different age ranges, locality types, and years that each have a different death rate for their respective populations. This would, of course, reduce the apparent correlation. When these variables are removed, as in the more limited Population Focus data frame (which has a single cause of death, age range, locality type, and year), we see a stronger correlation between population and the other quantitative values.

## Data Visualization

### Potentially Excess Deaths by Region
When visualizing the potentially excess deaths by HHS Region, is it better to look at the regional total of the absolute number of potentially excess deaths or the regional percent potentially excess deaths? That depends on the question being investigated and what the audience is expecting from the data visualization.

![Image of Regional Bar Graphs](https://github.com/kjdatamc/Data110/blob/c58660956d6ac3963ac42a9003e67ad2b11da14f/Project1/img_regional_barchart.png)

On the left, in red, is a bar graph of the regional *total number of potentially excess deaths*. If you want to determine which regions need more funding for health programs, the first graph might be most useful.

On the right, in blue, is a bar graph of the regional *percent potentially excess deaths*. If you're trying to determine which regions are furthest from meeting the benchmark, to give your audience a sense of the risk in each region, the second graph is more meaningful.

Based on the first graph, one might naturally assume that HHS Region 4, which has nearly twice the number of deaths of any other region, is struggling much more than the others. Yet the second graph shows HHS Region 6 actually has the highest proportion of deaths that are potentially excess. So why are these two bar graphs so different? The simple answer is *population*.

### Influence of State Population on Other Quantitative Values
A larger population, generally, will mean more chances for something to happen. The HHS Regions have different populations, and of course the states comprising each region have different populations. In plotting the state populations against other quantitative values, we can see a picture emerge of the level of influence population has on these variables.

![Image of Population Scatter Plots](https://github.com/kjdatamc/Data110/blob/c58660956d6ac3963ac42a9003e67ad2b11da14f/Project1/img_population_scatterplot.png)

On the left, in red, is a scatter plot of the expected deaths by state population, along with a line of regression. Given that expected deaths are calculated based on a benchmark deathrate and a state's population, it's unsurprising to see a very strong correlation - nearly perfectly aligned to the line of regression. There are a few slight outliers, such as one at around 20M population and just over 4K expected deaths. This is Florida, a known retirement destination and likely a state with a larger portion of the population that's older. Sitting so far above the line of regression suggests that the age of the population could also a variable influencing the calculation of expected deaths.

In the center, also in red, is a scatter plot of potentially excess deaths by state population, along with a line of regression. The correlation is weaker, but it is still visibly present. Population clearly influences this quantitative value, but it appears that other variables must also have significant influence.

Finally, on the right, in blue, is a scatter plot of percent potentially excess deaths by state population. There is no visible correlation between these variables because each percent is a ratio within a single population; essentially the population is being factored out. That makes it a preferable quantitative value to use for identifying trends or risk levels because it doesn't have population muddling the data.

### Leading Cause of Death Across Cumulative Age Ranges
What if we want to see how the leading causes of death in the U.S. change with age? We could plot the national total observed deaths by age range for each of the five causes. And we see that deaths from unintentional injury, which begin as the leading cause, then appear to maintain a fairly linear trend. Meanwhile, deaths from the other causes appear to increase exponentially as we age. I say appear because appearances can be deceiving.

![Image of Cause of Death by Age Line Graph](http://github.com/kthing1/DATA110-FALL2025/blob/data110-fall2025/Project1/img_cause_of_death_lineplot.png)

By looking at the national total, we eliminate the issue of different state populations influencing the absolute number of observed deaths. But we haven't really eliminated the influence of population. The dataset contains cumulative age ranges, which by definition can't be equal populations because they are accumulating population with each five-year expansion. What trends could that population influence be hiding?

### A Hidden Trend Across Age Ranges
To find the hidden trends, the data must be broken out into non-cumulative age ranges. Essentially, what is the delta between the quantitative values of each adjacent age range? This includes both the difference in population and the difference in observed deaths. Those values will also be used to calculate a death rate. We'll focus on deaths by unintentional injury, which appeared to be the only cause of death that didn't increase exponentially with age.

![Image of Unintentional Injury Hidden Trend Line Graph](https://github.com/kthing1/DATA110-FALL2025/blob/data110-fall2025/Project1/img_unintentional_injury_lineplot.png)

On the left, in red, is a recreation of the previous line plot focused only on the observed deaths from unintentional injury by cumulative age range. As before, it appears to be a fairly linear trend that many audiences might assume means the risk doesn't change much as you age. 

In the center, in gold, is a line plot of the observed deaths from unintentional injury by *non-cumulative age ranges* ("age bins"). We see a very different trend emerge, one that seems to show our risk decreasing significantly until our late 70s when it starts to rise again. But is this an accurate picture of our risk? We removed the cumulative population influence, but we're still looking at an absolute number of deaths from different size populations. Each five-year age bin has a different population, which generally decreases as the age bin increases.

On the right, in blue, is a line plot that uses the ratio of observed deaths to the population of each age bin, plotting a death rate by age bin that gives us the best sense of how our risk of death by unintentional injury changes as we age. Here we see a much smaller decrease in risk through our 50s and 60s. Then in our early 70s, the death rate (and risk) begins increasing and continues increasing so it nearly triples between our late 60s and early 80s. This change in risk was invisible in the original line plot.

### The Terrible, Horrible, No Good, Very Bad Data Visualization
![Image of Bad Pie Chart](https://github.com/kthing1/DATA110-FALL2025/blob/data110-fall2025/Project1/img_bad_piechart.png)

While a pie chart should normally be used to show the relative proportion of different categories that make up a whole, this data visualization is not doing that. The title and choice of pie graph could imply this is showing the portion of all excess deaths that are from each cause of death. Instead these meaningless "portions" come from sums of the percent potentially excess deaths from all the cumulative age ranges in the data frame (ratios with different denominators). There's no reason to sum ratios that aren't portions of a whole since they have different denominators.

Further, there's no reason to display ratios from multiple categories (again, different denominators) in a pie chart, implying they're portions of a whole when they are *not*. Even if they were accurate percent potentially excess deaths values from each cause of death, they still wouldn't share a denominator and so shouldn't be used for a pie chart. This is a bad data visualization on every level, and it would only confuse or mislead the audience.

## Conclusion

### Key Findings
Throughout my analysis, I explored the ways the same underlying dataset could be used to produce strikingly different data visualizations thanks to the influence of population on other quantitative values. When visualizing trends across different populations, the size of each population had a clear impact on absolute values from each population. For some questions, a rate would be the more appropriate quantitative value to use for analysis and data visualization.

All of the visualizations I produced are *accurate* and have valid use-cases (except the pie chart), but they could easily be misleading if the audience isn't able to understand the question each addresses *at a glance*. Small captions or paragraphs of explanation won't always effectively counter the first impression of a data visualization. That's why it's crucial to make sure your data visualization tells the honest, meaningful, and coherent story that you intend to communicate. Understand the context in which it will be viewed, and then select the appropriate data, visualization type, axes, and title to limit the chance of misinterpretation. And when looking at data visualizations made by others, consider whether they have done the same.

### Further Investigation
With this dataset, I was surprised to find it used cumulative age ranges rather than distinct (non-cumulative) age ranges. This approach could easily hide trends or minimize their apparent intensity. In my exploration of observed deaths due to unintentional injury, I found the hidden trend of increasing risk starting in one's 70s. That wasn't the focus of the dataset, but it makes one wonder what other trends may be concealed due to the choice of cumulative age ranges?

Given that HHS Regions 4 and 6, along the southern half of the U.S., had the highest percent potentially excess deaths from stroke, it would be good to identify if there are similar regional trends for the other causes of death. I would also recommend creating a heatmap by state and potentially by locality type (metropolitan vs non) to determine if there are any trends that may be hidden by the somewhat artificially created HHS Regions.

Finally, this dataset appears to be good at identifying a problem -*potentially excess deaths*- but not at offering anything to help identify possible causes. What other variables could be tested for correlation with percent potentially excess deaths? For example: median income, prevalence of certain industries (thinking specifically of mining potentially correlating with chronic lower respiratory disease), healthcare accessibility, Medicare/Medicaid expansion, etc. While this dataset is valuable, if the ultimate goal is to prevent the potentially excess deaths, further investigation of the causes will be necessary.
