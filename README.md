# 🌍 Comparing Lake-Breeze Simulations between WRF and Pangu-Weather / Comparaison des Simulations de Brise de Lac

**Language | Langue**: 🇺🇸 [English](#-english-version) | 🇫🇷 [Français](#-version-française)

---

## 🇺🇸 English Version

Author: [Piyush Teeloku](https://www.linkedin.com/in/piyush-teeloku/)  
Email: teelokup@gmail.com  
GitHub: [@Piyush-T31](https://github.com/Piyush-T31)

---

## 📄 Overview

This repository presents a comparative study between two weather prediction models—**WRF (Weather Research and Forecasting)** and **Pangu-Weather**, an advanced deep learning model—focused on simulating lake-breeze effects and their influence on convective storms over Southern Ontario.

---

## 📌 Objective

To assess how lake-breeze circulations impact convective storm development, and to evaluate the strengths and limitations of traditional NWP (WRF) and DLWP (Pangu-Weather) models in simulating these mesoscale phenomena.

---

## 🌊 Case Study

A storm event on **May 20, 2024** was analyzed:

- Storm tracks were evaluated using ECCC radar data.
- Specific lake-breeze effects around **Lake Huron**, **Lake Erie**, **Lake Ontario**, and **Lake St. Clair** were examined.

![_Figure: Radar snapshots from 6 p.m. to 12 a.m. UTC (May 20, 2024)_](radar/radar.gif)

📍 _Figure: Radar snapshots from 6 p.m. to 12 a.m. UTC (May 20, 2024)_

---

## 🛰️ Methodology

### WRF Model Setup

- **3 nested domains** (1.0 km resolution for innermost domain)
- **Input:** ERA5 reanalysis data
- **Simulation Period:** May 20, 00:00 UTC to May 21, 06:00 UTC
- **Spin-up Time:** 12 hours
- **Output Interval:** 30 minutes

![WRF nested domain](High_res/wps_show_dom.png)
📍 _Figure: WRF nested domain setup_  
![WRF T2 ws10](era5_init/t2_ws10/t2_ws10wrf.gif)
📍 _Figure: WRF surface temperature at 2m and wind field plots at 10 m_  
![WRF w500 ws500](era5_init/W_500/w500.gif)
📍 _Figure: Vertical wind plot at 500 m with wind field plots at 500 m_

### Pangu-Weather Model

- DL model using **3D Earth-Specific Transformer (3DEST)**
- Trained on 43 years of ERA5 data
- Uses hierarchical temporal aggregation (1h, 3h, 6h, 24h models)
- Inference performed using ONNX models on both CPU and GPU

![pangu transformer](pangu_data/3dest.png)
📍 _Figure: Diagram of the 3DEST architecture_

### Inference Pipeline (Python)

```python
# Run the inference session
# Initialize inputs for all models
hourly_upper_outputs = []
hourly_surface_outputs = []
input_6 , input_surface_6 = input , input_surface
input_1, input_surface_1 = input , input_surface
  for i in range (24): # 1- day forecast with hourly timesteps
      if (i + 1) \% 6 == 0: # Every 6 hours run the 6- hour model
       output , output_surface = ort_session_6 . run(None , {’input ’: input_6 , ’
       input_surface ’: input_surface_6 })
       input_6 , input_surface_6 = output , output_surface
      else: # Otherwise , run the 1- hour model
       output , output_surface = ort_session_1 . run(None , {’input ’: input , ’input_surface ’: input_surface})
# Aggregate outputs
   input , input_surface = output , output_surface # Use the 1 - hour model output as the base
# Append to lists
hourly_upper_outputs . append ( output )
hourly_surface_outputs . append ( output_surface )
# Convert lists to arrays
hourly_upper_outputs = np. array ( hourly_upper_outputs ) # Shape : (24 , ...)
hourly_surface_outputs = np. array ( hourly_surface_outputs ) # Shape : (24 , ...)
# Save the results
np. save (os. path . join ( output_data_dir , ’ hourly_upper_outputs . npy ’),
hourly_upper_outputs )
np. save (os. path . join ( output_data_dir , ’ hourly_surface_outputs .npy ’),
hourly_surface_outputs )
```

---

## 📊 Results

**Variable Validation**

- Pangu-Weather’s temperature and wind forecasts were visually and statistically close to ERA5 reanalysis
- Mean Absolute Error (MAE) and Root Mean Square Error (RMSE) were used for evaluation

![t2 mae gif](pangu_data/t2_pangu.png)
📍 _Figure: Global temperature at 2m - Pangu vs ERA5 at the end of simulation (H-24)_

![t2 mae gif](pangu_data/mae_plot/mae_t2m.gif)
📍 _Figure: Hourly MAE for global temperature at 2m_

**Local Station Comparison**

- Data from London Weather Station (43.03°N, 81.15°W and 278 m elevation) was used
- Compared T2 and WS10 from observations vs WRF and Pangu-Weather
- WRF-GFS matched afternoon temperatures best; Pangu underestimated wind peaks

|       Wind speed at 10m       |     Temperature at 2m     |
| :---------------------------: | :-----------------------: |
| ![ws10 london](comp_ws10.png) | ![t2 london](comp_t2.png) |

📍 _Figure: Wind speed at 10m and temperature at 2m plots compared to London station data_

![rmse](rmse_comp.png)
📍 _Figure: RMSE plots comparing Pangu-Weather station and WRF to London Weather station (T2 and WS10) and ERA5 data (U, V and T at 925 hPA)_

**Lake-Breeze Structure Comparison**

- Both models captured lake breeze onset (~16:00 UTC), peaking around 19:00 UTC
- WRF showed finer lake breeze effects due to higher spatial resolution
- Pangu-Weather showed strong convergence zones consistent with radar, despite lower resolution

|           T at 2m with 10m winds (WRF)           |       T at 2m with 10m winds (Pangu)        |
| :----------------------------------------------: | :-----------------------------------------: |
| ![ws10 london](era5_init/t2_ws10/t2_ws10wrf.gif) | ![t2 london](pangu_data/t2m_w10/t2_w10.gif) |

📍 _Figure: T2 and wind at 10 m over Southern Ontario (WRF vs Pangu)_

![div 500 london](pangu_data/divergence_plots/div925.gif)

📍 _Figure: Divergence for Pangu-Weather over Southern Ontario at 925hPA (~500 m)_

## ✅ Conclusion

- **Pangu-Weather** is a promising, cost-effective DLWP model with competitive accuracy compared to WRF.

- WRF, though computationally expensive, provides higher-resolution and more physically constrained simulations.

- The lake-breeze did contribute to storm intensification, and both models demonstrated this to varying extents.

- Key takeaways include the importance of initial conditions, physical parameterizations, and spatial resolution.

## 🇫🇷 Version Française

# 🌀 Comparaison des Simulations de Brise de Lac entre WRF et Pangu-Weather

Auteur : [Piyush Teeloku](https://www.linkedin.com/in/piyush-teeloku/)  
Email : teelokup@gmail.com  
GitHub : [@Piyush-T31](https://github.com/Piyush-T31)

---

## 📄 Aperçu

Ce dépôt présente une étude comparative entre deux modèles de prévision météorologique — **WRF (Weather Research and Forecasting)** et **Pangu-Weather**, un modèle avancé d’apprentissage profond — axée sur la simulation des effets de brise de lac et leur influence sur les orages convectifs en Ontario méridional.

---

## 📌 Objectif

Évaluer comment les circulations de brise de lac influencent le développement des orages convectifs, et analyser les forces et limites des modèles NWP (WRF) traditionnels et DLWP (Pangu-Weather) pour simuler ces phénomènes à méso-échelle.

---

## 🌊 Étude de Cas

L’événement orageux du **20 mai 2024** a été analysé :

- Les trajectoires des orages ont été étudiées à l’aide des radars d’ECCC.
- Les effets spécifiques de brise de lac autour des **lacs Huron**, **Érié**, **Ontario** et **St. Clair** ont été examinés.

![Images radar](radar/radar.gif)

📍 _Figure : Images radar de 6 p.m à 12 a.m UTC (20 mai 2024)_

---

## 🛰️ Méthodologie

### Configuration WRF

- **3 domaines imbriqués** (résolution de 0.5 km pour le plus interne)
- **Données d’entrée :** GFS Final Analysis et ERA5
- **Période de simulation :** 20 mai 00h00 UTC au 21 mai 06h00 UTC
- **Spin-up :** 12 heures
- **Sortie toutes les 30 minutes**

![WRF nested domain](High_res/wps_show_dom.png)
📍 _Figure : Configuration des domaines WRF_
![WRF T2 ws10](era5_init/t2_ws10/t2_ws10wrf.gif)
📍 _Figure : Température de surface WRF à 2 m et champs de vent à 10 m_
![WRF w500 ws500](era5_init/W_500/w500.gif)
📍 _Figure : Vent vertical à 500 m avec champs de vent à 500 m_

### Modèle Pangu-Weather

- Modèle IA basé sur le **transformer spécifique à la Terre 3D (3DEST)**
- Entraîné sur 43 ans de données ERA5
- Utilise une agrégation temporelle hiérarchique (modèles de 1h, 3h, 6h, 24h)
- Inférence réalisée avec ONNX sur CPU et GPU
  ![3dest](pangu_data/3dest.png)
  📍 _Figure : Schéma du modèle 3DEST_

### Script d’Inférence (Python)

```python
# Run the inference session
# Initialize inputs for all models
hourly_upper_outputs = []
hourly_surface_outputs = []
input_6 , input_surface_6 = input , input_surface
input_1, input_surface_1 = input , input_surface
  for i in range (24): # 1- day forecast with hourly timesteps
      if (i + 1) \% 6 == 0: # Every 6 hours run the 6- hour model
       output , output_surface = ort_session_6 . run(None , {’input ’: input_6 , ’
       input_surface ’: input_surface_6 })
       input_6 , input_surface_6 = output , output_surface
      else: # Otherwise , run the 1- hour model
       output , output_surface = ort_session_1 . run(None , {’input ’: input , ’input_surface ’: input_surface})
# Aggregate outputs
   input , input_surface = output , output_surface # Use the 1 - hour model output as the base
# Append to lists
hourly_upper_outputs . append ( output )
hourly_surface_outputs . append ( output_surface )
# Convert lists to arrays
hourly_upper_outputs = np. array ( hourly_upper_outputs ) # Shape : (24 , ...)
hourly_surface_outputs = np. array ( hourly_surface_outputs ) # Shape : (24 , ...)
# Save the results
np. save (os. path . join ( output_data_dir , ’ hourly_upper_outputs . npy ’),
hourly_upper_outputs )
np. save (os. path . join ( output_data_dir , ’ hourly_surface_outputs .npy ’),
hourly_surface_outputs )
```

## 📊 Résultats

**Évaluation des variables**

- Les prévisions de température et de vent de Pangu-Weather sont proches des données ERA5
- MAE et RMSE utilisés comme métriques d’évaluation

![t2 globale](pangu_data/t2_pangu.png)
📍 _Figure : Température globale à 2 m – Pangu vs ERA5 à la fin de la simulation (H-24)_

![mae t2](pangu_data/mae_plot/mae_t2m.gif)
📍 _Figure : MAE horaire pour la température globale à 2 m_

**Comparaison avec la Station de London**

- Données de la station météo de Londres (43.03°N, 81.15°O avec une élevation de 278 m)
- Comparaison des T2 et WS10 avec les modèles WRF et Pangu
- WRF-GFS reproduit mieux les températures diurnes ; Pangu sous-estime les pics de vent

|    Vitesse du vent à 10 m     |     Température à 2 m     |
| :---------------------------: | :-----------------------: |
| ![ws10 london](comp_ws10.png) | ![t2 london](comp_t2.png) |

📍 _Figure : Comparaison des courbes de la vitesse du vent à 10 m et de la température à 2 m avec les données de la station de London_

![rmse](rmse_comp.png)  
📍 _Figure : Courbes du RMSE comparant Pangu-Weather et WRF à la station météo de Londres (T2 et WS10) et aux données ERA5 (U, V et T à 925 hPa)_

**Structure de la Brise de Lac**

- Les deux modèles capturent l’apparition (~16h00 UTC) et le pic (~19h00 UTC) de la brise
- WRF montre des effets plus fins grâce à sa résolution
- Pangu capte les zones de convergence en accord avec les observations radar

|     Température à 2 m avec vent à 10 m (WRF)     | Température à 2 m avec vent à 10 m (Pangu)  |
| :----------------------------------------------: | :-----------------------------------------: |
| ![ws10 london](era5_init/t2_ws10/t2_ws10wrf.gif) | ![t2 london](pangu_data/t2m_w10/t2_w10.gif) |

📍 _Figure : Température à 2 m et vent à 10 m sur le sud de l’Ontario (WRF vs Pangu)_

![div 500 london](pangu_data/divergence_plots/div925.gif)

📍 _Figure : Divergence pour Pangu-Weather sur le sud de l’Ontario à 925 hPa (~500 m)_

## ✅ Conclusion

- **Pangu-Weather** est un modèle DLWP prometteur et efficace, avec une précision compétitive face à WRF.

- WRF fournit des simulations plus fines mais coûteuses en ressources.

- La brise de lac a contribué à l’intensification de l’orage ; les deux modèles l’ont représentée de manière cohérente.

- Le choix des conditions initiales, des schémas physiques et des résolutions est crucial pour simuler la méso-échelle.

## 📚 References

- WRF v4.1: Skamarock et al. (2019)

- Pangu-Weather: Bi et al. (2022, 2023)

- ERA5 Data: Hersbach et al. (2020)

- Environment and Climate Change Canada (ECCC) Radar & Weather Station Data

## 📁 Files Included

- Full report: [Research report](https://drive.google.com/file/d/18WIzc1xXRRxHLh7H8w5d9mEJgL81MVek/view?usp=sharing)
- Scripts: [Python code to run Pangu-Weather](pangu.py)

## 🧠 Future Work

- Explore other DL models like GraphCast

- Extend evaluation to more storm cases

- Improve microphysics schemes in WRF

- Incorporate real-time forecasting setups

## 🔗 Links

- [Pangu-Weather GitHub (Huawei)](https://github.com/198808xc/Pangu-Weather)

- [ERA5 Copernicus Data Portal](https://www.ecmwf.int/en/forecasts/dataset/ecmwf-reanalysis-v5)

- [ECCC Radar Portal](https://climate.weather.gc.ca/radar/index_e.html)

This work was conducted as part of a research initiative focused on operational mesoscale weather forecasting and deep learning applications in atmospheric science.
