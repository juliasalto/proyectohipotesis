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
        
        ```
        (SELECT COUNT(*) FROM `saltoproyecto2hipotesis.datos_hipotesis.track_in_spotify` WHERE track_id is null) AS null_id,
        (SELECT COUNT(*) FROM `saltoproyecto2hipotesis.datos_hipotesis.track_in_spotify` WHERE track_name is null) AS null_track,
        (SELECT COUNT(*) FROM `saltoproyecto2hipotesis.datos_hipotesis.track_in_spotify` WHERE artist_name is null) AS null_name,
        (SELECT COUNT(*) FROM `saltoproyecto2hipotesis.datos_hipotesis.track_in_spotify` WHERE artist_count is null) AS null_count,
        (SELECT COUNT(*) FROM `saltoproyecto2hipotesis.datos_hipotesis.track_in_spotify` WHERE released_year is null) AS null_year,
        (SELECT COUNT(*) FROM `saltoproyecto2hipotesis.datos_hipotesis.track_in_spotify` WHERE released_month is null) AS null_month,
        (SELECT COUNT(*) FROM `saltoproyecto2hipotesis.datos_hipotesis.track_in_spotify` WHERE released_day is null) AS null_day,
        (SELECT COUNT(*) FROM `saltoproyecto2hipotesis.datos_hipotesis.track_in_spotify` WHERE in_spotify_playlists is null) AS null_playlists,
        (SELECT COUNT(*) FROM `saltoproyecto2hipotesis.datos_hipotesis.track_in_spotify` WHERE in_spotify_charts is null) AS null_charts,
        (SELECT COUNT(*) FROM `saltoproyecto2hipotesis.datos_hipotesis.track_in_spotify` WHERE streams is null) AS null_streams
        
      - Duplicados:
        
        ```
        SELECT
        track_name, artist_name,
        count(*) AS cantidad
        FROM `saltoproyecto2hipotesis.datos_hipotesis.track_in_spotify`
        GROUP BY
        track_name, artist_name
        HAVING COUNT(*) > 1```
        
      - Fuera del alcance:
        
        ```
        SELECT *
        EXCEPT(key, mode)
        FROM
        `saltoproyecto2hipotesis.datos_hipotesis.track_technical_info`
        
      - Máximos, mínimos, promedios:
        
        ```
        SELECT
        MAX(`speechiness_%`) AS `max_speechiness_%`, MIN(`speechiness_%`) AS `min_speechiness_%`, AVG(`speechiness_%`) AS `avg_speechiness_%`, MAX(`danceability_%`) AS `max_danceability_%`, MIN(`danceability_%`) AS `min_danceability_%`, AVG(`danceability_%`) AS `avg_danceability_%`, MAX(`valence_%`) AS `max_valence_%`, MIN(`valence_%`) AS `min_valence_%`, AVG(`valence_%`) AS `avg_valence%`, MAX(`energy_%`) AS `max_energy_%`, MIN(`energy_%`) AS `min_energy_%`, AVG(`energy_%`) AS `avg_energy_%`,
        FROM
        `saltoproyecto2hipotesis.datos_hipotesis.track_technical_info```
        
      - Unión de tablas:
        
        ```
        SELECT spotify.*
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
        "SPIT IN MY FACE!")
        ```
        
      - Cuartiles:
        
        ```
        WITH
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
        unificado.new_streams=cuartiles.new_streams```
        
      La query para la tabla consolidada que usé para la mayoría de los hitos se ve así
    
    ![Captura de Pantalla 2024-07-01 a la(s) 12 35 25](https://github.com/juliasalto/proyectohipotesis/assets/136622322/8ef9a2da-f026-4935-a9a3-0625d0948b9c)

    
      - Correlaciones:
        ```
        SELECT
        CORR(new_streams, sum_playlists) AS correlation_value
        FROM
        `saltoproyecto2hipotesis.datos_hipotesis.4_view_unificado```

