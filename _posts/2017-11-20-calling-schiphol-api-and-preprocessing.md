[cleaned_dataset]: {{ site.baseurl }}/images/cleaned.png

Schiphol Airport becomes one of the few airports in the world that provide public API for developers, in this blog, I will show you how to extract data in a proper way from Schiphol Public API and present some preprocessing steps to clean the dataset.

First of all, you need to fill in a form by answering some basic questions to get api id and key, then let me show you how to extract data from Schiphol in a nice way (and I randomly chose a two-week period which is from Oct 1 to Oct 14):

```python

headers = {'ResourceVersion':'v3'}
params={'app_id':app_id,'app_key':app_key}
url = 'https://api.schiphol.nl/public-flights/flights'

day_list = list(pd.date_range('2017-10-01',periods=14,freq='D').astype(str))

l = []
for day in day_list:
    print(day)

    params['scheduledate'] = day
    r = requests.get(url=url, headers=headers, params=params)
    
    max_page = re.findall("page=(\d+)>; rel=\"last\"", r.headers['Link'])[0]

    for i in range(int(max_page)+1):
        page_params = params.copy()
        page_params['page'] = i
        r = requests.get(url=url, headers=headers, params=page_params)
        if r.text:
            data = json.loads(r.text)
            l.append(json_normalize(data['flights']))

flt = pd.concat(l)
```
Notice that in this example I add a re.findall method, that is because, due to upper limitation of throughput, we can only get a maximum of 500 pages in each call, and the size of data for each day is around 200-300 pages, therefore, our call will be terminated at a certain point. If we set the call function like above, we will make sure that each call can get a full day data! 

Once I got the data, I took a quick look and did some simple manipulations. (I didn’t show the import parts, so assuming all methods or libraries have been imported or built-in.)

```python
cols = set(flt.columns) - set(c for c in flt.columns if flt[c].isnull().all())

# replace column names
flt_analysis = flt[list(cols)].rename(columns = lambda x: x.replace('.','_')).reset_index().drop('index',axis=1)

# filter out some rows and assign a new column with airline prefix
flt_analysis = flt_analysis.query("mainFlight == flightName & serviceType == 'J'")\
    		           .drop_duplicates(subset=['scheduleDate','scheduleTime','mainFlight','flightDirection']) \
   			   .assign(prefix_mainflight = lambda x: x['mainFlight'].str.extract('^([A-Z]{3}|[A-Z0-9]{2})',expand=False))

```

In the next blog, I will present some analysis I have made about on-time rate for flights arriving at Schiphol, therefore, I need to clean the dataset a bit further to make it usable for my purpose.
```python
df_arrival_delay = flt_analysis.query("flightDirection == 'A'")\
    			       .loc[:,['scheduleDate','scheduleTime','actualLandingTime','prefix_mainflight']]\
    			       .assign(scheduleLandingTime = lambda x: pd.to_datetime(x['scheduleDate'] + ' ' + x['scheduleTime']).dt.tz_localize('CET'))\
    			       .assign(actualLandingTime = lambda x: pd.to_datetime(x['actualLandingTime']).dt.tz_localize('UTC').dt.tz_convert('CET'))\
   			       .assign(delay_min = lambda x: (x['actualLandingTime']-x['scheduleLandingTime']).astype('timedelta64[m]'))

# throw out rows where there are no actual landing time recorded
df_arrival_delay = df_arrival_delay.loc[df_arrival_delay['actualLandingTime'].notnull()]
```
You can see that only arrival flights are selected, and note that one of annoying thing in this API is the inconsistency of date time, therefore, I need to do something to align scheduleLandingTime and actualLandingTime to CET time zone.

Here is how the final dataset looks like:

![cleaned_dataset]

Okay! Now, the preprocessing part is done, and ready for the analysis part!


