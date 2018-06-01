
****Analysis****
1. The temperature is hotter when the city is close to the equater 
2. There is no strong correlation between Humidity, clodness, wind speed and latitudes
3. In most cities the wind speed is less than 8 m/s. 


```python
%matplotlib inline

# Dependencies
#import openweathermapy.core as owm
import pandas as pd
import warnings
import matplotlib.pyplot as plt
import numpy as np
import requests
import csv

from config import api_key
from citipy import citipy
from geopy.geocoders import Nominatim
```

**Randomly pick 500 cities and get their coordinates by geolocator**
1. use set() to keep unique cities
2. if geolocator can't find the respective coordinates, I remove the city


```python
#select 500 unique cities based on latitude and longitude => random generating coordinates, and keep unique ones
unique_city_list = set()
city_list=[]
country_list=[]

lat_list=[]
long_list=[]

geolocator = Nominatim()

while len(unique_city_list) < 500:
    lat = np.random.uniform(-90, 90, 1)
    long = np.random.uniform(-180, 180, 1)
    city = citipy.nearest_city(lat, long)
    if city.city_name not in unique_city_list:
        unique_city_list.add(city.city_name)
        city_list.append(city.city_name)
        country_list.append(city.country_code)
        city_country=f"{city.city_name}, {city.country_code}"
        try:
            location = geolocator.geocode(city_country)
            lat_list.append(location.latitude)                 
            long_list.append(location.longitude)
        
        except Exception as e: 
            unique_city_list.pop()
            city_list.pop()
            country_list.pop()
            print(city_country,":",e)
        
    
```

    rikitea, pf : 'NoneType' object has no attribute 'latitude'
    atuona, pf : 'NoneType' object has no attribute 'latitude'
    poum, nc : 'NoneType' object has no attribute 'latitude'
    longyearbyen, sj : 'NoneType' object has no attribute 'latitude'
    taolanaro, mg : 'NoneType' object has no attribute 'latitude'
    barentsburg, sj : 'NoneType' object has no attribute 'latitude'
    avera, pf : 'NoneType' object has no attribute 'latitude'
    cherskiy, ru : 'NoneType' object has no attribute 'latitude'
    tabiauea, ki : 'NoneType' object has no attribute 'latitude'
    saint-philippe, re : 'NoneType' object has no attribute 'latitude'
    karauzyak, uz : 'NoneType' object has no attribute 'latitude'
    pereslavl-zalesskiy, ru : 'NoneType' object has no attribute 'latitude'
    sinnamary, gf : 'NoneType' object has no attribute 'latitude'
    sentyabrskiy, ru : 'NoneType' object has no attribute 'latitude'
    longyearbyen, sj : 'NoneType' object has no attribute 'latitude'
    asau, tv : 'NoneType' object has no attribute 'latitude'
    faanui, pf : 'NoneType' object has no attribute 'latitude'
    moerai, pf : 'NoneType' object has no attribute 'latitude'
    velizh, ru : 'NoneType' object has no attribute 'latitude'
    kegayli, uz : 'NoneType' object has no attribute 'latitude'
    egvekinot, ru : 'NoneType' object has no attribute 'latitude'
    shaartuz, tj : 'NoneType' object has no attribute 'latitude'
    kuybysheve, ua : 'NoneType' object has no attribute 'latitude'
    blagoyevo, ru : 'NoneType' object has no attribute 'latitude'
    san pedro de ycuamandiyu, py : 'NoneType' object has no attribute 'latitude'
    tsumeb, na : Service timed out
    lolua, tv : 'NoneType' object has no attribute 'latitude'
    naze, jp : 'NoneType' object has no attribute 'latitude'
    faanui, pf : 'NoneType' object has no attribute 'latitude'
    andevoranto, mg : 'NoneType' object has no attribute 'latitude'
    grafton, au : Service timed out
    


```python
# This part is to validate geopy and citipy

# city_list=set()
# lat_list=[]
# long_list=[]
# random_lat=[]
# random_long=[]

# while len(city_list) < 10:
#     lat = np.random.uniform(-90, 90, 1)
#     long = np.random.uniform(-180, 180, 1)
#     city = citipy.nearest_city(lat, long)
#     if city.city_name not in city_list:
#         city_list.add(city.city_name)
#         country_list.append(city.country_code)
#         city_country=f"{city.city_name}, {city.country_code}"
#         random_lat.append(lat)
#         random_long.append(long)
        
#         try:
#             location = geolocator.geocode(city_country)
#             lat_list.append(location.latitude)                 
#             long_list.append(location.longitude)
        
#         except Exception as e: 
#             city_list.pop()
#             random_lat.pop()
#             random_long.pop()
#             print(city_country,":",e)

# check_df=pd.DataFrame(columns=['random_lat','random_long','lat_list','long_list','city_name'])
# check_df.lat_list=lat_list
# check_df.long_list=long_list
# check_df.random_lat=random_lat
# check_df.random_long=random_long
# check_df.city_name=city_list
        
    
    
```

**Use api to check weather and print city number, city name, and requested URL**
1. Mainly calling by city
2. If owm cant find the city, calling by coords -> Owm can always get weather by coords


```python
#create a database including Temperature, Humidity, Cloudiness, Wind Speed
city_df=pd.DataFrame(columns=["city_name","country","temperature","humidity","cloudness","wind_speed","lat","long","city_name_owm"])
city_df.city_name=city_list
city_df.country=country_list
city_df.lat=lat_list
city_df.long=long_list

```


