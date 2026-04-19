# Augment LOOP packets with almanac data

This extension includes a configurable WeeWX service that can add live almanac 
data to the LOOP packets.

## Contents

* [What is that service for?](#what-is-that-service-for)
* [Configuration instructions](#configration-instructions)
* [Observation types](#observation-types)
* [Add observation types to the WeeWX database](#add-observation-types-to-the-weewx-database)
* [Usage](#usage)
  * [Live updates](#live-updates)
  * [Diagrams](#diagrams)
  * [Mobile stations](#mobile-stations)
* [FAQ](#faq)
* [Links](#links)

## What is that service for?

Some skins do not only update the web pages once every archive interval but 
present live data out of LOOP packets. The live data service of this extension 
can add fast changing almanac values to the LOOP packet for live updates. 
To use them include them in MQTT output. 

## Configuration instructions

The configuration of the live data service is part of the general
configuration of the 
[weewx-skyfield-almanac extension](https://github.com/roe-dl/weewx-skyfield-almanac).
Here, the specific keys are explained only. For a complete description
see section 
[Configuration instructions](https://github.com/roe-dl/weewx-skyfield-almanac#configuration-instructions)
in the README file.

```
[Almanac]
    [[Skyfield]]
        # general options (required, not described here)
        ...
        # enable LOOP packet augmentation
        enable_live_data = true
        # which observation types to calculate live data for
        live_data_obersvations = altitude, azimuth
        # optional list of additional hevanly bodies
        live_data_bodies = 
        # further general options including sub-sections
        ...
```

* `enable_live_data`: enable live data service for fast changing almanac values (default: on)
* `live_data_observations`: list of observation types to calculate live data for. Optional. Default is altitude and azimuth. Possible additional values are declination and right ascension.
* `live_data_bodies`: list of additional heavenly bodies to include in live data (e.g. LOOP packets). Optional. The Sun is always included.

Possible values in `live_data_observations`:
* `altitude`, `azimuth`: If one of them is included in the list, calculate
  altitude and azimuth of the Sun and all the heavenly bodies listed in
  `live_data_bodies`
* `right ascension`, `declination`: If one of them is included in the list,
   calculate right ascension and declination of the Sun and all the heavenly
   bodies listed in `live_data_bodies`
* `libration`: If that is included in the list and additionally `moon` is
  included in the list of `live_data_bodies` then calculate the libration
  of the Moon

Possible values in `live_data_bodies`
* `moon`: calculate almanac data of the Moon
* names of planets and planets' moons: calculate almanac data of those
  heavenly bodies. You can only include heavenly bodies here, which
  are included in the ephemerides you configured to load.

## Observation types

According to the configuration a different set of observation types is
calculated. The names of the observation types contain of the body and
the almanac value, for example `solarAltitude` for the current altitude
of the Sun.

**First part of the observation type name:**

Heavenly body | Value in `live_data_bodies` | First part of the name | Example
--------------|-----------------------------|------------------------|--------
Sun           | always included             | `solar`                | `solarAltitude`
Moon          | `moon`                      | `lunar`                | `lunarAzimuth` 
Mercury       | `mercury`                   | `mercury`              | `mercuryRightAscension`
Venus         | `venus`                     | `venus`                | `venusDeclination`
Mars          | `mars`, `mars_barycenter`   | `mars`                 | `marsAltitude`
etc.          |                             |                        |

**Second part of the observation type name:**

Observation types | Value in `live_data_observations` | Second part of the name | Example
------------------|-----------------------------------|-------------------------|----------
altitude          | `altitude`                        | `Altitude` | `jupiterAltitude`
azimuth           | `azimuth`                         | `Azimuth` | `saturnAzimuth`
topocentric right ascension | `right ascension`       | `RightAscension` | `uranusRightAscension`
topocentric declination     | `declination`           | `Declination` | `neptuneDeclination`
libration latitude | `libration`                      | `LibrationLatitude` | `lunarLibrationLatitude`
libration longitude | `libration`                     | `LibrationLongitude` | `lunarLibrationLongitude`

**Additional observation types:**

* `solarTime`: current hour angle of the Sun, counted from midnight (that is sundial time)
* `solarPath`: current percentage of the way of the Sun from sunrise to sunset
* `lunarTime`: current hour angle of the Moon, counted from antitransit
  (if `moon` is included in `live_data_bodies` only)


## Add observation types to the WeeWX database

If you want to save those values to the database, you have to add columns 
using the 
[weectl utility](https://weewx.com/docs/latest/utilities/weectl-database/#add-a-new-observation-type-to-the-database). 
For example, if you want to include `solarAltitude` to the database use the 
following command:

in case of a WeeWX packet installation:

```shell
sudo weectl database add-column solarAltitude
```

in case of a WeeWX pip installation:

```shell
source ~/weewx-venv/bin/activate
weectl database add-column solarAltitude
```

## Usage

### Live updates

If you want to see some almanac values change every some seconds, first,
include them in MQTT output. Please, refer to the MQTT extension of your
choice for how to do that:
* [weewx-mqtt by Matthew Wall](https://github.com/matthewwall/weewx-mqtt)
* [mqtt publish by Rich Bell](https://github.com/weewx-mqtt/publish)

Alternatives to MQTT output are 
[SQL database upload](https://github.com/roe-dl/weewx-sqlupload)
or
[JSON LOOP data output](https://github.com/chaunceygardiner/weewx-loopdata).

Then configure your skin to include updating those values by JavaScript.
Refer to the skin documentation for how to do that.

### Diagrams

If you want to draw diagrams or plots of almanac values, first add the
respective observation types to the database as described above. Then
configure your skin to show those values in a diagram. Please refer
to the skin documentation for how to do that.

### Mobile stations

For mobile stations the location is updated from `latitude` and `longitude` 
observation types if they are present in the LOOP packet.

## FAQ

Q: What is the difference between `solarAzimuth`, `solarAltitude` and
`$almanac.sun.az` (`$almanac.sun.azimuth`), `$almanac.sun.alt`
(`$almanac.sun.azimuth`), respectively?

A: The difference is the use case:
* `solarAzimuth` and `solarAltitude` are included in LOOP packets and output to MQTT and thus allow live updates on web sites.
* As `solarAzimuth` and `solarAltitude` are observation types, they can be saved to the database and then displayed in diagrams.
* On the other side, `$almanac.sun.az` etc. can have parameters to adjust the output.

Q: Why I cannot setup altitude and azimuth separately?

A: Because the Skyfield module always calculates them together.

Q: Why I cannot setup right ascension and declination separately?

A: Because the Skyfield module always calculates them together.

## Links

* [weewx-skyfield-almanac extension](https://github.com/roe-dl/weewx-skyfield-almanac)
* [WeeWX](https://weewx.com)
* [Skyfield](https://rhodesmill.org/skyfield/)
* [WeeWX sky map extension](https://github.com/roe-dl/weewx-skymap-almanac)
* [weewx-mqtt by Matthew Wall](https://github.com/matthewwall/weewx-mqtt)
* [mqtt publish by Rich Bell](https://github.com/weewx-mqtt/publish)
* [SQL database upload](https://github.com/roe-dl/weewx-sqlupload)
* [JSON LOOP data output](https://github.com/chaunceygardiner/weewx-loopdata)
* [weewx-celestial](https://github.com/chaunceygardiner/weewx-celestial)
  (I did not notice that extension while working on the Skyfield based
  almanac extension for WeeWX. However, both extensions have different
  objectives.)
