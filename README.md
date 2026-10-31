# Human Activity Recognition with a Hidden Markov Model

Individual smartphone-sensor project for Still, Standing, Walking, and Jumping. It uses accelerometer and gyroscope recordings from one participant and an iPhone XR. Run the two notebooks in order; the second consumes processed outputs from the first.

## Project structure

```text
human_activity_hmm_github/
|-- data/
|   |-- raw/<Activity>/       # raw single-activity accelerometer, gyroscope, metadata CSVs
|   |-- sequences/            # raw mixed-sequence accelerometer/gyroscope CSVs
|   |-- processed/            # aligned rows, audit, split, features, scaler parameters
|   `-- README.md
|-- figures/                   # generated preprocessing and HMM visualizations
|-- notebooks/
|   |-- 01_data_preprocessing.ipynb
|   `-- 02_hmm_activity_recognition.ipynb
|-- requirements.txt
`-- README.md
```

## Run in order

1. Install dependencies with `pip install -r requirements.txt`.
2. Run `notebooks/01_data_preprocessing.ipynb`. It audits the raw data, aligns accelerometer and gyroscope readings by elapsed time, creates windows and 30 paired-source features, assigns complete-recording splits, fits the scaler on training recordings only, and writes outputs under `data/processed/`.
3. Run `notebooks/02_hmm_activity_recognition.ipynb`. It loads the scaled features and split metadata, fits and decodes the four-state Gaussian HMM, then evaluates once on the held-out recordings. It does not read raw CSVs, repeat feature extraction, or fit a scaler.

## Processed outputs

- `cleaned_sensor_readings.csv`: sorted, time-aligned accelerometer and gyroscope axes on a common measured-rate grid.
- `quality_audit.csv`: per-recording audit for both sensor streams.
- `recording_split.csv`: train, test, and qualitative-only recording assignments.
- `window_features_raw.csv` and `window_features_scaled.csv`: window metadata and 30 features with `acc_` and `gyro_` prefixes.
- `scaler_parameters.csv`: means and scales fitted on training windows only.

The final test contains exactly two complete recordings per activity (eight total). Test recordings do not contribute to normalization or HMM fitting. The current held-out run classified 8/8 recordings correctly; the small single-person, single-device sample does not establish cross-person or cross-device performance. Mixed-sequence activity boundaries are estimates, not ground truth.
