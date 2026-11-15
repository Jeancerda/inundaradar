# InundaRadar · ¿Se puede inundar tu zona? / Could your area flood?

**La danza del agua vista por el radar NISAR · The dance of water seen by NISAR radar**

NASA Space Apps Challenge 2026 · Reto / Challenge: **[Dancing with the SARs](https://www.spaceappschallenge.org/2026/challenges/dancing-with-the-sars/)** · Sede / Local event: Sangolquí, Ecuador (ESPE)

🔗 **Demo:** `https://<usuario>.github.io/<repo>/` *(reemplazar al publicar / replace after publishing)* · 📄 Licencia / License: [MIT](LICENSE) · 🤖 [Declaración de IA / AI statement](docs/declaracion_IA.md)

![InundaRadar: caso de la cuenca del Guayas, 31 de octubre de 2026](assets/img/captura_app.jpg)

![Antes y después: NISAR sobre la cuenca del Guayas (25 de septiembre vs. 31 de octubre de 2026, track 112 descendente)](assets/img/antes_despues_guayas.jpg)

---

## 🇪🇨 Español

### El problema
El fenómeno de **El Niño 2026** ya está inundando la Costa ecuatoriana. Según la Secretaría Nacional de Gestión de Riesgos, entre el 29 de agosto y el 9 de noviembre de 2026 hubo **12 fallecidos, 58 540 personas afectadas y 14 099 viviendas afectadas**. El 13 de noviembre se desbordaron 12 ríos y Milagro amaneció con agua «hasta el pecho». Cuando más llueve, los satélites ópticos solo ven nubes, y la gente se pregunta lo mismo: **¿se va a inundar mi barrio?**

### La solución
**InundaRadar** es una app web gratuita, bilingüe y para celular que usa el radar **NASA-ISRO NISAR** (banda L, ve a través de las nubes, de día y de noche) para mostrar **dónde y cuándo aparece el agua**. Con eso responde, en lenguaje simple:

1. **Busca tu barrio** (o toca el mapa o usa tu ubicación).
2. Recibes un **semáforo de riesgo** que combina:
   - la **susceptibilidad del terreno**: altura sobre el río más cercano (HAND), agua vista por NISAR alrededor y agua dinámica;
   - la **amenaza de los próximos días**: lluvia pronosticada y caudal del río (GloFAS) comparado con lo normal.
3. Ves la **serie de tiempo NISAR de tu punto**, calculada en vivo en tu navegador para cualquier lugar del mundo, y **qué hacer** según el nivel.
4. Exploras **la danza del agua** en zonas de estudio: animación por fecha, comparador antes/después, mapas de frecuencia, de cambio y de susceptibilidad.

### Resultados (cuenca baja del Guayas, El Niño 2026)
- **20 fechas NISAR GCOV** (del 21 de junio al 31 de octubre de 2026). Para comparar se usan fechas de la **misma geometría (track 112 descendente)**.
- Área de agua: **506 km² (25 de septiembre) → 1 118 km² (31 de octubre), ×2.2 en cinco semanas**. En ese mismo mes, **NASA POWER registró 574 mm de lluvia** (en octubre de 2025 fueron 22 mm).
- **Validación contra OPERA DSWx-S1** (Sentinel-1, sensor independiente; 14 pares a ≤ 2 días): **exactitud 95.7 %, κ = 0.53, correlación temporal del área r = 0.93**. En el río Napo (Amazonía, 25 pares) la exactitud es de **99.3 %, con F1 = 0.77 y κ = 0.77**.
- **El 76 % de la cuenca baja del Guayas está a menos de 5 m sobre el cauce más cercano (HAND).**

![Serie de tiempo NISAR vs. DSWx-S1 y lluvia NASA POWER](assets/img/serie_guayas.png)

### Datos de la NASA y de agencias aliadas
| Dataset | Uso | Acceso |
|---|---|---|
| **NISAR L2 GCOV** (NASA/JPL-ISRO, ASF DAAC): HH/HV, banda L | Clasificación de agua y cambio | [CMR](https://cmr.earthdata.nasa.gov/search/granules?short_name=NISAR_L2_GCOV_V1) · [GIBS](https://nasa-gibs.github.io/gibs-api-docs/) · [ASF Vertex](https://search.asf.alaska.edu/) · [Guía NISAR](https://nisar-docs.asf.alaska.edu/) |
| **OPERA L3 DSWx-S1** (NASA/JPL) | Referencia de validación | [GIBS](https://nasa-gibs.github.io/gibs-api-docs/) · [PO.DAAC](https://podaac.jpl.nasa.gov/) |
| **NASA POWER** (PRECTOTCORR) | Lluvia diaria histórica | [power.larc.nasa.gov](https://power.larc.nasa.gov/) |
| **GPM IMERG** (NASA) | Mapa de lluvia | GIBS |
| **SEDAC Natural Disaster Hotspots** (NASA) | Peligro histórico de inundación, 1985-2003 | GIBS |
| **VIIRS Flood y True Color** (NASA LANCE) | Comparación óptica (nubes) | GIBS |
| **ASF Global 30 m HAND** (Copernicus GLO-30) | Altura sobre el drenaje | [AWS Open Data](https://registry.opendata.aws/glo-30-hand/) |
| **GloFAS** (Copernicus EMS) vía Open-Meteo | Pronóstico de caudal | [open-meteo.com](https://open-meteo.com/en/docs/flood-api) |

### Método (pipeline reproducible)
```
pipeline/01_catalogo_nisar.py   CMR → 434 escenas GCOV únicas sobre Ecuador (1.5 TB); browse PNG públicos de ASF
pipeline/02_descargar_gibs.py   GIBS WMS → mosaicos NISAR por fecha y OPERA DSWx-S1 (pares a ≤ 2 días)
pipeline/03_hand.py             HAND de ASF (COG en AWS) → remuestreo a la malla de cada zona
pipeline/04_agua_y_riesgo.py    calibración contra DSWx-S1, clasificación de agua, frecuencia, cambio, ISI y validación
pipeline/05_lluvia_power.py     lluvia diaria NASA POWER por zona
pipeline/06_exportar_web.py     empaqueta todo para la app (WebP + data/datos_zonas.js)
pipeline/07_figuras.py          figuras para README y diapositivas
```
- **Regla de agua:** `agua = (L₃ₓ₃ < T  ∨  B₃ₓ₃ − max(R,G) > Δ) ∧ HAND < Hmax`. Se calibra con una búsqueda en malla que maximiza el F1 contra DSWx-S1 y da **T = 50, Δ = 0, Hmax = 10 m**. En el falso color de GIBS, la co-pol (HH) va en rojo y azul y la cross-pol (HV) en verde. El agua calma (reflexión especular) se ve oscura o azul.
- **Índice de susceptibilidad (0-100):** `ISI = 100·(0.55·f(HAND) + 0.30·min(1, 3·f̄_NISAR) + 0.15·dinámica)`.
- **Riesgo:** matriz 5×5 entre el nivel de ISI y el nivel de amenaza (lluvia a 72 h, caudal sobre la mediana anual y suelo saturado).
- **En vivo (cualquier lugar):** la app consulta CMR por las fechas NISAR del punto, pide a GIBS WMS una ventana de 56 m/píxel por fecha (la escala de calibración), clasifica con la misma regla y lee el HAND del COG por rangos HTTP.

### Cómo ejecutar
```bash
# App (cualquier servidor estático; también funciona abriendo index.html directamente)
python -m http.server 8000      # → http://localhost:8000

# Pipeline (opcional; regenera data/)
pip install -r pipeline/requirements.txt
python pipeline/01_catalogo_nisar.py && python pipeline/02_descargar_gibs.py && python pipeline/03_hand.py
python pipeline/04_agua_y_riesgo.py && python pipeline/05_lluvia_power.py && python pipeline/06_exportar_web.py
```

### Limitaciones
- Usamos el falso color de GIBS (derivado de GCOV) y no el γ⁰ calibrado en dB. El siguiente paso es usar Harmony y Earthdata para extraer HHHH y HVHV en COG.
- GIBS tiene NISAR sobre Ecuador desde junio de 2026, así que la temporada lluviosa de 2026 no está cubierta.
- Las superficies muy lisas (pistas, arena seca) se ven oscuras como el agua, y las ciudades dan doble rebote. En montaña hay sombra radar, que mitigamos con HAND.
- **Es un indicador orientativo. No reemplaza las alertas oficiales de la SNGR ni del INAMHI.**

### Hoja de ruta
Alertas por WhatsApp o SMS, integración con la SNGR, los COE y el ECU 911, γ⁰ calibrado, más provincias (Manabí, Esmeraldas, El Oro, Los Ríos), modo offline para zonas rurales y lahares del Cotopaxi en el Valle de los Chillos.

---

## 🌎 English

### The problem
**El Niño 2026** is already flooding Ecuador's coast. According to the National Risk Management Secretariat, between Aug 29 and Nov 9, 2026 there were **12 deaths, 58,540 people affected and 14,099 homes affected**. On Nov 13, 12 rivers overflowed and Milagro woke up with water "up to the chest". When it rains the most, optical satellites only see clouds, and people keep asking: **will my neighborhood flood?**

### The solution
**InundaRadar** is a free, bilingual, mobile-friendly web app that uses the **NASA-ISRO NISAR** radar (L-band; sees through clouds, day and night) to show **where and when water appears**. It answers in plain language:

1. **Search your neighborhood** (or tap the map, or use your location).
2. Get a **traffic-light risk rating** that combines:
   - **terrain susceptibility**: height above nearest drainage (HAND), NISAR-observed water nearby and dynamic water;
   - **hazard in the coming days**: forecast rain and river discharge (GloFAS) compared with normal.
3. See the **NISAR time series for your point**, computed live in your browser anywhere on Earth, and **what to do** for each level.
4. Explore **the dance of water** in study areas: per-date animation, before/after swipe, and frequency, change and susceptibility maps.

### Results (lower Guayas basin, El Niño 2026)
- **20 NISAR GCOV dates** (Jun 21 to Oct 31, 2026). Comparisons use dates from the **same geometry (track 112 descending)**.
- Water area: **506 km² (Sep 25) → 1,118 km² (Oct 31), ×2.2 in five weeks**. That same month, **NASA POWER recorded 574 mm of rain** (22 mm in Oct 2025).
- **Validation against OPERA DSWx-S1** (Sentinel-1, an independent sensor; 14 pairs ≤ 2 days apart): **95.7% accuracy, κ = 0.53, water-area time-series correlation r = 0.93**. On the Napo River (Amazon, 25 pairs) accuracy is **99.3%, with F1 = 0.77 and κ = 0.77**.
- **76% of the lower Guayas basin lies less than 5 m above the nearest channel (HAND).**

![NISAR time series vs DSWx-S1 and NASA POWER rainfall](assets/img/serie_guayas_en.png)

### NASA and partner data
NISAR L2 GCOV (ASF DAAC via CMR, GIBS), OPERA L3 DSWx-S1, NASA POWER, GPM IMERG, SEDAC Natural Disaster Hotspots, VIIRS (LANCE), plus ASF Global 30 m HAND (Copernicus GLO-30) and GloFAS (Copernicus EMS) via Open-Meteo. Links are in the table above.

### Method
A reproducible Python pipeline (steps 01-07 above): CMR catalog → GIBS imagery → HAND → calibration and validation against DSWx-S1 → water masks, frequency, change, susceptibility index → NASA POWER rain → web export. Water rule: `water = (L₃ₓ₃ < T ∨ B₃ₓ₃ − max(R,G) > Δ) ∧ HAND < Hmax`, with T = 50, Δ = 0 and Hmax = 10 m from a grid search maximizing F1. The live mode applies the same rule in the browser at the calibration scale (56 m/px) for any point on Earth.

### Run it
`python -m http.server 8000` and open `http://localhost:8000` (or just open `index.html`). Pipeline: `pip install -r pipeline/requirements.txt` and then run steps 01 to 07.

### Limitations
GIBS false color, not calibrated γ⁰ dB. NISAR over Ecuador in GIBS only since June 2026. Smooth surfaces and urban double bounce cause confusion, and mountains cause radar shadow (mitigated with HAND). **This is guidance only and does not replace official SNGR or INAMHI warnings.**

---

## Créditos y transparencia · Credits & transparency
- **Software:** Leaflet 1.9.4, Chart.js 4.4.1, geotiff.js 2.1.3 (included in `assets/vendor/`); Python: NumPy, SciPy, scikit-image, scikit-learn, rasterio, Pillow, requests, matplotlib.
- **Referencias / References:** NASA-ISRO SAR Mission Science Users' Handbook (2025, doi:10.48577/jpl.UD4HV3) · ASF NISAR Data User Guide (doi:10.5281/ZENODO.20075813) · Nobre et al. (2011), *J. Hydrology* 404 · Martinis et al. (2015), *ISPRS J.* 104 · Twele et al. (2016), *IJRS* 37(13) · OPERA DSWx-S1 ATBD · Dilley et al. (2005), Natural Disaster Hotspots.
- **Noticias / News:** [Primicias, 13 Nov 2026](https://www.primicias.ec/sociedad/lluvias-elnino-ecuador-inundaciones-milagro-babahoyo-guayas-13noviembre-137120/) · [Extra, 10 Nov 2026](https://www.extra.ec/noticia/ecuador/el-nino-ecuador-16-provincias-lluvias-rios-desbordados-10-noviembre-2026-167483.html).
- **IA / AI:** see [docs/declaracion_IA.md](docs/declaracion_IA.md). No AI-generated images; no NASA logo.
- Este proyecto no está afiliado a la NASA ni respaldado por ella. *This project is not affiliated with or endorsed by NASA.*
