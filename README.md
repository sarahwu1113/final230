
"""
Class: CS230--Section XXX
Name: Your name here
Description: (Give a brief description for Exercise name--See below)
I pledge that I have completed the programming assignment independently.
I have not copied the code from a student or any source.
I have not given my code to any student.
"""

import streamlit as st
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import pydeck as pdk
import altair as alt

f = 'Boston_Crime_Date.csv'

# load in the data from file
def get_data(file):
    df = pd.read_csv(file)
    return df
f = 'Boston_Crime_Date.csv'
df = get_data(f)



#### get the line chart

df4 = pd.read_csv(f)
df4.sort_values(by=['DISTRICT'])
alist = df4["DAY_OF_WEEK"].value_counts()
xx = list(alist.index)
yy = list(alist.values.tolist())

def incidents_by_day():
    fig1, ax1 = plt.subplots()
    ax1.plot(xx,yy,marker=",", color="red")
    ax1.set_title("Number of incidents happened by days in a week")
    return fig1




### Find the hour with the most incidents

df = get_data(f)
hours_day=df[['HOUR']].values.tolist()
get_list = {k:v for (k,v) in zip((df["INCIDENT_NUMBER"]),hours_day)}
hours_in_a_day = list(range(0,24))
hour_list = []
for h in hours_in_a_day:
    total_hour = 0
    for k in get_list:
        if get_list[k][0] == h:
            total_hour += 1
    hour_list.append(total_hour)
hour_max = max(hour_list)

#### get the bar chart

alist2 = df4["HOUR"].value_counts()
xx1 = list(alist2.index)
yy1 = list(alist2.values.tolist())

def incidents_by_hours():
    fig2, ax2 = plt.subplots()
    ax2.bar(xx1,yy1,color="blue")
    ax2.set_xlabel("Hours in a day")
    ax2.set_ylabel("Number of incidents happened")
    ax2.set_title("Number of incidents happened by hours")
    return fig2


### get the histogram

alist3 = df4["DISTRICT"].value_counts()
xx2 = list(alist3.index)
yy2 = list(alist3.values.tolist())
def incidents_by_district():
    fig3, ax3 = plt.subplots(figsize = (10,7))
    ax3.bar(xx2,yy2,color="green")
    ax3.set_xlabel("Districts in Boston")
    ax3.set_ylabel("Number of incidents happened")
    ax3.set_title("Number of incidents happened by district")
    return fig3

# get the map of all the incidents

df = pd.read_csv(f)
df_map = df.copy()
df_map = df_map[['OFFENSE_DESCRIPTION','Lat','Long']]
locate = df_map[['OFFENSE_DESCRIPTION','Lat','Long']]
locate=locate[(locate.Long != 0)]
locate = locate[['Lat','Long']]

midpoint = (np.mean(locate['Lat']), np.mean(locate['Long']))
initialV = pdk.ViewState(latitude=midpoint[0],
                       longitude=midpoint[1],
                       zoom=12,
                       pitch=50)

def get_map():
    st.write(pdk.Deck(
        map_style="mapbox://styles/mapbox/light-v9",
        initial_view_state=initialV,
        layers=[pdk.Layer(
                "ScatterplotLayer",
            data=locate,
            get_position=["Long", "Lat"],
            get_radius=100,
            elevation_scale=4,
            elevation_range=[0, 1000],
            get_fill_color=[180, 0, 200, 140],
            pickable=True,
            extruded=True,)]))



### get the map of selected districts

def get_map2():
    district = df["DISTRICT"]
    districtA = district.drop_duplicates(keep='first')
    districtA= districtA.dropna()

    district_list = []
    for di in districtA:
        if di not in district_list:
            district_list.append(di)
    district_list.insert(0, " ")
    district_sb = st.sidebar.selectbox("Choose a district to further learn about:", district_list)
    finalData = df[df["DISTRICT"] == district_sb][['Lat', 'Long', 'STREET']]
    finalData.rename(columns={'Lat': 'lat', 'Long': 'lon', 'STREET': 'street'}, inplace=True)
    finalData.dropna(axis=0, how='any', inplace=True)

    map2_1 = pdk.Layer("ScatterplotLayer",
                                 finalData,
                                 stroked=True,
                                 opacity=0.8,
                                 filled=True,
                                 radius_scale=48,
                                 get_position=['lon', 'lat'],
                                 auto_highlight=True,
                                 get_radius='SHOOTING',
                                 get_fill_color=[180, 0, 200, 140],
                                 pickable=True,

                                 )


    st.write(pdk.Deck(map_style='mapbox://styles/mapbox/streets-v11',
                      initial_view_state=initialV,
                      layers=[map2_1]

                    ))



# Streamlit to show the charts

st.title("Welcome to Sarah's Final Project!")
st.header("Visualizations of Boston Crimes in 2021")
if st.button("Click here to learn about Boston's safety background"):
    st.write(" Boston was comparatively lower in 85th place overall out of 182 cities. Boston ranked lower in safety than some other major cities such as Minneapolis, Minnesota; Salt Lake City, Utah and San Diego, California...")
    link = 'Click here to read more → [link](https://www.masslive.com/news/2022/10/several-new-england-cities-rank-among-safest-in-us-for-2022-wallethub-says.html)'
    st.write(link,unsafe_allow_html=True)
    st.caption("Please make selections on the sidebar to see more details! ")

option = st.sidebar.selectbox("Please select a topic here: ",
                              ("Home",
                               "Where did all the crimes happen?",
                               "At what time do crimes always happen?",
                               "On what day do crimes always happen?"
                               ))
st.caption("Sarah Wu")
st.caption("CS230")
st.caption("Professor Masloff")


if option == "Home":
    st.snow()
    st.subheader("~"*45)
    from PIL import Image
    imageBos = Image.open("BostonPic.jpg")
    st.image(imageBos,channels='RGB', output_format='auto')
    st.write("Boston   \ncr: extraspacestorage.com  ", unsafe_allow_html=True)
if option == "Where did all the crimes happen?":
    tab1, tab2 = st.tabs(["Location of incidents on maps","Incidents by districts in graph"])
    with tab1:
        st.markdown("This map shows the locations of all the incidents ")
        get_map()
        st.markdown("To see the incidents happened in specific districts:  ")
        get_map2()
        df = get_data(f)
    with tab2:
        st.pyplot(incidents_by_district())

if option == "At what time do crimes always happen?":
    st.markdown("The bar chart shows the time of incidents happened by hours in a day")
    st.pyplot(incidents_by_hours())


    if st.checkbox("Click to reveal the hour that's mostly likely to happen a crime: "):
        st.write(f"The hour that had the most incidents in a day is 17:00 which had {hour_max} incidents happened")


if option == "On what day do crimes always happen?":
    st.markdown("The line chart shows the time of incidents happened by days of week")
    st.pyplot(incidents_by_day())