# Hipótesis: Pruebas de Shapiro Wilk, Mann Whitney U, Regresión lineal
   - <I>Las canciones con un mayor BPM (Beats Por Minuto) tienen más éxito en términos de cantidad de streams en Spotify.</I>
     - Correlación: -0.0032001857690
       ```
       SELECT
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

  - <i>Las canciones más populares en el ranking de Spotify también tienen un comportamiento similar en otras plataformas como Deezer.</i>
     - Correlación: 0.60767802013080952
          ```
          SELECT
          CORR(in_deezer_charts, in_spotify_charts) as corr_charts,
          FROM
          `saltoproyecto2hipotesis.datos_hipotesis.4_view_unificado````
     
     - Test Shapiro Wilk:
    
       ![Captura de Pantalla 2024-07-01 a la(s) 19 10 31](https://github.com/juliasalto/proyectohipotesis/assets/136622322/d9d442f4-b2ec-4afb-b3a6-a444d1f45d20)
       ![Captura de Pantalla 2024-07-01 a la(s) 19 12 02](https://github.com/juliasalto/proyectohipotesis/assets/136622322/1aaaf95f-bb21-409d-a467-c8dd1095cbab)
       ![Captura de Pantalla 2024-07-01 a la(s) 19 12 15](https://github.com/juliasalto/proyectohipotesis/assets/136622322/13fbb3f6-70db-4880-941b-a3c119392b18)

       Se visualiza en la distribución que no tienen una configuración normal
       
     - Test Mann Whitney U:

       ![Captura de Pantalla 2024-07-01 a la(s) 19 12 29](https://github.com/juliasalto/proyectohipotesis/assets/136622322/6f1e964d-dbb2-4388-b7b6-6251f1454c52)

     - Regresión lineal:
    
       ![Captura de Pantalla 2024-07-01 a la(s) 19 14 54](https://github.com/juliasalto/proyectohipotesis/assets/136622322/90be3aa0-fd4b-40b2-8613-62a1ea071d2d)
       ![Captura de Pantalla 2024-07-01 a la(s) 19 15 09](https://github.com/juliasalto/proyectohipotesis/assets/136622322/cd6414b7-ca89-4c96-920d-df9ad898bf86)


  - <i>La presencia de una canción en un mayor número de playlists se relaciona con un mayor número de streams.</i>
     - Correlación: 0.783680301078940
       ```
       SELECT
       CORR(sum_playlists, new_streams) as corr_playlists,
       FROM
       `saltoproyecto2hipotesis.datos_hipotesis.4_view_unificado`
       
     - Test Shapiro Wilk
       
       ![Captura de Pantalla 2024-07-01 a la(s) 19 20 23](https://github.com/juliasalto/proyectohipotesis/assets/136622322/746936d4-2662-432b-a61f-6583e9495d35)
       ![Captura de Pantalla 2024-07-01 a la(s) 19 21 13](https://github.com/juliasalto/proyectohipotesis/assets/136622322/e9c0ad3f-d56d-44fd-b496-f952248a4953)
       ![Captura de Pantalla 2024-07-01 a la(s) 19 21 44](https://github.com/juliasalto/proyectohipotesis/assets/136622322/77fbf5e3-4738-4991-a039-2b9256cbc2ca)
       
       Se visualiza en la distribución que no tienen una configuración normal
       
     - Test Mann Whitney U:

       ![Captura de Pantalla 2024-07-01 a la(s) 19 20 36](https://github.com/juliasalto/proyectohipotesis/assets/136622322/b526148b-8dc5-471e-8ac6-f7d577074f41)

     - Regresión lineal:

     ![Captura de Pantalla 2024-07-01 a la(s) 19 23 45](https://github.com/juliasalto/proyectohipotesis/assets/136622322/888eba5d-1380-4cf0-82c3-17f97ff44c46)
     ![Captura de Pantalla 2024-07-01 a la(s) 19 23 56](https://github.com/juliasalto/proyectohipotesis/assets/136622322/b85e229e-d7da-44f2-b5c0-e87d31c1606e)
 
  - <i>Los artistas con un mayor número de canciones en Spotify tienen más streams.</i>
     - Correlación: 
       ```
       WITH track_count AS (
       SELECT
       cleaned_artist_name,
       COUNT(track_id) AS track_count,
       SUM(new_streams) AS total_streams
       FROM
       `saltoproyecto2hipotesis.datos_hipotesis.4_view_unificado`
       GROUP BY
       cleaned_artist_name
       )
       SELECT *,
       CORR(track_count.track_count, track_count.total_streams) OVER () AS correlation_value,
       FROM
       track_count

     - Test Shapiro Wilk:
       
       ![Captura de Pantalla 2024-07-01 a la(s) 19 26 44](https://github.com/juliasalto/proyectohipotesis/assets/136622322/b3f9e9b3-7de9-4eb7-873b-997d84af73e2)
       ![Captura de Pantalla 2024-07-01 a la(s) 19 28 00](https://github.com/juliasalto/proyectohipotesis/assets/136622322/08c8f6a4-50a6-475f-9f93-4154c679d553)
       ![Captura de Pantalla 2024-07-01 a la(s) 19 28 00](https://github.com/juliasalto/proyectohipotesis/assets/136622322/121f7ba3-3d6d-4834-8f29-37edb659e53e)

     - Test Mann Whitney U:

     ![Captura de Pantalla 2024-07-01 a la(s) 19 28 18](https://github.com/juliasalto/proyectohipotesis/assets/136622322/c7bb0971-2109-4045-832d-e76460dc90c8)

     - Regresión lineal:

       ![Captura de Pantalla 2024-07-01 a la(s) 19 30 11](https://github.com/juliasalto/proyectohipotesis/assets/136622322/da0b9487-3f36-4cca-8d25-c2541ea561d4)
       ![Captura de Pantalla 2024-07-01 a la(s) 19 30 22](https://github.com/juliasalto/proyectohipotesis/assets/136622322/19381a0e-7d84-4188-ad8a-c0f571fea2ed)
    
  - <i>Las características de la música influyen en el éxito en términos de cantidad de streams en Spotify.</i>
    - Correlación:
           - Danceability: -0.1056358995505
           - Valence: -0.04179795486959
           - Energy: -0.025738176754
           - Liveness: -0.051147025245
           - Speechiness: -0.11277393515058
           - Acoustciness: -0.00498576864951
           - Instrumentalness: -0.044039985415
      
      ```
      SELECT
      CORR(new_streams, danceability) AS corr_danceability,
      CORR(new_streams, valence) AS corr_valence,
      CORR(new_streams, energy) AS corr_energy,
      CORR(new_streams, liveness) AS corr_liveness,
      CORR(new_streams, speechiness) AS corr_speechiness,
      CORR(new_streams, acousticness) AS corr_acousticness,
      CORR(new_streams, instrumentalness) AS corr_instrumentalness,
      FROM
      `saltoproyecto2hipotesis.datos_hipotesis.4_view_unificado`
      
    - Test Shapiro Wilk:
      
     ![Captura de Pantalla 2024-07-01 a la(s) 19 44 39](https://github.com/juliasalto/proyectohipotesis/assets/136622322/c9b3ccbd-a91f-4408-995c-47152eb85c4a)
     ![Captura de Pantalla 2024-07-01 a la(s) 19 44 46](https://github.com/juliasalto/proyectohipotesis/assets/136622322/67a37135-1c69-4b1a-90f1-88f81253bca5)
     ![Captura de Pantalla 2024-07-01 a la(s) 19 45 01](https://github.com/juliasalto/proyectohipotesis/assets/136622322/b8e57490-31cb-40b9-a061-c7556f4df185)
     ![Captura de Pantalla 2024-07-01 a la(s) 19 45 14](https://github.com/juliasalto/proyectohipotesis/assets/136622322/3432c230-b2af-4f45-8a8f-996eb5dc96ed)
     ![Captura de Pantalla 2024-07-01 a la(s) 19 45 25](https://github.com/juliasalto/proyectohipotesis/assets/136622322/a409e2ff-c9e6-4550-b16a-26f58bbf289a)
     ![Captura de Pantalla 2024-07-01 a la(s) 19 41 40](https://github.com/juliasalto/proyectohipotesis/assets/136622322/958e7fdc-f395-4aea-bf2d-3d1925ef6340)
    ![Captura de Pantalla 2024-07-01 a la(s) 19 41 54](https://github.com/juliasalto/proyectohipotesis/assets/136622322/ac692c13-bbec-4d1c-bc7f-e1fd17510a88)
    ![Captura de Pantalla 2024-07-01 a la(s) 19 42 17](https://github.com/juliasalto/proyectohipotesis/assets/136622322/af431fee-e943-465c-a761-923628d4152c)
    ![Captura de Pantalla 2024-07-01 a la(s) 19 42 49](https://github.com/juliasalto/proyectohipotesis/assets/136622322/10114eb4-d64f-44d4-b6b4-3c56d8d92c8f)
     ![Captura de Pantalla 2024-07-01 a la(s) 19 43 06](https://github.com/juliasalto/proyectohipotesis/assets/136622322/05439cf1-4816-486c-a43c-7f7e19a1cd73)
     ![Captura de Pantalla 2024-07-01 a la(s) 19 43 25](https://github.com/juliasalto/proyectohipotesis/assets/136622322/f92eac50-d907-4a45-a164-1c22bc533f1b)
     ![Captura de Pantalla 2024-07-01 a la(s) 19 43 45](https://github.com/juliasalto/proyectohipotesis/assets/136622322/2a35945c-ba34-4709-b11d-c89fdb5cae27)

    - Test Mann Whitney U:

      ![Captura de Pantalla 2024-07-01 a la(s) 19 49 10](https://github.com/juliasalto/proyectohipotesis/assets/136622322/b29d4dc7-91e4-408d-96ca-f37ad0af7fd9)
     ![Captura de Pantalla 2024-07-01 a la(s) 19 49 23](https://github.com/juliasalto/proyectohipotesis/assets/136622322/144fd790-964f-43e6-a9db-cd12693ee0dd)

    - Regresión lineal:

      ![Captura de Pantalla 2024-07-01 a la(s) 20 00 53](https://github.com/juliasalto/proyectohipotesis/assets/136622322/42526acd-124e-461f-aaaa-fc07a91bf61b)
      ![Captura de Pantalla 2024-07-01 a la(s) 20 01 04](https://github.com/juliasalto/proyectohipotesis/assets/136622322/02d49b92-6400-4f3e-b99a-5f5516a8daa3)
      ![Captura de Pantalla 2024-07-01 a la(s) 20 01 23](https://github.com/juliasalto/proyectohipotesis/assets/136622322/b914e8ea-01b7-416e-8973-d2dfa0f1a717)
      ![Captura de Pantalla 2024-07-01 a la(s) 20 01 36](https://github.com/juliasalto/proyectohipotesis/assets/136622322/e5508ebf-b907-4471-8a13-4059c185fd65)
      ![Captura de Pantalla 2024-07-01 a la(s) 20 01 45](https://github.com/juliasalto/proyectohipotesis/assets/136622322/8e42407a-ac78-41dd-ad4b-e874edcd2215)
      ![Captura de Pantalla 2024-07-01 a la(s) 20 01 57](https://github.com/juliasalto/proyectohipotesis/assets/136622322/75c0027b-fd14-477d-8862-3f5f4fcb1b2b)
      ![Captura de Pantalla 2024-07-01 a la(s) 20 02 06](https://github.com/juliasalto/proyectohipotesis/assets/136622322/e644922d-f850-456c-a1d4-b2dd48f99f8e)
      ![Captura de Pantalla 2024-07-01 a la(s) 20 02 14](https://github.com/juliasalto/proyectohipotesis/assets/136622322/bc984ebd-e30d-4a8a-80c9-56cee315f9a7)
      ![Captura de Pantalla 2024-07-01 a la(s) 20 02 34](https://github.com/juliasalto/proyectohipotesis/assets/136622322/dd55958d-ceb9-467e-a10e-e62ba7ada265)
      ![Captura de Pantalla 2024-07-01 a la(s) 20 02 41](https://github.com/juliasalto/proyectohipotesis/assets/136622322/e8e9160b-bc7b-4497-8773-39a5097b783f)
      ![Captura de Pantalla 2024-07-01 a la(s) 20 02 50](https://github.com/juliasalto/proyectohipotesis/assets/136622322/26b08023-dfa8-4e80-98b4-60fcd28f04f1)
      ![Captura de Pantalla 2024-07-01 a la(s) 20 02 58](https://github.com/juliasalto/proyectohipotesis/assets/136622322/58c67671-c9d4-46b7-a744-ddac801c87c6)
      ![Captura de Pantalla 2024-07-01 a la(s) 20 03 07](https://github.com/juliasalto/proyectohipotesis/assets/136622322/9dc1cbc6-15d6-4421-8cb0-e43258bbe0d1)
      ![Captura de Pantalla 2024-07-01 a la(s) 20 03 18](https://github.com/juliasalto/proyectohipotesis/assets/136622322/e5bb6b66-908c-4e33-9c8b-73f43bc4ae8e)

  
# Conclusiones generales:
- Consideramos que al lanzar un track debería poder estar en la mayor cantidad de plataformas posible.
- Observamos, además, que los tracks más escuchados (60%) fueron de artistas solistas
- La escala major es la que predomina en toda la muestra de datos, por lo tanto sugerimos considerarla al momento de la composición de un track.
- Las características de la canción no tienen mayor influencia que la que podría llegar a tener el hecho de que los artistas que lideran las cantidades de reproducciones son conocidos/as a nivel mundial.
- Sobre las hipótesis: en general, los datos no reflejan una distribución normal, por lo que considero necesario revisar outliers, tamaño de la muestra, entre otros.
  
  
- **Limitaciones/Próximos Pasos:**
    - Explorar otros modelos a los que se puedan ajustar variables continuas que no tienen una distribución normal.
    - Profundizar en las nociones de hipótesis y pruebas de hipótesis. 
