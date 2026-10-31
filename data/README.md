# Dataset and processed table guide

## Raw inputs

Raw Sensor Logger files remain in their existing locations. `raw/<Activity>/` contains 20 recordings each for Still, Standing, Walking, and Jumping. Each has calibrated accelerometer, gyroscope, and metadata CSVs sharing a filename stem. The three mixed recordings are in `sequences/`, each with accelerometer and gyroscope CSVs. The notebooks do not modify these raw files.

Sensor CSV columns are `time`, `seconds_elapsed`, `z`, `y`, `x`. Elapsed seconds drive time alignment; axes are selected by column name. Metadata includes `version`, `device name`, `recording epoch time`, `recording time`, `recording timezone`, `platform`, `appVersion`, `device id`, `sensors`, `sampleRateMs`, `standardisation`, and `platform version`. Notebook 01 requires `device name`, `recording time`, and `sampleRateMs`.

## Quality audit and cleaning

Notebook 01 audits every labeled recording and each mixed sequence for both sensor streams. It reports rows, duration, median/minimum/maximum timestamp interval, effective median rate, missing values, duplicate timestamps, non-monotonic timestamps, large gaps (over 1.5 times the median interval), paired-rate difference, and common timestamp coverage. It checks for 20 recordings and at least 90 seconds in each activity.

Readings are sorted by elapsed time before alignment. Missing or non-numeric required values, duplicate timestamps, non-finite values, large gaps, missing files/columns, or paired rates differing by more than 2% stop processing with a clear error. Missing values are reported and not imputed. The supplied data have no missing values, duplicate or non-monotonic timestamps, large gaps, or removed rows. Measured per-file median rates span about 99.988-100.020 Hz.

## Alignment, windows, and features

Accelerometer and gyroscope streams are linearly interpolated by elapsed time over their common overlap onto a shared grid at the lower measured median rate. Both sources remain present in the cleaned outputs. Frequency features use the measured target rate for each recording.

A 1.0-second window contains about 100 samples near 100 Hz and gives approximately 1 Hz periodogram resolution. The 0.5-second hop advances about 50 samples and creates 50% overlap. Mixed-sequence windows are retained only when the full window lies within an estimated stable interval, so windows do not cross unlabeled transitions.

The 30 features retain separate `acc_` and `gyro_` names: per-axis mean, standard deviation, RMS; magnitude mean, standard deviation, RMS; dominant frequency; spectral energy; and spectral entropy. Magnitudes reduce orientation sensitivity. Means/RMS/standard deviations describe posture and intensity; dominant frequency captures cadence; energy and entropy help distinguish periodic from irregular motion.

## Processed outputs from notebook 01

- `processed/cleaned_sensor_readings.csv`: sorted, common-grid sensor axes plus recording ID, source, sampling rate, partition, and estimated activity when available.
- `processed/quality_audit.csv`: per-recording audit with paired sensor statistics.
- `processed/recording_split.csv`: whole-recording `train`, `test`, or `qualitative_only` partition. The final test has two recordings per activity; the third mixed sequence is qualitative only.
- `processed/window_features_raw.csv`: 30 unscaled features plus recording ID, activity/estimated activity, source, partition, rate, and window start/end/centre.
- `processed/window_features_scaled.csv`: same rows after training-only normalization; notebook 02 reads this directly.
- `processed/scaler_parameters.csv`: feature means/scales and count of training windows used to fit them. Test recordings are transformed only after fitting.

Mixed-sequence intervals are approximate. They provide estimated transition examples and qualitative review, not ground-truth labels.
