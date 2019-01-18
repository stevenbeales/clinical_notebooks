UPDATE: pretty sure I've not done my dataframe preparation right... should probably aggregate to individual practices in the prescribing data then map to CCGs then aggregate practices to CCGs.. and do more sanity checks. Think have fixed for iron analysis... will revisit shortly.


So on [this american life](http://www.thisamericanlife.org/radio-archives/episode/505/use-only-as-directed) this week they suggested that there might be more liver failure occuring due to paracetamol than we realise... I thought it'd be fun to see if there is an association between paracetamol usage and liver deaths per 10,000 population by CCG.

Let's practice Choropleth makage again by making a Paracetamol Choropleth using open data, ogr2ogr, folium and pandas (and indirectly leaflet.js, d3.js, GeoJSON, open street maps and moar) and talk about using gist and blocks. I will skip steps from last [time](http://bl.ocks.org/drcjar/6621154).

0. get some prescribing data from here www.hscic.gov.uk/gpprescribingdata e.g 

   ```
   wget 'http://datagov.ic.nhs.uk/presentation/2013_04_April/T201304PDPI+BNFT.csv'
   ``` 

1. we can re-use the ccg boundaries GeoJSON we made last time which is [here](https://gist.github.com/drcjar/6716180#file-ccgs-json). 

2. so now all we need to do is prepare our data (the prescribing data) such that it has a column that matches our CCG Codes in the GeoJSON and a column with our thing of interest. I've decided our thing of interest is the number of items prescribed containing paracetamol per 10,000 population by CCG so we want a dataframe with that and our CCG Code as headings. 

3. a bit of footwork is required to prepare the dataset because our prescribing data has individual GP practices so we need to map them to CCGs. We also need to add some CCG population size data into the mix in order to the per 10,000 bit. 

   ```
   wget 'http://www.connectingforhealth.nhs.uk/systemsandservices/data/ods/ccginterim/interimpcmem_v5.zip'
   wget 'https://indicators.ic.nhs.uk/download/Clinical%20Commissioning%20Group%20Indicators/Data/CCG_registered_patients_2012.csv'
   ```
The above .csv file from connecting for health tells us which practice belongs to which CCG without too much trouble. The above .csv file of 'CCG_registered_patients_2012' is helpfully broken down into age ranges which we don't care about for our choropleth making hackery so we roll them up. All of this is done using python pandas (see [ipython notebook part one](http://nbviewer.ipython.org/urls/gist.github.com/drcjar/6716180/raw/e6b49264e5cd5ef61f94db4545bb19b487f49c49/gp_to_ccg_code_map.ipynb)).

5. make the choropleth using [folium](https://github.com/wrobstory/folium) magic python code (see [ipython notebook part two](http://nbviewer.ipython.org/urls/gist.github.com/drcjar/6716180/raw/87f06edabd1038a7a2bd444ad808b8cddd75c4ee/ParacetamolAnalytics.ipynb))

   ```python
   map = folium.Map(location=[54.2, -2.45], zoom_start=5)
   map.geo_json(geo_path=ccg_geo, data_out='data10.json', data=para_analysis,
      columns=['CCG13CD', 'Per_person_para_by_ccg'],
      key_on='feature.properties.CCG13CD',
      threshold_scale=[5, 6, 7, 8, 9, 10]
      fill_color='PuBu', fill_opacity=0.7, line_opacity=0.3,
      legend_name='Number of paracetamol items prescribed in April per 10,000 population by CCG')
   map.create_map(path='map_10.html')
   ```
6. kinda need some data on liver deaths now... 
