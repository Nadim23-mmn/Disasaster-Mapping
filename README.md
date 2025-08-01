# Disasaster-Mapping
A disaster map creation of 2015-2025 using python 
#Install the necessary library for this as following 

!pip install pandas folium

#Then run this code

import pandas as pd
import folium
from folium.plugins import MarkerCluster, MeasureControl
import matplotlib.cm as cm
import matplotlib.colors as mcolors

#Load your Excel file
df = pd.read_excel("/content/Disaster List.xlsx")

#Clean latitude and longitude values
df['Latitude'] = df['Latitude'].astype(str).str.replace('–', '-', regex=False).astype(float)
df['Longiitude'] = df['Longiitude'].astype(str).str.replace('–', '-', regex=False).astype(float)
df = df.rename(columns={'Longiitude': 'Longitude'})

#Filter out "Other" disasters if they exist
df = df[df['Type'] != 'Other']

#Count each unique disaster type
disaster_counts = df['Type'].value_counts().to_dict()

#Generate a unique color for each disaster type
disaster_types = list(disaster_counts.keys())
num_types = len(disaster_types)
colormap = cm.get_cmap('tab20', num_types)  # 20 distinct colors

disaster_colors = {}
for i, dtype in enumerate(disaster_types):
    rgba = colormap(i)
    hex_color = mcolors.to_hex(rgba)
    disaster_colors[dtype] = hex_color

#Create base map
m = folium.Map(location=[20, 90], zoom_start=5.5, tiles="CartoDB positron")
marker_cluster = MarkerCluster().add_to(m)

#Plot each disaster with a unique color
for _, row in df.iterrows():
    lat = row['Latitude']
    lon = row['Longitude']
    dtype = row['Type']
    name = row['name']
    url = row['url']
    description = row['Description']
    date = row['date_created']

    color = disaster_colors.get(dtype, 'gray')

    popup_html = f"""
    <b>{name}</b><br>
    <b>Type:</b> {dtype}<br>
    <b>Date:</b> {date}<br>
    <b>Description:</b> {description}<br>
    <a href="{url}" target="_blank">More Info</a>"""

    folium.CircleMarker(
        location=[lat, lon],
        radius=6,
        popup=folium.Popup(popup_html, max_width=300),
        color=color,
        fill=True,
        fill_color=color,
        fill_opacity=0.7
    ).add_to(marker_cluster)

#Create HTML legend
legend_items = ""
for dtype, count in disaster_counts.items():
    color = disaster_colors[dtype]
    legend_items += f'''
        <i style="background:{color}; width:10px; height:10px;
                  float:left; margin-right:6px; border-radius:50%;"></i>
        {dtype} ({count})<br>
    '''

legend_html = f'''
     <div style="
         position: fixed;
         bottom: 50px; left: 50px; width: 250px;
         background-color: white;
         border: 2px solid grey;
         z-index:9999;
         font-size:14px;
         padding: 10px;
         border-radius: 8px;
         box-shadow: 2px 2px 6px rgba(0,0,0,0.3);">
         <b>Disaster Legend</b><br>
         {legend_items}
     </div>
'''

#Add legend
m.get_root().html.add_child(folium.Element(legend_html))

#Add title
title_html = '''
     <div style="
         position: fixed;
         top: 10px; left: 50%; transform: translateX(-50%);
         font-size: 20px;
         background-color: white;
         border: 2px solid grey;
         z-index: 1000;
         padding: 10px;
         border-radius: 8px;">
         <b>Disaster Events Map (2015-2025)</b>
     </div>
'''
m.get_root().html.add_child(folium.Element(title_html))

#Add north arrow
north_arrow_html = '''
    <div style="
         position: fixed;
         top: 10px; left: 10px;
         font-size: 20px;
         background-color: white;
         z-index: 1000;
         padding: 10px;
         border-radius: 50%;
         border: 2px solid black;">
        <b>↑</b>
    </div>
'''
m.get_root().html.add_child(folium.Element(north_arrow_html))

#Add scale bar
MeasureControl(primary_length_unit='kilometers', secondary_length_unit='miles').add_to(m)

#Save the map as an HTML file
m.save("disaster_map.html")

#Display
m

# I hope it will help you! 
My pleasure to sharing knowledge.
