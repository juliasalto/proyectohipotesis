# Proyecto 2: Hipótesis
1. [Ficha técnica](#ficha-técnica-proyecto-de-análisis-de-datos)
2. [Procesamiento y análisis](#procesamiento-y-análisis)
     - [SQL en Big Query](#sql-en-bigquery)
     - [Hipótesis: Pruebas de Shapiro Wilk, Mann Whitney U, Regresión lineal](#hipótesis-pruebas-de-shapiro-wilk-mann-whitney-u-regresión-lineal)
3. [Conclusiones generales](#conclusiones-generales)
   
# Ficha Técnica: Proyecto de Análisis de Datos
- **Título del Proyecto: Hipótesis**
- Enlaces de interés:
    - [Dashboard en PowerBi](https://drive.google.com/file/d/1imqUqeg5PMAqadV52D5vBEr37rrdmGz3/view?usp=sharing)
    - [Presentación](https://drive.google.com/file/d/1bsM08YrfMaNCngJQjkju52jtgrDTvnZL/view?usp=sharing)
    - [Video presentación](https://www.loom.com/share/cefa07de469041f79c391fc519218b3e?sid=5fdafcd1-ec99-491d-8354-9746024ba5a1)
    - [Google Colab - tests de hipótesis para características](https://colab.research.google.com/drive/1Cxjwd7OKF8yIDM7Qi91X4IioylGqlrgH#scrollTo=VBGkFieHx09R)
    - [Google Colab - test de normalidad de datos e hipótesis](https://colab.research.google.com/drive/19farAP8zDu5pr5-pDVn-fW7N1MrnR2BJ?usp=sharing#scrollTo=XW7xqKp6cSUL)
    - [Google Colab - regresión lineal](https://colab.research.google.com/drive/1btsSO5Gya90dgzPDz87_lr2cdawEsV9-?usp=sharing)
    - [Hoja de cálculo para utilizar en Google Colab en tests de normalidad e hipótesis](https://docs.google.com/spreadsheets/d/1DQNR3Y-vPfLrzuN9V1QfOEKk8rihXLR_JFF_M3rdJWM/edit?usp=sharing)
- **Objetivo:**
    - Preparar la información de la base de datos que corresponde a los datos de las reproducciones de canciones más escuchadas en el año 2023 en la plataforma Spotify, Deezer y Apple, para comprender, respaldar y conocer el comportamiento que hace que una canción sea más o menos escuchada en una plataforma. Analizar los datos para convertirlos en información respaldada por cálculos estadísticos.
    - Se pretende obtener las respuestas para validar o refutar las siguientes hipótesis:
        - Las canciones con un mayor BPM (Beats Por Minuto) tienen más éxito en términos de cantidad de streams en Spotify.
        - Las canciones más populares en el ranking de Spotify también tienen un comportamiento similar en otras plataformas como Deezer.
        - La presencia de una canción en un mayor número de playlists se relaciona con un mayor número de streams.
        - Los artistas con un mayor número de canciones en Spotify tienen más streams.
        - Las características de la música influyen en el éxito en términos de cantidad de streams en Spotify.
- **Equipo:** Julieta Salto - Osiris Berbesia
- **Herramientas y Tecnologías:**
    - Google BigQuery para el procesamiento de datos
    - PowerBi para configurar un dashboard
    - Google Colab y Python para realizar pruebas estadísticas
    - Documentación de Google Console
    - Documentación Python
    - OpenAI - ChatGPT
    - Figma para la presentación final
# Procesamiento y análisis:
  - # SQL en BigQuery
      - Nulos:
        ```SELECT
        track_id, track_name, artist_name, artist_count, released_year,released_month, released_day, in_spotify_playlists, in_spotify_charts, streams,
        count(track_id) AS null_track, count(track_name) AS null_track_name, count(artist_name) AS null_artist_name, count(artist_count) AS null_artist_count, count(released_year) AS null_year, count(released_month) AS null_month, count(released_day) AS null_day, count(in_spotify_playlists) AS null_playlists, count(in_spotify_charts) AS null_chart, count(streams) AS null_streams
         FROM
        saltoproyecto2hipotesis.datos_hipotesis.track_in_spotify
        WHERE
        track_id is null and track_name is null and artist_name is null and artist_count is null and released_year is null and released_month is null and released_day is null and in_spotify_playlists is null and in_spotify_charts is null and streams is null
        GROUP BY
        track_id, track_name, artist_name, artist_count, released_year,released_month, released_day, in_spotify_playlists, in_spotify_charts, streams```
      - Duplicados:
        ```SELECT
        track_name, artist_name,
        count(*) AS cantidad
        FROM `saltoproyecto2hipotesis.datos_hipotesis.track_in_spotify`
        GROUP BY
        track_name, artist_name
        HAVING COUNT(*) > 1```
      - Fuera del alcance:
        ```SELECT *
        EXCEPT(key, mode)
        FROM
        `saltoproyecto2hipotesis.datos_hipotesis.track_technical_info`
      - Máximos, mínimos, promedios:
        ```SELECT
        MAX(`speechiness_%`) AS `max_speechiness_%`, MIN(`speechiness_%`) AS `min_speechiness_%`, AVG(`speechiness_%`) AS `avg_speechiness_%`, MAX(`danceability_%`) AS `max_danceability_%`, MIN(`danceability_%`) AS `min_danceability_%`, AVG(`danceability_%`) AS `avg_danceability_%`, MAX(`valence_%`) AS `max_valence_%`, MIN(`valence_%`) AS `min_valence_%`, AVG(`valence_%`) AS `avg_valence%`, MAX(`energy_%`) AS `max_energy_%`, MIN(`energy_%`) AS `min_energy_%`, AVG(`energy_%`) AS `avg_energy_%`,
        FROM
        `saltoproyecto2hipotesis.datos_hipotesis.track_technical_info```
      - Unión de tablas:
        ```SELECT spotify.*
        EXCEPT(track_name, artist_name,
        released_year,
        released_month,
        released_day,
        streams),
        REGEXP_REPLACE(track_name, r'[\p{So}\p{Cn}]', '') AS cleaned_track_name,
        REGEXP_REPLACE(artist_name, r'[\p{So}\p{Cn}]', '') AS cleaned_artist_name,
        CONCAT (CAST(released_year AS string),"-", CAST(released_month AS string),"-", CAST(released_day AS string)) AS released_date,
        CAST(streams AS integer) AS new_streams,
        techical_info.* EXCEPT(track_id),
        track_in_competition.* EXCEPT(track_id),
        FROM
        `saltoproyecto2hipotesis.datos_hipotesis.track_in_spotify` AS spotify
         LEFT JOIN
        `saltoproyecto2hipotesis.datos_hipotesis.track_technical_info` AS techical_info
        ON
        spotify.track_id = techical_info.track_id
        LEFT JOIN
        `saltoproyecto2hipotesis.datos_hipotesis.track_in_competition` AS track_in_competition
        ON
        spotify.track_id = track_in_competition.track_id
        WHERE
        streams NOT LIKE "%B%"
        AND NOT track_name IN("SNAP",
        "About Damn Time",
        "Take My Breath",
        "SPIT IN MY FACE!")```
      - Cuartiles:
        ```WITH
        cuartiles AS(
        SELECT
        new_streams,
        NTILE(4) OVER(ORDER BY new_streams) AS cuartiles_streams
        FROM
        `datos_hipotesis.view_unificado_3` )
        SELECT
        unificado.*,
        cuartiles.cuartiles_streams,
        IF(cuartiles.cuartiles_streams = 4, "alto", if(cuartiles.cuartiles_streams = 3, "medio alto", if(cuartiles.cuartiles_streams = 2, "medio bajo", "bajo"))) as categoria_streams
        FROM
        `datos_hipotesis.view_unificado_3` unificado
        LEFT JOIN!
        cuartiles
        ON
        unificado.new_streams=cuartiles.new_streams
      La query para la tabla consolidada que usé para la mayoría de los hitos se ve así
      [Captura de Pantalla 2024-07-01 a la(s) 12 35 25](https://github.com/juliasalto/proyectohipotesis/assets/136622322/f42b277b-177d-4fac-94a5-38012ca419a7)

      - Correlaciones:
        ```SELECT
        CORR(new_streams, sum_playlists) AS correlation_value
        FROM
        `saltoproyecto2hipotesis.datos_hipotesis.4_view_unificado```

# Hipótesis: Pruebas de Shapiro Wilk, Mann Whitney U, Regresión lineal
   - Las canciones con un mayor BPM (Beats Por Minuto) tienen más éxito en términos de cantidad de streams en Spotify.
     - CORRELACIÓN: 0.60767802013080
       ```SELECT
       CORR(new_streams, sum_playlists) AS correlation_value
       FROM
        `saltoproyecto2hipotesis.datos_hipotesis.4_view_unificado````
      - Test Shapiro Wilk:
        
        ![Captura de Pantalla 2024-07-01 a la(s) 14 55 40](https://github.com/juliasalto/proyectohipotesis/assets/136622322/403c1dea-3d1e-4ccb-86d6-03263961852e)
        ![Captura de Pantalla 2024-07-01 a la(s) 14 58 34](https://github.com/juliasalto/proyectohipotesis/assets/136622322/a326df53-736f-4a09-bff2-31fc420d29bd)
        ![Captura de Pantalla 2024-07-01 a la(s) 14 58 49](https://github.com/juliasalto/proyectohipotesis/assets/136622322/c994107f-8742-40d6-b925-5b181e5dbe66)

        Se visualiza en la distribución que no tienen una configuración normal
      - Test Mann Whitney U:
        
        ![Captura de Pantalla 2024-07-01 a la(s) 15 00 54](https://github.com/juliasalto/proyectohipotesis/assets/136622322/2191f882-74e5-4e31-be39-204546a181af)
        
      - Regresión lineal:
        
        ![Captura de Pantalla 2024-07-01 a la(s) 15 02 12](https://github.com/juliasalto/proyectohipotesis/assets/136622322/a7c5e6e0-822a-456d-b075-334e86c4980e)
        ![Captura de Pantalla 2024-07-01 a la(s) 15 02 48](https://github.com/juliasalto/proyectohipotesis/assets/136622322/80bcf66b-df9b-4aaa-b944-88e383b1f45c)

   - Las canciones más populares en el ranking de Spotify también tienen un comportamiento similar en otras plataformas como Deezer.
   - La presencia de una canción en un mayor número de playlists se relaciona con un mayor número de streams.
   - Los artistas con un mayor número de canciones en Spotify tienen más streams.
   - Las características de la música influyen en el éxito en términos de cantidad de streams en Spotify.
  
# Conclusiones generales:
- Consideramos que al lanzar un track debería poder estar en la mayor cantidad de plataformas posible.
- Observamos, además, que los tracks más escuchados (60%) fueron de artistas solistas
- La escala major es la que predomina en toda la muestra de datos, por lo tanto sugerimos considerarla al momento de la composición de un track.
- Las características de la canción no tienen mayor influencia que la que podría llegar a tener el hecho de que los artistas que lideran las cantidades de reproducciones son conocidos/as a nivel mundial.
  
- **Limitaciones/Próximos Pasos:**
    - Sobre las hipótesis: en general, los datos no reflejan una distribución normal, por lo que considero necesario revisar outliers, tamaño de la muestra, entre otros.
    - Profundizar en las nociones de hipótesis y pruebas de hipótesis. 
