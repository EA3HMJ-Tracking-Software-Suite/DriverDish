# Proposal: Local JPL Kernel Ephemeris Server

## Motivation

DriverDish currently obtains Sun, Moon, and other target ephemerides from the JPL Horizons API. This normally provides authoritative data without requiring DriverDish to maintain an astronomical calculation engine. However, an external service outage, a network failure, or an observing site without Internet access can stop ephemeris acquisition and therefore prevent tracking.

The proposed solution is a small local ephemeris server based on downloaded JPL SPICE kernels. It would preserve the current DriverDish workflow: instead of changing the tracking and parsing code, DriverDish would send the same type of request to either JPL Horizons or the local server. The local response would reproduce the subset of the Horizons text format that DriverDish already consumes, including the `$$SOE` and `$$EOE` data section markers.

## Goals

- Continue observing when the Horizons service is unavailable.
- Allow fully offline operation after the required kernels have been downloaded.
- Minimize changes to the existing DriverDish ephemeris and tracking logic.
- Use official JPL kernels and a well-tested SPICE implementation.
- Begin with the Sun (NAIF ID 10) and Moon (NAIF ID 301), then extend support to other targets.
- Make switching between remote and local data explicit and reversible.

## Proposed architecture

### Local service

The service can be implemented as a lightweight HTTP application running on the same computer or another computer on the local network. A Python implementation using SpiceyPy is a practical first option, although the HTTP contract should remain independent of the programming language.

The server would:

1. Load a configured set of JPL SPICE kernels at startup.
2. Accept the location, target, start time, stop time, and step size required by DriverDish.
3. Calculate topocentric azimuth, elevation, range, and radial velocity for each requested time.
4. Apply the same time scale, coordinate conventions, atmospheric/refraction policy, and units expected by DriverDish.
5. Return a Horizons-compatible text response containing only the fields used by the current parser.
6. Expose a health/status endpoint showing the loaded kernels and their validity periods.

### DriverDish integration

The Targets form would provide:

- a **Use local ephemeris server** checkbox;
- a configurable local server URL, for example `http://127.0.0.1:5000`;
- a connection/status indication;
- automatic validation that the requested date is covered by the installed kernels.

The selection would change only the base URL used by the ephemeris request. The existing request parameters, response parser, interpolation, and tracking pipeline should remain unchanged wherever possible.

An optional fallback mode may be considered later: try Horizons first and use the local server only after a timeout. The first implementation should use an explicit checkbox so the operator always knows which source is active.

## Kernel management

Kernels should not be committed to the application repository because they can be large and are updated independently. The server should provide a documented setup command or script that downloads verified kernels from an official JPL source and records:

- file name and source URL;
- download date;
- checksum;
- covered time interval;
- kernel version.

The first kernel set should include the planetary ephemeris, leap seconds, Earth orientation/frame data, and any additional kernels required for accurate topocentric Sun and Moon positions.

## Compatibility contract

Before replacing Horizons during tracking, captured Horizons responses should become reference fixtures. Automated tests should send the same requests to both sources and compare:

- timestamps and number of samples;
- azimuth and elevation within an agreed angular tolerance;
- range and radial velocity within agreed tolerances;
- parsing behavior and error responses;
- results near horizon crossings, UTC day boundaries, and daylight-saving changes in the user interface.

The local format only needs to support the Horizons fields and options currently used by DriverDish. Unsupported targets or parameters must return a clear error rather than silently producing approximate data.

## Suggested implementation phases

1. Document the exact Horizons requests and response fields currently used by DriverDish.
2. Build a prototype service for Sun and Moon with fixed test locations and dates.
3. Add comparison tests against saved Horizons responses.
4. Add configuration, health checks, and kernel coverage reporting.
5. Add the checkbox and local URL setting to the Targets form.
6. Test online, during a simulated Horizons outage, and on a computer with no Internet connection.
7. Package the server and kernel installer/updater for normal users.

## Open questions

- Which kernel set and coverage interval should be supplied by default?
- Should the service run inside DriverDish, as a Windows service, or as a separate application?
- What angular and velocity tolerances are acceptable for tracking and Doppler calculations?
- Should automatic Horizons-to-local fallback be added after the explicit selection mode is validated?
- How should kernel updates and checksum verification be presented to non-technical users?

This document proposes the interface and development direction. It does not yet add the ephemeris server or change DriverDish runtime behavior.

