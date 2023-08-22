# Milano PV Power forecasting

## Description
We have data on PV Power production from the Politecnico di Milano.
Our task is to make forecasts 3 hours ahead in time.

The following features are provided in the [dataset](https://ieee-dataport.org/open-access/photovoltaic-power-and-weather-parameters).
In particular, the cleaned dataset is composed of the following variables and specifics, with a time resolution of 1 hour:

- Time: column with time recordings; the data format is “dd-MM-yyyy hh:mm:ss”, with the time always expressed in Central European Time (CET).
- PV_Power: power recordings from the PV module (W); module tilt: 30°.
- T_air: ambient temperature (°C) measured by the weather station described in SolarTech Lab website (http://www.solartech.polimi.it/instrumentation/).
- G_h: measured Global Horizontal Irradiance (W/m2).
- G_tilt: global irradiance measured on the plane of array (30°).
- W_s: measured wind speed (m/s).
- W_d: measured wind direction (°), assuming 0° east, positive south.

Our approach is to build a multistep multivariate LSTM model for forecasting.
## Files

- data_prep.ipynb

Data preperation and cleaning.
- eda.ipynb

Exploratory data analysis
- models.ipynb

This is where functions for training the model are defined. Also contains the
example usage (forecasting the PV power production 3 hours ahead). This should be ran inside of Google Colab because of the long training time on normal machines. To setup inside Google Colab we must create a directory called 'pv' inside our Google Drive and place the 'cleaned_data.csv' inside it. Otherwise we have to modify the path variable in the second code block.

- model_lstm

The best model is saved in this directory, so it can later be loaded and used for forecasting without having to re-train the model. 


## Results
Currently the best model for predicting 3 hours ahead has the following error metrics:
(Stacked LSTM 128 -> 64 units, relus)

t+1 RMSE: 7.096446  MAE:  2.821945  MAPE: 1.281153

t+2 RMSE: 19.651424 MAE:  9.410577  MAPE: 1.586897

t+3 RMSE: 25.725664 MAE:  13.800363 MAPE: 3.584065

(RMSE - root mean squared error, MAE - mean absolute error, MAPE - mean absolute percentage error)


## Next steps
### Experiment with Encoder-Decoder LSTM 
#### Univariate approach
We can  update the LSTM to use an encoder-decoder model. This means that the model will not ouptut a vector sequence directly. Instead, the model will be comprised of two sub models, the encoder to read and encode the input sequence, and the decoder that will read the encoded input sequence and make a one-step prediction for each element in the output sequence. The difference is subtle, as in practice both approaches do in fact predict a sequence output. The important diffrence is that an LSTM model is used in the decoder, allowing it to both know what was predicted for the prior timestep in the sequence and accumulate internal state while outputting the sequence.

#### Multivariate approach
We can update the Univariate Encoder-Decoder LSTM by providing each one-dimensional time series to the model as a separate sequence of input. The LSTM will in turn create an internal representation of each input sequence that will together be interpreted by the decoder.

### Experiment with CNN-LSTM Encoder-Decoder 
A CNN, can be used as the encoder in an encoder-decoder architecture. The CNN does not directly support sequence input; instead, a 1D CNN is capable of reading across sequence input and automatically learning the salient features. These can then be interpreted by an LSTM decoder as per normal. The CNN expects the input data to have the same 3D structure as the LSTM model, although multiple features are read as different channels that ultimately have the same effect.

### Experiment with ConvLSTM Encoder-Decoder
A further extension of the CNN-LSTM approach is to preform the convolutions of the CNN as part of the LSTM for each timestep. This combination is also used for spatiotemporal data. Unlike an LSTM that reads the data in directly in order to calculate internal state and state transitions, and unlike the CNN-LSTM that is interpreting the output from CNN models, the ConvLSTM is using convulutions directly as part of reading input into the LSTM units themselves. Keras provides the ConvLSTM2D class that supports the ConvLSTM model for 2D data. It can be configured for 1D multivariate time series forecasting. ConvLSTM2D by default expects input data to have the shape: [samples, timesteps, rows, cols, channels]. Where each time step of data is defined as an image of (rows * colums) data points.

