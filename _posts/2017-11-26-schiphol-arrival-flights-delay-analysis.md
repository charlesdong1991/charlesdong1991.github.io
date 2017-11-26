[analysis1]: {{ site.baseurl }}/images/analysis1.png
[analysis2]: {{ site.baseurl }}/images/analysis2.png
[analysis3]: {{ site.baseurl }}/images/analysis3.png
[analysis4]: {{ site.baseurl }}/images/analysis4.png

In the last blog, I have presented how to extract data from Schiphol Public Flight API, and made a basic preprocessing on the arrival flights, thus in this blog, I am going to have an analysis on on-time rate of flights to Schiphol Airport by using the dataset I have parsed.

First of all, let’s take a look at the scheduled arriving time of flights in 15 min interval agains the actual arrival time. 

```python
# round the scheduled time and actual arrival time to 15min interval
df_arrival_delay_15min  = df_arrival_delay.assign(schedule_rounded_15min = lambda x: x['scheduleLandingTime'].dt.round('15min'))\
                                                                    .assign(actual_rounded_15min = lambda x: x['actualLandingTime'].dt.round('15min'))  

def merge_to_15min(df, col):
    """
    df: the original dataset (in this project, that is df_arrival_delay_15min)
    col: the column needs to be grouped by

    return: the merged dataset with all time ranges in every 15min and its corresponding amount of flights
    """
    df_timerange_backup = pd.DataFrame(pd.date_range(start='00:00:00',end='23:45:00',freq='15min'))\
                            .rename(columns={0:'h_m_s'})\
                            .assign(h_m_s = lambda x: x['h_m_s'].dt.strftime("%H:%M:%S"))
            
    df_parsed = (df.assign(h_m_s = lambda x: x[col].dt.strftime("%H:%M:%S"))
                    .groupby("h_m_s")
                    .size()
                    .to_frame()
                    .reset_index()
                    .rename(columns={0:'frequency'})
                )
    df_merged = pd.merge(df_timerange_backup, df_parsed, on='h_m_s',how='left').fillna(0)
    
    return df_merged

df_actual_15min = merge_to_15min(df_arrival_delay_15min,'actual_rounded_15min')
df_schedule_15min = merge_to_15min(df_arrival_delay_15min, 'schedule_rounded_15min')

# Concatenate two sets to one that can be used for plotting
df_analysis1 = pd.concat([df_actual_15min.assign(indicator = lambda x: 'Actual'),df_schedule_15min.assign(indicator = lambda x: 'Schedule')])

sns.factorplot(x='h_m_s',y='frequency',hue='indicator',data=df_analysis1, size = 12, kind='bar',palette="muted")
```

Then, we can get a plot which shows how much off the actual arrival time against the scheduled time, as shown in the figure, seems on average, flights to Schiphol has quite good punctuality, since rounding to 15min is a quite high standard for flights:
![analysis1]

Secondly, let’s only take a look at the 15 airlines that have the most arrival flights to Schiphol, and see the distribution of the number of flights in hourly interval.

```python

def stacked_bar_plot(df):
    df = df.copy()
    flight_top_list = list(df.prefix_mainflight.value_counts().head(15).index)
    _ = df.loc[df['prefix_mainflight'].isin(flight_top_list)]\
                .assign(hour = lambda x: x['actualLandingTime'].dt.hour)\
                .groupby(['hour','prefix_mainflight'])\
                .size()\
                .to_frame('frequency')\
                .reset_index()\
                .pivot_table(index='hour',columns=['prefix_mainflight'])['frequency']\
                .plot(kind='bar',stacked=True,figsize=(20,8),label='_nolegend_')

# plot the distribution of the number of flights in hourly level.
df_arrival_delay.pipe(stacked_bar_plot)
```
The result is shown below:
![analysis2]

We can learn that in the early morning and late night, HV (Transavia) flights fit in most of the slots, and the rest of time, KLM is dominant which is easy to imagine because Schiphol is KLM’s hub. And then, let’s give an attention to the hourly averaging delay of flights in Schiphol.

```python
def delay_boxplot(df):
    df = df.copy()
    df = df.assign(hour = lambda x: x['actualLandingTime'].dt.hour)
    
    fig, ax = plt.subplots(1,1, figsize=(16,10))
    
    _ = sns.boxplot(x="hour",y="delay_min",data=df)
    _ = ax.axhline(y = 0, color='r')

df_arrival_delay.pipe(delay_boxplot)
```
![analysis3]

As it’s shown in the figure above, we can conclude that the punctuality of arrival flights to Schiphol looks very good, although there are some outliers, the averaging results are still awesome. Of course, we should not make such conclusion so early because we need also to take weather or other conditions that may influence delays into account. Since there is no such data in this public API, so we can firstly take a glimpse of hourly delay level in these 14 days separately.

```python
def delay_heatmap(df):
    df = df.copy()
    
    df = df.assign(hour = lambda x: x['actualLandingTime'].dt.hour)\
           .groupby(['scheduleDate','hour'])\
           .agg({'delay_min':'mean'})['delay_min']\
           .unstack(0)
        
    fig, ax = plt.subplots(figsize= (16,12))
    _ = sns.heatmap(df)

df_arrival_delay.pipe(delay_heatmap)
```

The heatmap of delays in hourly level in these days is plotted below, the colour bar indicates the delay level. The darker the bar is, the shorter averaging delay minutes the flights have in the cube. The completely white cubes show that there is no fight in that hourly slot. And we can directly see most of time the cubes are dark which reflect the good on-time rate of flights to Schiphol.

![analysis4]

Well, the above four plots and some analysis on on-time rate of arrival flights in Schiphol in a collected 14-day dataset from Schiphol’s public API have been done, and I can more or less conclude that Schiphol has a quite good arrival flight on-time rate although the sample is very small.