```python

url = 'http://api.openweathermap.org/data/2.5/weather?'

for index, row in city_df.iterrows():
    lon_setting = row['long']
    lat_setting = row['lat']
    params1 = {"units": "metric", "APPID": api_key, "q":row['city_name']}
    params2 = {"units": "metric", "APPID": api_key, "lon":lon_setting, "lat":lat_setting}
    #print(settings)

    
    try:
        response1 = requests.get(url, params=params1)
        current_weather = response1.json()  
        temp = current_weather['main']['temp']
        humidity = current_weather["main"]['humidity']
        cloud = current_weather["clouds"]["all"]
        wind = current_weather["wind"]["speed"]
        city_name_owm=current_weather['name']

        city_df.at[index,'temperature'] = temp
        city_df.at[index,'humidity'] = humidity
        city_df.at[index,'cloudness'] = cloud
        city_df.at[index,'wind_speed'] = wind
        city_df.at[index,'city_name_owm'] = city_name_owm
        
        print(f"{index} : The weather in {current_weather['name']} is {current_weather['weather'][0]['description']}")
        print(response1.url)
    
    except Exception as e:
        response2 = requests.get(url, params=params2)
        current_weather = response2.json()  
        temp = current_weather["main"]["temp"]
        humidity = current_weather["main"]["humidity"]
        cloud = current_weather["clouds"]["all"]
        wind = current_weather["wind"]["speed"]
        city_name_owm = current_weather['name']

        city_df.at[index,'temperature'] = temp
        city_df.at[index,'humidity'] = humidity
        city_df.at[index,'cloudness'] = cloud
        city_df.at[index,'wind_speed'] = wind
        city_df.at[index,'city_name_owm'] = city_name_owm
        
        print(f"{index} : The weather at {current_weather['coord']} is {current_weather['weather'][0]['description']}")
        print(response2.url)
    

```

    0 : The weather in Tessalit is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=tessalit
    1 : The weather in Punta Arenas is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=punta+arenas
    2 : The weather in Busselton is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=busselton
    3 : The weather in Mataura is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=mataura
    4 : The weather in Bluff is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bluff
    5 : The weather in San Rafael is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=san+rafael
    6 : The weather in Katobu is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=katobu
    7 : The weather in New Norfolk is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=new+norfolk
    8 : The weather in Gornopravdinsk is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=gornopravdinsk
    9 : The weather in Hithadhoo is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=hithadhoo
    10 : The weather in East London is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=east+london
    11 : The weather in Pisco is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=pisco
    12 : The weather in Albany is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=albany
    13 : The weather in Hilo is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=hilo
    14 : The weather in Sharjah is mist
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=sharjah
    15 : The weather in Port Lincoln is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=port+lincoln
    16 : The weather in Sisimiut is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=sisimiut
    17 : The weather in Saint-Pierre is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=saint-pierre
    18 : The weather in Masuda is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=masuda
    19 : The weather in Kondinskoye is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kondinskoye
    20 : The weather in Havre-Saint-Pierre is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=havre-saint-pierre
    21 : The weather in Provideniya is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=provideniya
    22 : The weather in Mataura is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=mataura
    23 : The weather in Ushuaia is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ushuaia
    24 : The weather in Pangnirtung is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=pangnirtung
    25 : The weather in Churapcha is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=churapcha
    26 : The weather in Ambulu is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ambulu
    27 : The weather in Avarua is light intensity drizzle
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=avarua
    28 : The weather in Raymondville is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=raymondville
    29 : The weather in Jamestown is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=jamestown
    30 : The weather in Staroshcherbinovskaya is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=staroshcherbinovskaya
    31 : The weather in Hasaki is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=hasaki
    32 : The weather in Georgetown is light intensity shower rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=georgetown
    33 : The weather in Bileca is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bileca
    34 : The weather in Champerico is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=champerico
    35 : The weather at {'lon': -171.43, 'lat': -14} is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=-171.4287032&lat=-13.9973264
    36 : The weather in Port Alfred is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=port+alfred
    37 : The weather in Kavieng is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kavieng
    38 : The weather in Batticaloa is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=batticaloa
    39 : The weather in Biak is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=biak
    40 : The weather in Hithadhoo is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=hithadhoo
    41 : The weather in Bredasdorp is moderate rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bredasdorp
    42 : The weather in Kodiak is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kodiak
    43 : The weather in Fukue is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=fukue
    44 : The weather in Cabo San Lucas is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=cabo+san+lucas
    45 : The weather in Onguday is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=onguday
    46 : The weather in Graham is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=graham
    47 : The weather in Skjervoy is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=skjervoy
    48 : The weather in Kaitangata is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kaitangata
    49 : The weather in Coquimbo is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=coquimbo
    50 : The weather in Wellington is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=wellington
    51 : The weather at {'lon': 174.76, 'lat': -1.21} is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=174.7571277&lat=-1.2130376
    52 : The weather in Upernavik is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=upernavik
    53 : The weather in Shakiso is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=shakiso
    54 : The weather in Esperance is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=esperance
    55 : The weather in Livingston is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=livingston
    56 : The weather in Aswan is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=aswan
    57 : The weather in Deputatskiy is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=deputatskiy
    58 : The weather in Adrar is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=adrar
    59 : The weather in Saint George is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=saint+george
    60 : The weather in Port Elizabeth is moderate rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=port+elizabeth
    61 : The weather in Burnie is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=burnie
    62 : The weather at {'lon': 52.34, 'lat': 71.54} is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=52.337048&lat=71.5386769
    63 : The weather at {'lon': 86.17, 'lat': 41.72} is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=86.173541&lat=41.7238743
    64 : The weather in Lesnoy is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=lesnoy
    65 : The weather at {'lon': 82.93, 'lat': 41.71} is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=82.9308106&lat=41.7136343
    66 : The weather in Tuktoyaktuk is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=tuktoyaktuk
    67 : The weather in Rawson is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=rawson
    68 : The weather at {'lon': -23.71, 'lat': 64.9} is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=-23.7084345&lat=64.8959304
    69 : The weather in Inhambane is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=inhambane
    70 : The weather in Ostrovnoy is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ostrovnoy
    71 : The weather in Sedalia is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=sedalia
    72 : The weather in Kangaba is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kangaba
    73 : The weather in Mar del Plata is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=mar+del+plata
    74 : The weather at {'lon': 136.13, 'lat': 71.44} is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=136.130646&lat=71.440445
    75 : The weather in Kaka is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kaka
    76 : The weather in Onokhoy is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=onokhoy
    77 : The weather in Solnechnyy is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=solnechnyy
    78 : The weather in Hobart is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=hobart
    79 : The weather in Saint-Joseph is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=saint-joseph
    80 : The weather in Klaksvik is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=klaksvik
    81 : The weather at {'lon': -75.08, 'lat': -15.22} is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=-75.0830344456747&lat=-15.2155331
    82 : The weather in La Palma is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=la+palma
    83 : The weather in Catabola is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=catabola
    84 : The weather in Arraial do Cabo is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=arraial+do+cabo
    85 : The weather in Kungurtug is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kungurtug
    86 : The weather in Leh is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=leh
    87 : The weather in Torbay is light intensity drizzle
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=torbay
    88 : The weather in Gairo is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=gairo
    89 : The weather in Tura is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=tura
    90 : The weather in Bukama is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bukama
    91 : The weather in Gallup is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=gallup
    92 : The weather in Rio Gallegos is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=rio+gallegos
    93 : The weather in Ribeira Grande is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ribeira+grande
    94 : The weather at {'lon': -23.25, 'lat': 66.16} is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=-23.2507273&lat=66.1575503
    95 : The weather in Petropavlovsk-Kamchatskiy is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=petropavlovsk-kamchatskiy
    96 : The weather at {'lon': -172.33, 'lat': -13.45} is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=-172.3318889&lat=-13.4518478
    97 : The weather at {'lon': 61.56, 'lat': 69.76} is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=61.5600439&lat=69.7634883
    98 : The weather in Khorol is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=khorol
    99 : The weather in Minsk is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=minsk
    100 : The weather in Qaanaaq is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=qaanaaq
    101 : The weather in Meulaboh is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=meulaboh
    102 : The weather in Souillac is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=souillac
    103 : The weather in North Bend is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=north+bend
    104 : The weather in Buchanan is moderate rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=buchanan
    105 : The weather in Rocha is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=rocha
    106 : The weather in Candelaria is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=candelaria
    107 : The weather in Lagoa is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=lagoa
    108 : The weather in Khatanga is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=khatanga
    109 : The weather in Smithers is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=smithers
    110 : The weather in Torbat-e Jam is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=torbat-e+jam
    111 : The weather in Victoria is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=victoria
    112 : The weather in Dukat is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=dukat
    113 : The weather in Tateyama is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=tateyama
    114 : The weather in Barrow is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=barrow
    115 : The weather in Nemuro is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=nemuro
    116 : The weather in La Ronge is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=la+ronge
    117 : The weather in Saint-Pierre is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=saint-pierre
    118 : The weather at {'lon': 6.78, 'lat': 31.23} is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=6.77751844903644&lat=31.2270384
    119 : The weather in Cape Town is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=cape+town
    120 : The weather in Leua is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=leua
    121 : The weather at {'lon': 40.25, 'lat': 29.92} is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=40.252222&lat=29.919444
    122 : The weather in Dwarka is haze
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=dwarka
    123 : The weather in De Aar is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=de+aar
    124 : The weather at {'lon': -21.97, 'lat': 70.49} is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=-21.9654126308067&lat=70.4857907
    125 : The weather at {'lon': 31.33, 'lat': 30.1} is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=31.3298196&lat=30.1002261
    126 : The weather in Quimper is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=quimper
    127 : The weather in Stokmarknes is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=stokmarknes
    128 : The weather in Mogadishu is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=mogadishu
    129 : The weather in Vaini is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=vaini
    130 : The weather in Auchi is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=auchi
    131 : The weather in Clyde is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=clyde
    132 : The weather in Skorodnoye is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=skorodnoye
    133 : The weather in Pocone is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=pocone
    134 : The weather in Pecos is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=pecos
    135 : The weather in Tuatapere is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=tuatapere
    136 : The weather in Hermanus is moderate rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=hermanus
    137 : The weather in Isangel is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=isangel
    138 : The weather in Tessalit is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=tessalit
    139 : The weather in Sao Felix do Xingu is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=sao+felix+do+xingu
    140 : The weather in Andros Town is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=andros+town
    141 : The weather in Necochea is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=necochea
    142 : The weather in Santa Fe is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=santa+fe
    143 : The weather in Yellowknife is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=yellowknife
    144 : The weather in Sanquhar is mist
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=sanquhar
    145 : The weather in Huntsville is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=huntsville
    146 : The weather in Boa Vista is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=boa+vista
    147 : The weather in Markova is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=markova
    148 : The weather in Monticello is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=monticello
    149 : The weather at {'lon': -179.41, 'lat': 68.89} is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=-179.4130958&lat=68.8928813
    150 : The weather in Ambon is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ambon
    151 : The weather in Grindavik is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=grindavik
    152 : The weather in Alyangula is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=alyangula
    153 : The weather in Talmenka is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=talmenka
    154 : The weather in Tiksi is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=tiksi
    155 : The weather in Touros is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=touros
    156 : The weather in Bethel is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bethel
    157 : The weather in Kapaa is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kapaa
    158 : The weather in Fairbanks is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=fairbanks
    159 : The weather in Bredasdorp is moderate rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bredasdorp
    160 : The weather in Leningradskiy is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=leningradskiy
    161 : The weather in Peace River is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=peace+river
    162 : The weather in Nelson Bay is shower rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=nelson+bay
    163 : The weather in Lianzhou is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=lianzhou
    164 : The weather in Luderitz is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=luderitz
    165 : The weather in Saltillo is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=saltillo
    166 : The weather in Pevek is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=pevek
    167 : The weather in Nikolskoye is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=nikolskoye
    168 : The weather in Sioux Lookout is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=sioux+lookout
    169 : The weather in Srandakan is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=srandakan
    170 : The weather at {'lon': 25.23, 'lat': 56.02} is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=25.2265171&lat=56.0227453
    171 : The weather in Suntar is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=suntar
    172 : The weather in Padang is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=padang
    173 : The weather in Evensk is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=evensk
    174 : The weather at {'lon': 39.85, 'lat': 39.67} is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=39.8506602454707&lat=39.6704427
    175 : The weather at {'lon': -86.83, 'lat': 21.13} is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=-86.8340163&lat=21.1330071
    176 : The weather in Lebu is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=lebu
    177 : The weather in San Patricio is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=san+patricio
    178 : The weather in Margate is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=margate
    179 : The weather in Kahului is shower rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kahului
    180 : The weather in Shache is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=shache
    181 : The weather in Elko is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=elko
    182 : The weather in Ligayan is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ligayan
    183 : The weather in Tucuman is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=tucuman
    184 : The weather in Pacific Grove is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=pacific+grove
    185 : The weather in Aklavik is light intensity shower rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=aklavik
    186 : The weather in Lorengau is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=lorengau
    187 : The weather in Shubarkuduk is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=shubarkuduk
    188 : The weather in Chuy is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=chuy
    189 : The weather in Severo-Kurilsk is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=severo-kurilsk
    190 : The weather in Hualmay is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=hualmay
    191 : The weather in Ponta do Sol is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ponta+do+sol
    192 : The weather in Natal is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=natal
    193 : The weather in Coihaique is mist
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=coihaique
    194 : The weather in Butaritari is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=butaritari
    195 : The weather in Kidal is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kidal
    196 : The weather in Bud is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bud
    197 : The weather at {'lon': 41.84, 'lat': -1.24} is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=41.843358&lat=-1.236385
    198 : The weather in Vila Velha is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=vila+velha
    199 : The weather in Lompoc is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=lompoc
    200 : The weather in Guerrero Negro is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=guerrero+negro
    201 : The weather in Husavik is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=husavik
    202 : The weather in Ketchikan is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ketchikan
    203 : The weather in Yumen is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=yumen
    204 : The weather in Castro is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=castro
    205 : The weather in Kholmogory is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kholmogory
    206 : The weather in Tibati is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=tibati
    207 : The weather in Abu Kamal is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=abu+kamal
    208 : The weather in Provideniya is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=provideniya
    209 : The weather in Iqaluit is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=iqaluit
    210 : The weather in Mumford is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=mumford
    211 : The weather in Geilo is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=geilo
    212 : The weather in Labuhan is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=labuhan
    213 : The weather in Olinda is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=olinda
    214 : The weather in Omagh is mist
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=omagh
    215 : The weather in Kaa-Khem is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kaa-khem
    216 : The weather in Saint-Augustin is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=saint-augustin
    217 : The weather in Monrovia is moderate rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=monrovia
    218 : The weather in Bella Vista is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bella+vista
    219 : The weather in Alexandria is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=alexandria
    220 : The weather in Miandrivazo is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=miandrivazo
    221 : The weather in Talnakh is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=talnakh
    222 : The weather in Bambous Virieux is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bambous+virieux
    223 : The weather in Monchegorsk is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=monchegorsk
    224 : The weather at {'lon': 178.32, 'lat': -37.89} is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=178.3187514&lat=-37.8919955
    225 : The weather at {'lon': 28.92, 'lat': -30.82} is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=28.9217761780498&lat=-30.82491
    226 : The weather in Broome is fog
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=broome
    227 : The weather in Dikson is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=dikson
    228 : The weather in Puerto Escondido is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=puerto+escondido
    229 : The weather in Vilyuysk is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=vilyuysk
    230 : The weather in Kabalo is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kabalo
    231 : The weather in Hofn is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=hofn
    232 : The weather in Nongstoin is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=nongstoin
    233 : The weather in Messina is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=messina
    234 : The weather in Lasa is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=lasa
    235 : The weather in Shahrud is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=shahrud
    236 : The weather in Danville is mist
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=danville
    237 : The weather in Puerto Ayora is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=puerto+ayora
    238 : The weather in Chokurdakh is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=chokurdakh
    239 : The weather at {'lon': 30.56, 'lat': 36.55} is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=30.5617305&lat=36.5506873
    240 : The weather in Koumac is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=koumac
    241 : The weather in Feijo is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=feijo
    242 : The weather at {'lon': 109.01, 'lat': 11.57} is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=109.0089526&lat=11.5690854
    243 : The weather in Airai is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=airai
    244 : The weather in Imbituba is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=imbituba
    245 : The weather at {'lon': 140.88, 'lat': 38.25} is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=140.8838453&lat=38.2462589
    246 : The weather in Nosy Varika is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=nosy+varika
    247 : The weather in Bowen is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bowen
    248 : The weather in Solton is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=solton
    249 : The weather in Colwyn Bay is mist
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=colwyn+bay
    250 : The weather in Dingle is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=dingle
    251 : The weather in Shimoda is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=shimoda
    252 : The weather in Dawlatabad is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=dawlatabad
    253 : The weather in Norman Wells is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=norman+wells
    254 : The weather in Sitka is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=sitka
    255 : The weather in Aksarka is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=aksarka
    256 : The weather in Clyde River is light snow
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=clyde+river
    257 : The weather in Presidencia Roque Saenz Pena is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=presidencia+roque+saenz+pena
    258 : The weather in Menongue is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=menongue
    259 : The weather in Clifton is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=clifton
    260 : The weather in Port-de-Bouc is mist
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=port-de-bouc
    261 : The weather in Ponta Grossa is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ponta+grossa
    262 : The weather in Chara is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=chara
    263 : The weather at {'lon': 32.5, 'lat': 34.92} is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=32.4982122742881&lat=34.9169592
    264 : The weather in Ixtapa is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ixtapa
    265 : The weather in Mayo is light intensity shower rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=mayo
    266 : The weather in Puerto Leguizamo is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=puerto+leguizamo
    267 : The weather in Cidreira is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=cidreira
    268 : The weather in Bassar is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bassar
    269 : The weather in West Wendover is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=west+wendover
    270 : The weather in Colares is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=colares
    271 : The weather in Port-Cartier is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=port-cartier
    272 : The weather in Sibu is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=sibu
    273 : The weather in Mangrol is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=mangrol
    274 : The weather in Inuvik is light intensity shower rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=inuvik
    275 : The weather in Alekseyevsk is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=alekseyevsk
    276 : The weather in Atar is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=atar
    277 : The weather in Nizhnyaya Tavda is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=nizhnyaya+tavda
    278 : The weather in Los Llanos de Aridane is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=los+llanos+de+aridane
    279 : The weather in Vincennes is thunderstorm with heavy rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=vincennes
    280 : The weather in Termoli is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=termoli
    281 : The weather in Marathon is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=marathon
    282 : The weather in Lalmohan is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=lalmohan
    283 : The weather in Sorland is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=sorland
    284 : The weather in Vidim is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=vidim
    285 : The weather in Iquique is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=iquique
    286 : The weather in Carnarvon is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=carnarvon
    287 : The weather in Huilong is moderate rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=huilong
    288 : The weather in Zhangye is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=zhangye
    289 : The weather in Gorontalo is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=gorontalo
    290 : The weather in Kamennomostskoye is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kamennomostskoye
    291 : The weather in Mount Isa is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=mount+isa
    292 : The weather in Diffa is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=diffa
    293 : The weather in Kruisfontein is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kruisfontein
    294 : The weather in Xuddur is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=xuddur
    295 : The weather in Celestun is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=celestun
    296 : The weather in Atambua is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=atambua
    297 : The weather in Sandpoint is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=sandpoint
    298 : The weather in Komsomolskiy is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=komsomolskiy
    299 : The weather in San Quintin is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=san+quintin
    300 : The weather in Kapit is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kapit
    301 : The weather in Dno is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=dno
    302 : The weather in Qinzhou is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=qinzhou
    303 : The weather in Ardakan is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ardakan
    304 : The weather in Praia da Vitoria is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=praia+da+vitoria
    305 : The weather in Yeniseysk is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=yeniseysk
    306 : The weather in Waingapu is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=waingapu
    307 : The weather in Calella is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=calella
    308 : The weather in Pitimbu is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=pitimbu
    309 : The weather in Kapuvar is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kapuvar
    310 : The weather in Namatanai is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=namatanai
    311 : The weather in Makokou is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=makokou
    312 : The weather in Arlit is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=arlit
    313 : The weather in Ilulissat is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ilulissat
    314 : The weather in Neiafu is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=neiafu
    315 : The weather in Mamallapuram is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=mamallapuram
    316 : The weather in Vila is fog
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=vila
    317 : The weather in Nanortalik is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=nanortalik
    318 : The weather in Ilhabela is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ilhabela
    319 : The weather in Umm Lajj is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=umm+lajj
    320 : The weather in Sosnogorsk is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=sosnogorsk
    321 : The weather in Dakar is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=dakar
    322 : The weather in Poyarkovo is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=poyarkovo
    323 : The weather in Kaitangata is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kaitangata
    324 : The weather in Xai-Xai is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=xai-xai
    325 : The weather in Delta del Tigre is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=delta+del+tigre
    326 : The weather in Puerto Ayora is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=puerto+ayora
    327 : The weather in Riyadh is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=riyadh
    328 : The weather in Conde is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=conde
    329 : The weather in Vila Franca do Campo is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=vila+franca+do+campo
    330 : The weather in Portland is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=portland
    331 : The weather at {'lon': 74.36, 'lat': 46.63} is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=74.359215&lat=46.630707
    332 : The weather in Pimentel is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=pimentel
    333 : The weather at {'lon': 45.06, 'lat': 9.41} is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=45.0643362&lat=9.4082429
    334 : The weather in Byron Bay is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=byron+bay
    335 : The weather in Trelew is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=trelew
    336 : The weather in Bethal is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bethal
    337 : The weather in Punta Alta is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=punta+alta
    338 : The weather in San Juan is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=san+juan
    339 : The weather in Duma is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=duma
    340 : The weather in Bossangoa is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bossangoa
    341 : The weather at {'lon': 45.75, 'lat': 35.33} is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=45.7519039&lat=35.3324228
    342 : The weather in Hun is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=hun
    343 : The weather in Codrington is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=codrington
    344 : The weather in Guangyuan is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=guangyuan
    345 : The weather in Cayenne is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=cayenne
    346 : The weather in Bhag is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bhag
    347 : The weather in Lavrentiya is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=lavrentiya
    348 : The weather in Nizwa is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=nizwa
    349 : The weather in San Vicente is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=san+vicente
    350 : The weather in Alofi is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=alofi
    351 : The weather at {'lon': 82.46, 'lat': 65.7} is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=82.458855&lat=65.704239
    352 : The weather in Katherine is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=katherine
    353 : The weather in Codo is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=codo
    354 : The weather in Yaan is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=yaan
    355 : The weather in Cotonou is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=cotonou
    356 : The weather in DAULTALA is haze
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=daultala
    357 : The weather in Trincomalee is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=trincomalee
    358 : The weather in Kaeo is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kaeo
    359 : The weather in Cooma is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=cooma
    360 : The weather in Tarata is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=tarata
    361 : The weather in Todos Santos is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=todos+santos
    362 : The weather at {'lon': 1.85, 'lat': 47.9} is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=1.8546041&lat=47.8952522
    363 : The weather in Dombarovskiy is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=dombarovskiy
    364 : The weather in Yangcun is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=yangcun
    365 : The weather in Marsh Harbour is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=marsh+harbour
    366 : The weather in Lucapa is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=lucapa
    367 : The weather in Salalah is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=salalah
    368 : The weather in Oskarshamn is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=oskarshamn
    369 : The weather in Inverness is fog
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=inverness
    370 : The weather in Beaufort is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=beaufort
    371 : The weather in Seoul is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=seoul
    372 : The weather in Estelle is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=estelle
    373 : The weather in San Policarpo is moderate rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=san+policarpo
    374 : The weather in Stephenville is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=stephenville
    375 : The weather in Comodoro Rivadavia is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=comodoro+rivadavia
    376 : The weather in Paracuru is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=paracuru
    377 : The weather in College is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=college
    378 : The weather in Krapkowice is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=krapkowice
    379 : The weather at {'lon': 73.55, 'lat': 3.47} is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=73.5474882370166&lat=3.4720121
    380 : The weather in Strezhevoy is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=strezhevoy
    381 : The weather in Kardailovo is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kardailovo
    382 : The weather at {'lon': -84.49, 'lat': 38.07} is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=-84.4859523&lat=38.0722901
    383 : The weather in Lakes Entrance is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=lakes+entrance
    384 : The weather at {'lon': 71.58, 'lat': 36.71} is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=71.5767799&lat=36.7090116
    385 : The weather at {'lon': 102.54, 'lat': -3.52} is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=102.5359834&lat=-3.5186763
    386 : The weather in Trindade is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=trindade
    387 : The weather in Zeya is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=zeya
    388 : The weather in Saskylakh is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=saskylakh
    389 : The weather in Flinders is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=flinders
    390 : The weather in Ungaran is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ungaran
    391 : The weather in Kavaratti is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kavaratti
    392 : The weather in Monroe is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=monroe
    393 : The weather in Santiago de Cao is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=santiago+de+cao
    394 : The weather in Palmer is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=palmer
    395 : The weather in Salinas is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=salinas
    396 : The weather at {'lon': 33.83, 'lat': 27.24} is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=33.8267842&lat=27.2405016
    397 : The weather at {'lon': 35.71, 'lat': 65.02} is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=35.7103677&lat=65.0221405
    398 : The weather in Boke is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=boke
    399 : The weather in Constitucion is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=constitucion
    400 : The weather in Mantenopolis is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=mantenopolis
    401 : The weather in Sarangani is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=sarangani
    402 : The weather in Nikko is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=nikko
    403 : The weather in Mahebourg is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=mahebourg
    404 : The weather in Zarasai is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=zarasai
    405 : The weather in Nicoya is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=nicoya
    406 : The weather in Derzhavinsk is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=derzhavinsk
    407 : The weather in Srednekolymsk is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=srednekolymsk
    408 : The weather in Saldanha is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=saldanha
    409 : The weather in Gat is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=gat
    410 : The weather in Baoding is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=baoding
    411 : The weather in Allanridge is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=allanridge
    412 : The weather in Tambovka is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=tambovka
    413 : The weather in Carballo is fog
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=carballo
    414 : The weather in Troitskoye is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=troitskoye
    415 : The weather in Kudahuvadhoo is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kudahuvadhoo
    416 : The weather in Katsuura is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=katsuura
    417 : The weather in Rosetta is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=rosetta
    418 : The weather in Port-Gentil is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=port-gentil
    419 : The weather in Port Hardy is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=port+hardy
    420 : The weather in Mizdah is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=mizdah
    421 : The weather in Zharkent is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=zharkent
    422 : The weather in Port Blair is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=port+blair
    423 : The weather in Tasiilaq is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=tasiilaq
    424 : The weather in Sur is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=sur
    425 : The weather at {'lon': 93.79, 'lat': 18.63} is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=93.7896070834169&lat=18.6340631
    426 : The weather in Luxor is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=luxor
    427 : The weather in Emmett is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=emmett
    428 : The weather in Methoni is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=methoni
    429 : The weather in Hoi An is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=hoi+an
    430 : The weather in Marawi is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=marawi
    431 : The weather in Pundaguitan is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=pundaguitan
    432 : The weather in Amazar is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=amazar
    433 : The weather in Vestmanna is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=vestmanna
    434 : The weather in Ambilobe is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ambilobe
    435 : The weather in Kathu is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kathu
    436 : The weather in Durango is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=durango
    437 : The weather in Zheleznodorozhnyy is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=zheleznodorozhnyy
    438 : The weather in Korsakovo is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=korsakovo
    439 : The weather in Hobyo is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=hobyo
    440 : The weather in Kostomuksha is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kostomuksha
    441 : The weather in Zhigansk is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=zhigansk
    442 : The weather in Narsaq is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=narsaq
    443 : The weather in Nome is mist
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=nome
    444 : The weather in Thompson is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=thompson
    445 : The weather in Killybegs is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=killybegs
    446 : The weather at {'lon': 57.68, 'lat': -20.31} is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=57.6797351&lat=-20.3106424
    447 : The weather in Kumluca is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=kumluca
    448 : The weather in Piacabucu is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=piacabucu
    449 : The weather in Lao Cai is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=lao+cai
    450 : The weather in Mardin is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=mardin
    451 : The weather in Saint Anthony is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=saint+anthony
    452 : The weather in Chulym is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=chulym
    453 : The weather at {'lon': 31.47, 'lat': 26.75} is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=31.466667&lat=26.75
    454 : The weather in Straumen is thunderstorm with rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=straumen
    455 : The weather in Ulaangom is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ulaangom
    456 : The weather in Bilibino is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bilibino
    457 : The weather in Sangar is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=sangar
    458 : The weather in Acari is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=acari
    459 : The weather in Marion is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=marion
    460 : The weather in Opuwo is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=opuwo
    461 : The weather in Charters Towers is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=charters+towers
    462 : The weather in Ahipara is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ahipara
    463 : The weather in Surok is moderate rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=surok
    464 : The weather in Shirokiy is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=shirokiy
    465 : The weather in Gao is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=gao
    466 : The weather in Trinidad is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=trinidad
    467 : The weather in Pulandian is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=pulandian
    468 : The weather in Pokhara is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=pokhara
    469 : The weather in Peleduy is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=peleduy
    470 : The weather in Magdalena is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=magdalena
    471 : The weather in Biritiba-Mirim is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=biritiba-mirim
    472 : The weather in Panama City is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=panama+city
    473 : The weather in Gunjur is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=gunjur
    474 : The weather in Mokrousovo is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=mokrousovo
    475 : The weather in Quatre Cocos is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=quatre+cocos
    476 : The weather in Chulman is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=chulman
    477 : The weather in Zhob is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=zhob
    478 : The weather in Usinsk is overcast clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=usinsk
    479 : The weather in Lynn Haven is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=lynn+haven
    480 : The weather in Bintulu is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=bintulu
    481 : The weather in Karratha is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=karratha
    482 : The weather in Tsogni is light rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=tsogni
    483 : The weather in Fortuna is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=fortuna
    484 : The weather in Hirara is shower rain
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=hirara
    485 : The weather in Vardo is fog
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=vardo
    486 : The weather in Harper is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=harper
    487 : The weather in Oistins is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=oistins
    488 : The weather in Ibra is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=ibra
    489 : The weather at {'lon': 134.73, 'lat': -14.73} is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&lon=134.7323402&lat=-14.7341439
    490 : The weather in Adamus is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=adamus
    491 : The weather in Tazovskiy is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=tazovskiy
    492 : The weather in Khani is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=khani
    493 : The weather in Atbasar is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=atbasar
    494 : The weather in Jumla is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=jumla
    495 : The weather in Pandamatenga is few clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=pandamatenga
    496 : The weather in Arauca is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=arauca
    497 : The weather in Khatassy is clear sky
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=khatassy
    498 : The weather in Matagami is scattered clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=matagami
    499 : The weather in Half Moon Bay is broken clouds
    http://api.openweathermap.org/data/2.5/weather?units=metric&APPID=6ab6449af7cc8fcd5dc2f8da535473bb&q=half+moon+bay
    


