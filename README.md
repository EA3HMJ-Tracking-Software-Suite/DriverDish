# DriverDish

DriverDish is the main Windows application of the [EA3HMJ Tracking Software Suite](https://github.com/EA3HMJ-Tracking-Software-Suite). It calculates target ephemerides, generates pointing commands, communicates with [ControllerDish](https://github.com/EA3HMJ-Tracking-Software-Suite/ControllerDish), and provides the tools required to operate and evaluate a high-precision two-axis antenna tracking system.

It is intended for amateur Earth-Moon-Earth communication (EME), radio astronomy, amateur Deep Space Network (DSN), and other space communication applications.

## AI technical support / Soporte tecnico con IA

> [!TIP]
> [Ask about DriverDish ->](https://ea3hmj-support-assistant.qlfecv.chatgpt.site/?product=DriverDish) / [Pregunta sobre DriverDish ->](https://ea3hmj-support-assistant.qlfecv.chatgpt.site/?product=DriverDish)

## Issues and contributions

Because of the complexity of the project and its wide range of possible hardware, software, and observing configurations, GitHub Issues and Pull Requests are used to manage DriverDish development more effectively.

- Use [Issues](https://github.com/EA3HMJ-Tracking-Software-Suite/DriverDish/issues) to report one bug or request one feature at a time. Include the DriverDish and Windows versions, hardware and SDR configuration, reproduction steps, logs, and screenshots when available.
- Use [Pull Requests](https://github.com/EA3HMJ-Tracking-Software-Suite/DriverDish/pulls) for focused code or documentation changes. Explain the reason for the change, link the related Issue, and describe how it was tested.
- Read the complete [contribution guide](CONTRIBUTING.md) before submitting a report or change.
- Review the proposed [local JPL kernel ephemeris server](docs/LOCAL_EPHEMERIS_SERVER.md), designed to provide a Horizons-compatible fallback and offline ephemerides.

## Current version: 3.3

DriverDish 3.3 adds the Q37 two-dimensional pointing-correction model, SkyDip measurements, improved signal-search and weather logging, while retaining integrated ephemerides, antenna control, pass planning, RA/Dec tracking, pointing calibration, and antenna-performance analysis.

[Download DriverDish 3.3](https://github.com/EA3HMJ-Tracking-Software-Suite/DriverDish/releases/tag/3.3)

## ControllerDish compatibility

> [!IMPORTANT]
> DriverDish 3.1 and later require **ControllerDish firmware 4.x or later**.

ControllerDish firmware 4.x introduced the Modbus protocol used for communication between DriverDish and ControllerDish. The standard serial connection uses **Modbus RTU over RS-485**.

Earlier ControllerDish firmware versions use the previous communication protocol and are not compatible with current DriverDish versions. Update ControllerDish with the firmware package that matches its hardware family before connecting it to DriverDish:

- ControllerDish hardware 1.x uses the `Hv1` firmware package.
- ControllerDish hardware 2.x uses the `Hv2` firmware package.

See the [ControllerDish releases](https://github.com/EA3HMJ-Tracking-Software-Suite/ControllerDish/releases/latest) for the current compatible firmware packages.

## Quick start

1. Install the latest DriverDish release and the matching ControllerDish firmware.
2. Open **Setup**, select and configure the dish-controller driver, then connect it.
3. Open **Targets**, enter the station latitude, longitude, and height, and refresh the ephemerides.
4. Select a target on the main screen.
5. Configure **Radios** and **SNR** when Doppler control or signal-based measurements are required.
6. Confirm that the calculated elevation is above the horizon and that the movement path is safe.
7. Select the desired correction mode and click **Track to Target**.

> [!WARNING]
> Automatic correction, Heatmap, Drift Scan, SkyDip, and ON/OFF observations move the antenna automatically. Verify mechanical limits, clearance, emergency-stop access, and valid position feedback before starting them.

## Main features

### Q37 pointing model and correction workflows - new in 3.3

- Keeps the existing simple one-dimensional correction workflow based on `corrections.txt`.
- Adds an advanced two-dimensional Q37 pointing model based on the 37 constants stored in `AzimuthCorrectionQ.txt`.
- Supports automatic signal-maximum searches at configurable azimuth intervals.
- Records correction measurements for later analysis and model fitting.

### SkyDip and environmental logging - new in 3.3

- Performs configurable elevation scans and records azimuth, elevation, and signal level to CSV.
- Includes direct access to the external Sky Profile Analysis tool for interpreting SkyDip data.
- Writes pressure, temperature, and relative humidity to the `wxlog` directory every ten minutes.
- Uses valid local environmental sensors first and falls back to recent Open-Meteo data when local sensors are unavailable.

### RA/Dec fixed-source tracking

- Tracks fixed celestial and radio sources using right ascension and declination coordinates.
- Accepts RA/Dec coordinates in space-separated, colon-separated, HMS/DMS, or decimal-degree formats.
- Validates coordinate format and range before tracking begins.
- Allows the active RA/Dec source to be changed without restarting the tracking engine.
- Loads source catalogs from `radec.txt`; the supplied catalog includes 51 bright stars with Hipparcos coordinates.
- Applies precession, nutation, annual aberration, the equation of the equinoxes, and atmospheric refraction corrections.
- Provides astrometric precision better than 0.001 degrees under normal operating conditions.
- Maintains a two-hour trajectory buffer at 0.5-second resolution and reloads it automatically in the background.
- Retrieves local pressure, temperature, and relative humidity from Open-Meteo for atmospheric-refraction correction; no API key is required.

### Integrated ephemeris calculation

- Calculates target coordinates directly inside DriverDish.
- Eliminates the need to run a separate ephemeris server.
- Continuously updates azimuth and elevation commands during tracking.

### Antenna control and monitoring

- Sends target positions and movement commands to ControllerDish.
- Displays actual azimuth and elevation feedback from the antenna sensors.
- Provides manual movement, automatic tracking, stop, and positioning controls.
- Supports pointing offsets and system-status monitoring.
- Uses the feedback provided by ControllerDish for closed-loop tracking.

### Pass planning and tracking

- Displays upcoming target passes and their relevant tracking information.
- Provides a dedicated pass window for preparing and following a complete pass.
- Allows the operator to select a target and monitor its movement throughout the available tracking period.

![Target pass tracking](https://github.com/user-attachments/assets/6167fed9-f845-4470-a70f-0cc7c95d2baf)

### Automatic pointing correction

- Uses received signal-level measurements to determine the best pointing position.
- Performs automatic scans around the predicted target position.
- Calculates and applies the best azimuth pointing offset found during the scan.
- Supports solar and lunar calibration workflows.
- Uses SpectraVue as the signal source for the current automatic-correction workflow.

![Automatic pointing correction](https://github.com/user-attachments/assets/d8460a52-5b40-4bd6-bf35-83b27513417d)

### Drift Scan and HPBW measurement

- Records signal level while the target drifts through the antenna beam.
- Calculates the antenna's measured half-power beamwidth (HPBW).
- Compares measured and theoretical HPBW.
- Provides graphical analysis and CSV data handling.
- Includes signal-analysis results such as peak position and Y-factor.

![Real HPBW measurement](https://github.com/user-attachments/assets/06a16067-a27e-4440-b160-ecd08380a0e3)

### Pointing analysis tools

- Characterizes systematic azimuth pointing error.
- Calculates correction data from measurements taken at different positions.
- Provides heatmap and signal-mapping tools for evaluating antenna response.
- Supports offset optimization based on the maximum received signal.

### Radio and Doppler control

- Provides a dedicated **Radios** tab for controlling compatible transceivers.
- Uses **OmniRig** for CAT communication with supported radios.
- Calculates Doppler-corrected frequency from the target's radial velocity.
- Allows the tracking and radio-control workflow to be managed from the same application.

## External integrations

### Radio control

DriverDish uses **OmniRig** from the Radios tab. OmniRig must be installed and the required radio must be configured correctly before CAT control can be used.

### Signal-level sources

DriverDish can use compatible external signal-analysis software as the measurement source for automatic correction, Drift Scan, and related tools:

- **SpectraVue**
- **SigDigger**
- **External radiometer input**

The selected application must be running and correctly configured before starting an automated signal-based measurement.

## Documentation

The following documents describe installation, configuration, and specialist workflows. Some documents were written for earlier DriverDish versions but remain useful as complementary reference material.

- [DriverDish v1 - English](https://github.com/EA3HMJ-Tracking-Software-Suite/.github/blob/main/DriverDish%20v1%20ENG.pdf)
- [DriverDish v1 - Spanish](https://github.com/EA3HMJ-Tracking-Software-Suite/.github/blob/main/DriverDish%20v1%20ESP.pdf)
- [DriverDish configuration and use v1 - Spanish](https://github.com/EA3HMJ-Tracking-Software-Suite/.github/blob/main/Guia%20DriverDish.App%20v1%20ESP.pdf)
- [Getting Started Guide v2 by EA4LE - Spanish](https://github.com/EA3HMJ-Tracking-Software-Suite/.github/blob/main/GettingStartedGuide%20v2%20ea4le%20ESP.pdf)
- [Automatic correction - English](https://github.com/EA3HMJ-Tracking-Software-Suite/.github/blob/main/DriverDish.App%20Autocorrection%20V1.0%20ENG.pdf)
- [Automatic correction - Spanish](https://github.com/EA3HMJ-Tracking-Software-Suite/.github/blob/main/DriverDish.App%20Autocorrection%20V1.0%20ESP.pdf)
- [Drift Scan - English](https://github.com/EA3HMJ-Tracking-Software-Suite/.github/blob/main/DriverDish.App%20DriftScan%20V1.0%20ENG.pdf)
- [Drift Scan - Spanish](https://github.com/EA3HMJ-Tracking-Software-Suite/.github/blob/main/DriverDish.App%20DriftScan%20V1.0%20ESP.pdf)

## Releases

- [Latest release](https://github.com/EA3HMJ-Tracking-Software-Suite/DriverDish/releases/latest)
- [All DriverDish releases](https://github.com/EA3HMJ-Tracking-Software-Suite/DriverDish/releases)

### Major version history

| Version | Main changes |
| --- | --- |
| **1.x** | Original DriverDish application and external ephemeris-server workflow |
| **2.0** | Complete application rewrite and new ControllerDish safety and communication architecture |
| **2.2** | Improved pointing-offset configuration |
| **3.0** | Integrated ephemeris engine and automatic Sun/Moon pointing calibration |
| **3.1** | Target-pass window, azimuth-error analysis, Drift Scan, real HPBW measurement, and general optimizations |
| **3.2** | RA/Dec fixed-source tracking, flexible coordinate input, source catalogs, high-precision astrometric corrections, and weather-based refraction correction |
| **3.3** | Q37 two-dimensional pointing model, SkyDip measurements, azimuth signal search, WX logging with Open-Meteo fallback, and reliability fixes |

## Experience required

DriverDish is part of an antenna tracking system that combines motor control, high-current electronics, position sensors, mechanical equipment, and radio-frequency measurements. It is intended for experienced users familiar with antenna tracking, electronics, and software configuration.

## Disclaimer

DriverDish and all related materials are provided **as is**, without any express or implied warranty. The author cannot provide individual support and does not guarantee the accuracy, completeness, suitability, or continued availability of the software.

The author shall not be liable for equipment damage, data loss, personal injury, pointing errors, or any other consequences arising from the installation or use of DriverDish or the associated tracking hardware.
