## Páctica BDA

### Idea general

La finalidad de la práctica es facilitar el turísmo de las familias que deseen visitar la ciudad de Barcelona.
 
Por tanto, se obtendrán datos de diferentes páginas web como transporte y sitios turísticos en Barcelona ciudad, para crear un top 50 de los mejores apartamentos, basándonos en el número de habitaciones, la ubicación, servicios del apartamento, tales como lavadora y ascensor, y el precio por noche.
 
### Nombre del producto
Top 50 turismo familiar en Barcelona ciudad.

### Estrategia del DAaaS

Creación de página web con python y flask, sobre los 50 mejores apartamentos en Barcelona para familias con niños, que obtendrá el resultado del segmento de cloud, el cual se actualizará manualmente cada més.

Se volverán a obtener los csv de airbnb y transportes de Barcelona, y se realizará un crawler con scrapy de [los parques](https://www.barcelona.cat/ca/que-pots-fer-a-bcn/parcs-i-jardins/tots-els-parcs).
Se subirán al segmento de cloud.
Se creará el cluster y se realizarán de nuevo las tareas.

El resultado se guardará en la carpeta inputm en el segmento de cloud, y se subira a la web.

#### Arquitectura

Arquitectura Cloud basada en Scrapy + Google Cloud Storage + Hive + Dataproc.

Crawler con scrapy que leerá de [parques de Barcelona](https://www.barcelona.cat/ca/que-pots-fer-a-bcn/parcs-i-jardins/tots-els-parcs) y guardará los datos obtenidos en un csv, que se subirá a un segmento en Google Cloud.

Se insertará en dataset de airbnb y el de TMB en HIVE.
Todos ellos se colocarán en un segmento de Google Cloud.

Desde Google Storage se crearan 3 tablas de HIVE.
Se realizará una query para obtener los apartamentos que dispongan de dos o más habitaciones.
Después con los datos obtenidos se hará una query con un Join que reste la distancia entre cada airbnb y los parques con un máximo de 1km. 
Por último se realizará una query con un Join que reste la distancia entre cada airbnb y las paradas de transporte público.

El resultado de la query estará en Google Storage en el segmento dentro de la carpeta /output.

### Operating Model:

* En primer lugar, se obtendrá un csv de airbnb de [airbnb](https://public.opendatasoft.com/explore/dataset/airbnb-listings/export/?disjunctive.host_verifications&disjunctive.amenities&disjunctive.features&dataChart=eyJxdWVyaWVzIjpbeyJjaGFydHMiOlt7InR5cGUiOiJjb2x1bW4iLCJmdW5jIjoiQ09VTlQiLCJ5QXhpcyI6Imhvc3RfbGlzdGluZ3NfY291bnQiLCJzY2llbnRpZmljRGlzcGxheSI6dHJ1ZSwiY29sb3IiOiJyYW5nZS1jdXN0b20ifV0sInhBeGlzIjoiY2l0eSIsIm1heHBvaW50cyI6IiIsInRpbWVzY2FsZSI6IiIsInNvcnQiOiIiLCJzZXJpZXNCcmVha2Rvd24iOiJyb29tX3R5cGUiLCJjb25maWciOnsiZGF0YXNldCI6ImFpcmJuYi1saXN0aW5ncyIsIm9wdGlvbnMiOnsiZGlzanVuY3RpdmUuaG9zdF92ZXJpZmljYXRpb25zIjp0cnVlLCJkaXNqdW5jdGl2ZS5hbWVuaXRpZXMiOnRydWUsImRpc2p1bmN0aXZlLmZlYXR1cmVzIjp0cnVlfX19XSwidGltZXNjYWxlIjoiIiwiZGlzcGxheUxlZ2VuZCI6dHJ1ZSwiYWxpZ25Nb250aCI6dHJ1ZX0%3D&q=Barcelona&location=16,41.38377,2.15774&basemap=jawg.streets), el csv de los transportes públicos de [tmb](https://opendata-ajuntament.barcelona.cat/data/es/dataset/transports/resource/e07dec0d-4aeb-40f3-b987-e1f35e088ce2#additional-info) y el de los parques de la ciudad de Barcelona de crawlear con scrapy [parques de barcelona](https://www.barcelona.cat/ca/que-pots-fer-a-bcn/parcs-i-jardins/tots-els-parcs)

* En segundo lugar se subirán los datos al segmento de Google Cloud dentro de la carpeta 'csv'.

* En tercer lugar, se creará un cluster en Google Cloud de 3 contenedores, 1 master y 2 workers. 

-----------------------------------------------------------per revisar------------------------
crear una tabla con el csv de airbnb, una para transporte del csv obtenido de la página de tmb.cat,   y otra de los lugares turísticos obtenido de la página barcelonaturismo.com, de esta mima también se obtendrían la ubicación de parques infantiles.

Una vez tengamos toda esa información, habría que empezar descartando los pisos tengan menos de dos habitaciones o camas, y a partir de alli hacer un join, no se muy bien aún que datos mostraría por por pantalla supongo que lo suyo sería el número de habitaciones, la url al apartamento y el precio por noche

Los resultados que se obtendrían de todo esto sería:

* Primero eliminaríamos los pisos con menos de dos habitaciones.
* Seguidamente de haría un join para comprobar cuales se encuentran dentro del radio de 1 km de distancia de algún parque infantil.
* Con los restantes se haría un join para obtener los que se encuentren dentro del radio de 1km de distancia de paradas de transporte público.
* Con todo esto, ya tendríamos los datos que queremos para obtener el top 50 de los pisos, de los que se mostraría, la dirección del piso, los parques, el transporte público, número de habitaciones, servicios que ofrece el piso y el precio por noche.

Se ha calculado 1 km, por que la media de tiempo en adultos para recorrerlo está en los 12 minutos.

Montar una web con el resultado obtenido de la query. 

### Desarrollo

#### Creación del clúster en Google Cloud:
Se crearán en la Región de [europe-west2](https://github.com/enkhara/PracticaDBA/blob/master/Regiones-google-cloud.jpg) en la Zona [europe-west2-c](https://github.com/enkhara/PracticaDBA/blob/master/Zonas-google-cloud.jpg).
Modo del clúster:
``Estándar (nodos maestros: 1; nodos de trabajo: N)``

 -Nodo maestro:
 
![imágen configuración](https://github.com/enkhara/PracticaDBA/blob/master/Config-nodo-maestro.jpg)

Nodos workers:

![imágen configuración](https://github.com/enkhara/PracticaDBA/blob/master/Config-nodos-workers.jpg)

* Tabla de aibnb: 
``CREATE TABLE airbnb (ID INT, Listing_Url STRING, Scrape_ID STRING, Last_Scraped STRING, Name STRING, Summary STRING, Space STRING, Description STRING, Experiences_Offered STRING, Neighborhood_Overview STRING, Notes STRING, Transit STRING, Access STRING, Interaction STRING, House_Rules STRING, Thumbnail_Url STRING, Medium_Url STRING, Picture_Url STRING, XL_Picture_Url STRING, Host_ID STRING, Host_URL STRING, Host_Name STRING, Host_Since STRING, Host_Location STRING, Host_About STRING, Host_Response_Time STRING, Host_Response_Rate STRING, Host_Acceptance_Rate STRING, Host_Thumbnail_Url STRING, Host_Picture_Url STRING, Host_Neighbourhood STRING, Host_Listings_Count STRING, Host_Total_Listings_Count STRING, Host_Verifications STRING, Street STRING, Neighbourhood STRING, Neighbourhood_Cleansed STRING, Neighbourhood_Group_Cleansed STRING, City STRING, State STRING, Zipcode STRING, Market STRING, Smart_Location STRING, Country_Code STRING, Country STRING, Latitude STRING, Longitude STRING, Property_Type STRING, Room_Type STRING, Accommodates STRING, Bathrooms STRING, Bedrooms STRING, Beds STRING, Bed_Type STRING, Amenities STRING, Square_Feet STRING, Price FLOAT, Weekly_Price STRING, Monthly_Price STRING, Security_Deposit STRING, Cleaning_Fee STRING, Guests_Included STRING, Extra_People STRING, Minimum_Nights INT, Maximum_Nights INT, Calendar_Updated STRING, Has_Availability STRING, Availability_30 STRING, Availability_60 STRING, Availability_90 STRING, Availability_365 STRING, Calendar_last_Scraped STRING, Number_of_Reviews STRING, First_Review STRING, Last_Review STRING, Review_Scores_Rating STRING, Review_Scores_Accuracy STRING, Review_Scores_Cleanliness STRING, Review_Scores_Checkin STRING, Review_Scores_Communication STRING, Review_Scores_Location STRING, Review_Scores_Value STRING, License STRING, Jurisdiction_Names STRING, Cancellation_Policy STRING, Calculated_host_listings_count STRING, Reviews_per_Month STRING, Geolocation STRING, Features STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ';'; ``

* Tabla de tmb:
``CREATE TABLE  tmb (CODI_CAPA STRING,CAPA_GENERICA STRING, NOM_CAPA STRING, ED50_COORD_X FLOAT, ED50_COORD_Y FLOAT, ETRS89_COORD_X FLOAT, ETRS89_COORD_Y FLOAT, LONGITUD FLOAT, LATITUD FLOAT, EQUIPAMENT STRING, DISTRICTE STRING, BARRI STRING, NOM_DISTRICTE STRING, NOM_BARRI STRING, ADRECA STRING, TELEFON STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';``


* Query de HIVE.
- Query para seleccionar los apartamentos con 2 o más habitaciones:
``SELECT * FROM TABLE airbnb WHERE Bedrooms >= '2';``
* Pantallazo final.

### Diagrama

### Crawler

https://colab.research.google.com/github/enkhara/PracticaDBA/blob/master/Georgina_SCRAPY.ipynb#scrollTo=Lvz1BspfpvTl