```python
city_df.head()
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
      <th>city_name</th>
      <th>country</th>
      <th>temperature</th>
      <th>humidity</th>
      <th>cloudness</th>
      <th>wind_speed</th>
      <th>lat</th>
      <th>long</th>
      <th>city_name_owm</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tessalit</td>
      <td>ml</td>
      <td>25.97</td>
      <td>17</td>
      <td>8</td>
      <td>2.31</td>
      <td>20.200899</td>
      <td>1.014032</td>
      <td>Tessalit</td>
    </tr>
    <tr>
      <th>1</th>
      <td>punta arenas</td>
      <td>cl</td>
      <td>5</td>
      <td>80</td>
      <td>40</td>
      <td>1.5</td>
      <td>-53.162665</td>
      <td>-70.908098</td>
      <td>Punta Arenas</td>
    </tr>
    <tr>
      <th>2</th>
      <td>busselton</td>
      <td>au</td>
      <td>14.81</td>
      <td>100</td>
      <td>64</td>
      <td>1.86</td>
      <td>-33.644499</td>
      <td>115.348875</td>
      <td>Busselton</td>
    </tr>
    <tr>
      <th>3</th>
      <td>mataura</td>
      <td>pf</td>
      <td>0.06</td>
      <td>85</td>
      <td>0</td>
      <td>1.46</td>
      <td>-23.364216</td>
      <td>-149.522902</td>
      <td>Mataura</td>
    </tr>
    <tr>
      <th>4</th>
      <td>bluff</td>
      <td>nz</td>
      <td>20.62</td>
      <td>26</td>
      <td>0</td>
      <td>4.11</td>
      <td>-46.601404</td>
      <td>168.339868</td>
      <td>Bluff</td>
    </tr>
  </tbody>
</table>
</div>




```python
# This part is to compare the results by calling coordinates and cities

