---
name: clima
description: Obtiene el clima actual y pronóstico de Pinamar, Argentina (ciudad del usuario). Usar cuando pida clima, temperatura, lluvia o pronóstico.
---

# Clima local

Ciudad fija: `Pinamar,Argentina`. Siempre usar esta ciudad, sin importar otra ubicación detectada.

## Pasos

1. Obtener datos (wttr.in, sin API key). En Windows usar PowerShell (no hay Python/jq):

```powershell
$d = Invoke-RestMethod "https://wttr.in/Pinamar,Argentina?format=j1&lang=es"
$c = $d.current_condition[0]
"$($c.lang_es[0].value) $($c.temp_C)C (sens $($c.FeelsLikeC)C) hum $($c.humidity)% viento $($c.windspeedKmph) km/h $($c.winddir16Point) UV $($c.uvIndex)"
foreach($w in $d.weather){ "$($w.date) min $($w.mintempC) max $($w.maxtempC) lluvia $(($w.hourly.chanceofrain | % {[int]$_} | measure -Maximum).Maximum)% $($w.hourly[4].lang_es[0].value)" }
```

2. Verificar que `nearest_area[0].areaName` sea Pinamar (o cercano); si no, informarlo.

3. Responder en español, conciso: condición, temp (°C), sensación, humedad, viento, y tabla de próximos días (min/max, prob. lluvia, cielo).

## Notas

- Si falla la red, informar el error; no inventar datos.
- Unidades métricas (°C, km/h).
- Solo lectura: no modificar archivos del proyecto.
