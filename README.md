# 🐝 Beehive Health Monitoring with IoT  

## 📌 Overview  
This project was developed in collaboration with Burgos & Howest university to measure and analyze the health of beehives using IoT sensor data. The focus is on applying time-series analysis and regression techniques to better understand hive conditions and detect potential anomalies.

Disclaimer:
The data used in this project originates from Appalachain state university and was permitted to use on this project on request I can therefore not publish it publicly.

## 🎯 Objectives  
- Clean and preprocess IoT sensor data from beehives.  
- Explore time-series characteristics (autocorrelation, seasonal trends).  
- Build regression models to forecast and explain hive conditions.  

## 🛠️ Tech Stack  
- **Python**: Data processing and modeling  
- **Pandas, NumPy**: Data handling and transformation  
- **Matplotlib, Seaborn**: Visualization  
- **scikit-learn**: Machine learning (regression, preprocessing)  
- **statsmodels**: Time-series autocorrelation analysis  
- **Jupyter Notebook**: Experimentation and reporting  

## 🤖 Machine Learning & Data Analysis  
### Preprocessing  
- Feature scaling with **MinMaxScaler**  
- Train-test splitting with **train_test_split**  

### Models  
- **Linear Regression** for baseline predictions  
- **Polynomial Regression** to capture non-linear patterns  
- **Ridge Regression** for regularized linear modeling  
- **Lasso Regression** for feature selection and sparsity  

### Time-Series Analysis  
- **Autocorrelation plots (ACF)** to inspect temporal dependencies  

## 📊 Example Insights  
- Polynomial regression can capture non-linear fluctuations in hive data.  
- Regularization (Ridge/Lasso) helps stabilize predictions against noisy sensor inputs.  
- Autocorrelation analysis reveals whether hive conditions depend strongly on past values.  