# url = 'http://api.openweathermap.org/data/2.5/weather?'
# check_city_df=pd.DataFrame(columns=["city_name","country","temperature","humidity","cloudness","wind_speed","lat","long","city_name_owm"])
# check_city_df.city_name=city_list
# check_city_df.country=country_list
      
# for index, row in city_df.iterrows(): 
#     city_country=f"{row['city_name']},{row['country']}"
#     print(city_country)
   
#     params = {"units": "metric", "APPID": api_key, "q":row['city_name']}#lon":lon_setting, "lat":city_df.lat[0].round(0)}
#     try:
#         weather_test = requests.get(url, params=params).json()
        
#         temp=weather_test["main"]["temp"]
#         humidity=weather_test["main"]["humidity"]
#         cloud=weather_test["clouds"]["all"]
#         wind=weather_test["wind"]["speed"]
#         city_name_owm=weather_test['name']
        
#         check_city_df.at[index,"temperature"]=temp
#         check_city_df.at[index,"humidity"]=humidity
#         check_city_df.at[index,"cloudness"]=cloud
#         check_city_df.at[index,"wind_speed"]=wind
#         check_city_df.at[index,"city_name_owm"]=city_name_owm
#         print(f"{index} : The weather at {weather_test['coord']} is {weather_test['weather'][0]['description']}")
#     except Exception as e:
#         print(index,":",city_country)
#         print(e)



