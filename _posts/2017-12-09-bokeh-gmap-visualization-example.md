
[bokeh1]: {{ site.baseurl }}/images/bokeh1.png
[bokeh2]: {{ site.baseurl }}/images/bokeh2.png

Since I have shown some visualizations for demonstrating the delay condition for flights landed in Schiphol in my last blog. In this blog, I will continue to visualise the data from Schiphol Public API using bokeh library. I think most of people already know bokeh which is a Python library widely used for building advanced and modern data visualisation web applications. Here I mainly introduce a way to use GMap plot in bokeh library.

First, here is a sample of dataset with longitude and latitude of an area, we can use Polygon which is imported from shapely.geometry to plot this area:
![bokeh1]

```python
Polygon(list(zip(tmp['POINT_X'],tmp['POINT_Y'])))
```
If we have a point and would like to know if this point is within this polygon area or not, we can also easily achieve it using polygon, let’s assume the point is (4.8, 52):
```python
polygon = Polygon(list(zip(tmp['POINT_X'],tmp['POINT_Y'])))
point = Point((4.8,52))
polygon.contains(point)
```
Then the boolean result will be printed out telling you if this point (4.8,52) is located in this area or not.

Next, I will create a function that helps us find the centroid of the polygon area:
```python
# aggregate them and put all x or y coordinate in a list for each group
tmp = tmp.groupby('VOPCODE')\
                  .agg({'POINT_X': lambda x: list(x),'POINT_Y':lambda x: list(x)})\
                  .reset_index()

# define a function to get the centroid coordinate
def find_centroid(df):
    df = df.copy()
    
    for index, row in df.iterrows():
        row_poly = Polygon(list(zip(row['POINT_X'],row['POINT_Y'])))
        x,y = row_poly.exterior.centroid.xy
        df.loc[index,'centroid_x'] = x[0]
        df.loc[index,'centroid_y'] = y[0]
    
    return df

tmp = tmp.pipe(find_centroid)
```
After applying this function on the dataset, we will get two extra columns giving coordinates of the centroid. Given that we have applied this and get the centroid of all polygon areas we need in this case, then running the code below will give you a GMap plot that shows all gate locations in Schiphol terminal.
```python
map_options = GMapOptions(lat=52.3241801, lng=4.7160893, map_type="terrain", zoom=13)

p = GMapPlot(
    x_range=DataRange1d(), y_range=DataRange1d(), map_options=map_options
)
p.title.text = "Schiphol"

p.api_key = 'your_google_api_key'

source = ColumnDataSource(data = dict(lon = tmp['centroid_x'].tolist(), lat = tmp['centroid_y'].tolist(), gate = tmp['VOPCODE'].tolist()))
circle = Circle(x="lon", y="lat", size= 10, fill_color="blue", fill_alpha=0.5, line_color=None)
p.add_glyph(source, circle)

p.plot_height = 800
p.plot_width = 1200

p.add_tools(PanTool(), WheelZoomTool(), BoxSelectTool())

output_file("gmap_plot.html")
show(p)
```
The result is shown below:
![bokeh2]

In this example, we output the final bokeh figure to a local html file that is shown in a new window of browser. Alternatively, you can output it in jupyter notebook, however, probably due to the request of version of jupyter, the figure is not able to present in a perfect way in my jupyter notebook, thus I will not give example for this option.



