## Analysis of Taxi Travel Behavior in Chicago

Import Libraries:
```python
import pandas as pd
import matplotlib.pyplot as plt
from scipy import stats as st
```

I Beautiful Soup, request , and Pandas to extract and create a dataframe named weather_records with the weather information for Chicago in 2017:
```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
URL='https://practicum-content.s3.us-west-1.amazonaws.com/data-analyst-eng/moved_chicago_weather_2017.html'
req = requests.get(URL)
soup = BeautifulSoup(req.text, 'lxml')
table = soup.find('table', attrs={"id": "weather_records"})
heading_table=[]
for row in table.find_all('th'):
    heading_table.append(row.text)   
content=[]
for row in table.find_all('tr'):
    if not row.find_all('th'):
        content.append([element.text for element in row.find_all('td')])
weather_records = pd.DataFrame(content, columns = heading_table)
print(weather_records)
```
Additionally, I used SQL to combine the 'trip' dataframe with the dataframe named 'weather_records'. Then, I created a column classifying the weather as "Good" or "Bad" based on specific conditions:
```SQL
SELECT
    t.start_ts, 
    w.weather_conditions,
    t.duration_seconds
FROM trips t
INNER JOIN (
    SELECT 
        ts,
        CASE
            WHEN description LIKE '%rain%' OR description LIKE '%storm%' THEN 'Bad'
            ELSE 'Good'
        END AS weather_conditions
    FROM weather_records
) w
    ON t.start_ts = w.ts   -- puente entre viajes y clima
WHERE 
    t.pickup_location_id = 50   -- Loop
    AND t.dropoff_location_id = 63  -- O'Hare
    AND EXTRACT(DOW FROM t.start_ts) = 6  -- sábado
ORDER BY t.trip_id;
```
Data description:
1. Company Information: Contains two columns with the following data:
* company_name: Name of the taxi company
* trips_amount: The number of trips for each taxi company on November 15 and 16, 2017.

2.Trips:
* dropoff_location_name: Chicago neighborhoods where trips ended
* average_trips: The average number of trips that ended in each neighborhood in November 2017.

3.This contains data about trips from the Loop to O'Hare International Airport. Remember, these are the field values ​​in the table:
* start_ts: Pickup date and time
* weather_conditions: Weather conditions at the start of the trip
* duration_seconds: Trip duration in seconds

1. Loop and River North are the neighborhoods with the highest concentration of destination trips, suggesting that they are quite popular areas in the city. However, Streeterville and West Loop, while having a considerably lower number of destination trips compared to the other 6 neighborhoods, are also noteworthy. It could be said that there is a significant concentration among these 4 mentioned neighborhoods compared to the others.

<p align="center">
  <img src="https://github.com/Cristian-Tello/Analysis-of-Travel-Patterns-in-Chicago/blob/main/Top10Neighborhoods.png" alt="Sample Image">
</p>

2. Flash Cab has positioned itself as the leader in the taxi service market, demonstrating clear dominance over its competitors. In contrast, the other companies have a more fragmented and distributed market share, without achieving significant individual market share.
<p align="center">
  <img src="https://github.com/Cristian-Tello/Analysis-of-Travel-Patterns-in-Chicago/blob/main/Top10TripsCompanies.png" alt="Sample Image">
</p>

## Test the hypothesis:

1.  Null Hypothesis: "The average travel time from the Loop to O'Hare International Airport changes on rainy Saturdays."
2. Alternative Hypothesis: "The average travel time from the Loop to O'Hare International Airport does not change on rainy Saturdays"

```python
# Grupo 1: Sábados lluviosos
grupo_lluvioso = viajes_datos[
    viajes_datos['weather_conditions'].str.contains('Bad', na=False)
]['duration_seconds']

# Grupo 2: Sábados sin lluvia  
grupo_sin_lluvia = viajes_datos[
    viajes_datos['weather_conditions'].str.contains('Good', na=False)
]['duration_seconds']

# Verificar tamaños de muestra
print(f"Viajes en sábados lluviosos: {len(grupo_lluvioso)}")
print(f"Viajes en sábados sin lluvia: {len(grupo_sin_lluvia)}")

# Configurar nivel de significación
alpha = 0.05

# Aplicar prueba t para muestras independientes
resultado = st.ttest_ind(grupo_lluvioso, grupo_sin_lluvia)

print(f"Estadístico t: {resultado.statistic}")
print(f"Valor p: {resultado.pvalue}")

# Interpretación de resultados
if resultado.pvalue < alpha:
    print("Rechazamos la hipótesis nula")
    print("SÍ hay diferencia significativa en la duración de viajes")
    print("La lluvia SÍ afecta la duración promedio de los viajes")
else:
    print("No podemos rechazar la hipótesis nula") 
    print("NO hay evidencia de diferencia significativa")
    print("La lluvia NO afecta significativamente la duración")

# Mostrar estadísticas descriptivas
print(f"\nDuración promedio en días lluviosos: {grupo_lluvioso.mean():.2f} segundos")
print(f"Duración promedio en días sin lluvia: {grupo_sin_lluvia.mean():.2f} segundos")
```


## Main Conclusions:
1. The Loop neighborhood has the highest number of recorded trips according to Google Maps. Along with River North and other downtown Chicago areas, it stands out for its strategic location in the heart of the city. This result is consistent with urban dynamics: as these are commercial, tourist, and high-density activity areas, it's logical that they have a higher average number of trips.

2. We reject the null hypothesis, There IS a significant difference in trip duration and Rain does affect the average trip duration.

Average duration on rainy days: 2427.21 seconds
Average duration on dry days: 1999.68 seconds
