## Analysis of Taxi Travel Behavior in Chicago

Import Libraries:
```python
import pandas as pd
import requests
import matplotlib.pyplot as plt
from scipy import stats as st
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
  
```python

```

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
We reject the null hypothesis.
There IS a significant difference in trip duration.
Rain DOES affect the average trip duration.

Average duration on rainy days: 2427.21 seconds
Average duration on dry days: 1999.68 seconds

## Main Conclusions:
1. The Loop neighborhood has the highest number of recorded trips according to Google Maps. Along with River North and other downtown Chicago areas, it stands out for its strategic location in the heart of the city. This result is consistent with urban dynamics: as these are commercial, tourist, and high-density activity areas, it's logical that they have a higher average number of trips.