```

** Create plots and export to PDF **
** Export all the data to csv **


```python
city_df.dtypes
city_df.temperature = pd.to_numeric(city_df.temperature)
city_df.wind_speed = pd.to_numeric(city_df.wind_speed)
city_df.cloudness = pd.to_numeric(city_df.cloudness)
city_df.humidity = pd.to_numeric(city_df.humidity)
```


```python
#create plots
city_df.plot(kind="scatter", x="lat", y="temperature")
plt.title("Temperature vs. Latitude (05/28/2018)")
plt.xlim(-90,90)
plt.xlabel("Latitude (degree)")
plt.ylabel("Temp (C)")
plt.savefig('.\output\celsius.pdf')

```


![png](main_files/main_12_0.png)



```python
city_df.plot(kind="scatter",x="lat",y="humidity")
plt.title("Humidity vs. Latitude (05/28/2018)")
plt.xlim(-90,90)
plt.xlabel("Latitude (degree)")
plt.ylabel("Humidity (%)")
plt.savefig('.\output\humidity.pdf')

```


![png](main_files/main_13_0.png)



```python
city_df.plot(kind="scatter",x="lat",y="cloudness")
plt.title("Cloudness vs. Latitude (05/28/2018)")
plt.xlim(-90,90)
plt.xlabel("Latitude (degree)")
plt.ylabel("Coudness (%)")
plt.savefig(".\output\cloudness.pdf")

```


![png](main_files/main_14_0.png)



```python
wind_fig=city_df.plot(kind="scatter",x="lat",y="wind_speed")

plt.title("Wind speed vs. Latitude (05/28/2018)")
plt.xlim(-90,90)
plt.xlabel("Latitude (degree)")
plt.ylabel("Wind Speed (m/s)")
plt.savefig(".\output\wind_speed.pdf")

```


![png](main_files/main_15_0.png)



```python
#output figs and the data into files
pd.DataFrame.to_csv(city_df,".\output\weather_city_data.csv",sep=',')



```


```python
$ ipython nbconvert --to markdown main.ipynb
```


      File "<ipython-input-32-85620785119d>", line 1
        $ ipython nbconvert --to markdown main.ipynb
        ^
    SyntaxError: invalid syntax
    

