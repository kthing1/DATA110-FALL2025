# Project 1 Report
Kavya Thing
>[Link to Slides](https://slides.com/kavyathing/copy-of-kvthing-project-1-excess-deaths/) \
>[Link to GitHub Colab File](https://github.com/kthing1/DATA110-FALL2025/blob/data110-fall2025/Project1/KTHING_project1_colab.ipynb)

## Introduction

During the COVID-19 pandemic, the news often showed maps that used color to display how many COVID cases each U.S. state had. These were called heatmaps. Most of the time, big states like California, Texas, Florida, and New York were shown in the darkest colors because they had the highest total number of cases. Smaller states like Wyoming, Alaska, North Dakota, and South Dakota were shown in the lightest colors, meaning they had fewer total cases. Seeing this pattern made me wonder if the COVID map really looked any different from a map showing just population by state. To find out, I used data from the 2020 Census to make a population heatmap. When I compared the two maps, I noticed they looked very similar, with only a few small differences.

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

It makes sense that there’s a link between how many people live in a state and how many total COVID cases it has. But it raises an important question: does that make the COVID heatmap a poor data visualization? The answer depends on what question the map is supposed to answer—and how the viewers are likely to understand it. Most people watching the news would probably assume the COVID heatmap shows how much risk there is of getting sick in their state.

In the book *How to Lie with Statistics*, the author gives a similar example: “More people were killed by airplanes last year than in 1910. Therefore modern planes are more dangerous? Nonsense. There are hundreds of times more people flying now, that’s all.” The book reminds us that to truly understand risk, we need to look at rates, not raw numbers. That means considering cases or deaths per person (or per population), instead of just total counts. Rates give a more accurate picture of where the real risks are.

For this project, I used another public health dataset to explore this idea—how understanding relationships between variables can change what a data visualization seems to say. The dataset I chose is Potentially Excess Deaths from the Five Leading Causes of Death, published by the National Center for Health Statistics (NCHS). The [NCHS](https://www.cdc.gov/nchs/nvss/potentially_excess_deaths.htm) explains that “potentially excess deaths” are found by subtracting the number of expected deaths from the actual number of deaths. The expected number is based on how well the best-performing states are doing.

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

The full dataset contains 205,920 rows, but there are several missing (null) values in the numerical data. After reviewing a sample of these missing values, I found that most of them come from two main causes:
(a) smaller states like Delaware and Rhode Island, which don’t have non-metropolitan areas, and
(b) younger age groups that have no expected or observed deaths for certain causes of death in specific states or locality types.

### Distribution
Although I used the `describe` function on the full dataset, the overall distribution wasn’t very helpful because it included both individual state values and totals for the entire U.S. To get a clearer picture, I focused on two smaller data frames: `pop_focus` and `age_focus`. These provided more useful summary statistics. They helped me better understand how the Percent Potentially Excess Deaths varied across states and confirmed that the population sizes in each age group aren’t evenly distributed.

### Setting Parameters
I built two main data frames for deeper analysis:

`pop_focus` — to study how a state’s population size might influence other numerical variables, and

`age_focus` — to study how the size of each age group might affect the number of observed deaths.

For both data frames, I set up consistent parameters to make comparisons easier and ensure the analysis stayed focused on how population and age factors relate to death statistics.
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
![Image of Correlation Matrices](https://github.com/kthing1/DATA110-FALL2025/blob/data110-fall2025/Project1/img_correlation_matrix.png))

I created Correlation Matrices for both the overall dataset and the Population Focus (`pop_focus`) data frame. There's a noticeable difference, specifically how comparatively low the correlation between population and other absolute values was for the overall dataset. That's likely because the overall dataset contains multiple causes of death for the same population size, as well as different age ranges, locality types, and years that each have a different death rate for their respective populations. This would, of course, reduce the apparent correlation. When these variables are removed, as in the more limited Population Focus data frame (which has a single cause of death, age range, locality type, and year), we see a stronger correlation between population and the other quantitative values.

## Data Visualization

### Potentially Excess Deaths by Region
When visualizing the potentially excess deaths by HHS Region, is it better to look at the regional total of the absolute number of potentially excess deaths or the regional percent potentially excess deaths? That depends on the question being investigated and what the audience is expecting from the data visualization.

![Image of Regional Bar Graphs](https://github.com/kthing1/DATA110-FALL2025/blob/data110-fall2025/Project1/img_regional_barchart.png))

On the left (in red) is a bar chart showing the *total number of potentially excess deaths* by region. This version is useful if your goal is to figure out which regions might need more funding or resources for health programs, since it shows the overall impact in terms of raw numbers.

On the right (in blue) is a bar chart showing the *percent of potentially excess deaths* by region. This version is more helpful if you want to understand which regions are the furthest from reaching the benchmark for expected deaths. In other words, it gives your audience a clearer sense of the relative risk in each region.

Looking only at the first chart, it’s easy to think that HHS Region 4 is doing far worse than any other area, since it has almost twice as many deaths as the next region. But when we look at the second chart, we see a different story—HHS Region 6 actually has the highest percentage of potentially excess deaths. So why do these two charts seem to tell different stories? The simple explanation is *population size*.

### Influence of State Population on Other Quantitative Values
In general, a larger population means more opportunities for any event (such as illness or death) to occur. The HHS Regions each have different total populations, and the states within those regions also vary widely in size. When we plot state population against other numerical variables, we start to see how strongly population size influences the results. This helps us understand which patterns are truly meaningful and which ones simply reflect how many people live in each region.

![Image of Population Scatter Plots](https://github.com/kthing1/DATA110-FALL2025/blob/data110-fall2025/Project1/img_population_scatterplot.png)

On the left (in red) is a scatter plot showing expected deaths versus state population, with a regression line included. Since expected deaths are calculated using a benchmark death rate multiplied by the state’s population, it’s not surprising to see a very strong correlation here—the points fall almost perfectly along the line. There are a few small outliers, like one around a population of 20 million and just over 4,000 expected deaths. That point represents Florida, a popular retirement destination with a relatively older population. Its position above the regression line suggests that the age makeup of a state may also affect the expected number of deaths.

In the center, also in red, is a scatter plot of potentially excess deaths by state population, again with a regression line. The relationship here is weaker, but still noticeable. Population size clearly plays a role, yet other factors must also contribute significantly to the number of excess deaths.

Finally, on the right (in blue), is a scatter plot of percent potentially excess deaths versus state population. This time, there’s no visible correlation. That’s because the percentage is already a ratio within each state’s population—so population effects have been removed. This makes percent potentially excess deaths a more useful metric for spotting real patterns or differences in risk across states, since it isn’t distorted by population size.

### Leading Cause of Death Across Cumulative Age Ranges
What if we want to see how the leading causes of death in the U.S. change as people age? One way to explore this is by plotting the national total of observed deaths for each cause across different age ranges. The results show that unintentional injury starts as the leading cause of death among younger age groups and then follows a fairly steady, almost linear increase. The other causes of death, however, seem to rise sharply with age—almost exponentially. The keyword here is seem, because visual patterns like this can be misleading if we don’t consider what’s happening behind the data.

![Image of Cause of Death by Age Line Graph](http://github.com/kthing1/DATA110-FALL2025/blob/data110-fall2025/Project1/img_cause_of_death_lineplot.png)

By looking at the national total, we remove the problem of different state populations affecting the total number of observed deaths. However, we haven’t actually removed the influence of population. The dataset uses cumulative age ranges, meaning each age group includes everyone from all younger groups as well. Because of that, the population sizes of these ranges aren’t equal—each new group contains more people than the one before it. So what patterns might be hidden by this uneven population distribution?

### A Hidden Trend Across Age Ranges
To uncover hidden trends, we need to break the data into non-cumulative age ranges. In other words, we should look at the difference between each consecutive age range—both in population and in observed deaths. With those values, we can also calculate a death rate. For this example, we’ll focus on deaths from unintentional injury, which was the only cause of death that didn’t appear to increase exponentially with age in the earlier charts.

![Image of Unintentional Injury Hidden Trend Line Graph](https://github.com/kthing1/DATA110-FALL2025/blob/data110-fall2025/Project1/img_unintentional_injury_lineplot.png)

On the left (in red) is a remake of the earlier line plot, showing observed deaths from unintentional injury by cumulative age range. As before, the line looks fairly straight, suggesting that the risk doesn’t change much as people get older. Many viewers might assume that risk stays roughly constant across ages.

In the middle (in gold) is a new line plot showing the same data, but by non-cumulative age ranges (five-year “age bins”). Here we see a very different pattern: risk seems to drop sharply until about age 75, then rise again afterward. However, this view is still misleading because it shows total deaths from groups of different population sizes—each older age bin contains fewer people than the younger ones.

On the right (in blue) is a line plot showing the death rate—the ratio of observed deaths to the population in each age bin. This gives a much clearer view of how risk changes with age. The data shows a small drop in risk through the 50s and 60s, followed by a steady and sharp increase starting in the early 70s. Between the late 60s and early 80s, the death rate nearly triples. This real change in risk was hidden in the earlier cumulative chart.

### The Terrible, Horrible, No Good, Very Bad Data Visualization
![Image of Bad Pie Chart](https://github.com/kthing1/DATA110-FALL2025/blob/data110-fall2025/Project1/img_bad_piechart.png)

A pie chart should be used to show how different categories make up a single whole. Unfortunately, this chart doesn’t do that. The title and pie format might suggest it shows what portion of all potentially excess deaths come from each cause of death—but that’s not true. Instead, the “slices” are based on sums of percent potentially excess deaths from multiple cumulative age ranges. Since each percentage comes from a different denominator, they can’t be added together meaningfully.

In short, this pie chart compares unrelated ratios and presents them as if they were parts of a whole. Even if the underlying data were correct, the format would still be misleading because the values don’t share the same base. This makes the chart a poor (and confusing) visualization.

## Conclusion

### Key Findings
In this analysis, I explored how the same dataset can produce very different-looking visualizations depending on how population influences the numbers. When working with data from groups of different sizes, population can heavily affect totals and mask true relationships. For many questions, it’s better to use rates or percentages instead of raw counts.

All of the visualizations I created (except the pie chart) are accurate and valid in their own contexts, but they could easily mislead viewers who don’t understand what question each one is answering. Small notes or captions aren’t always enough to fix that misunderstanding. A good data visualization should clearly communicate its message and avoid giving the wrong impression. It’s important to choose the right data, chart type, scales, and titles so the story your visualization tells matches your intent. And when reviewing others’ visualizations, always consider whether they’ve done the same.

### Further Investigation
One surprising discovery was that this dataset uses cumulative age ranges rather than distinct age bins. That approach can hide real patterns or make trends appear less significant. By separating the data into non-cumulative age ranges, I found a hidden trend—an increase in risk from unintentional injury deaths starting in the 70s. This wasn’t the dataset’s main focus, but it shows how much valuable information can be missed because of how data is structured.

Given that HHS Regions 4 and 6 (covering much of the southern U.S.) had the highest percent of potentially excess deaths from stroke, it would be useful to check whether similar regional trends exist for other causes of death. A heatmap by state, or even by metropolitan versus non-metropolitan areas, could reveal additional patterns hidden by the broader regional categories.

Finally, while this dataset identifies where excess deaths are happening, it doesn’t explain why. To move toward prevention, we’d need to explore possible contributing factors—such as median income, major industries (for example, mining and chronic lower respiratory disease), healthcare access, or Medicaid expansion. Understanding these connections could turn a descriptive dataset into one that helps uncover the causes—and solutions—for potentially excess deaths.
