import json, requests
from geojson import MultiLineString, Feature, FeatureCollection, dump
import pandas as pd
key = '941ffd03a44c71fa98993c359002221b' # api data.mos.ru

datasets = ['2468', '3141', '60601', '61581'] # road repairs datasets numbers for 2016, 2017, 2018, 2019

json_2016 = requests.get('https://apidata.mos.ru/v1/datasets/2468/rows?api_key=%s' % key).json()
# keys differ from other datasets 
for row in json_2016:
    r=row['Cells']
    r['WorksBeginDate'] = r.pop('WorkStartDate')
    r['WorksPlace'] = r.pop('Location')
    r['WorksStatus'] = r.pop('WorkStatus')
    
all_years = json_2016
for year in datasets[1:]:
    all_years += requests.get('https://apidata.mos.ru/v1/datasets/%s/rows?api_key=%s' % (year, key)).json()
    
def get_address(row):
    return row['Cells']['WorksPlace']

def get_date(row):
    return row['Cells']['WorksBeginDate']

def row_to_address_date(row):
    return {'date': get_date(row), 'address': get_address(row), 'id' : row['Cells']['global_id']}

# make CSV for all addresses with repair start date and ID
df = []
for row in all_years:
    df.append(row_to_address_date(row))
pd.DataFrame(df).to_csv('all_adresses_all_years.csv', index = False)

# make normal GeoJSON for all addresses for Tableau
features = []
for row in all_years:
    features.append(Feature(geometry=MultiLineString(row['Cells']['geoData']['coordinates']), 
                            properties={key:row['Cells'][key] for key in ['global_id', 'WorksBeginDate', 'WorksPlace']}))

feature_collection = FeatureCollection(features)

with open('all_years_construction.geojson', 'w') as f:
    dump(feature_collection, f)
