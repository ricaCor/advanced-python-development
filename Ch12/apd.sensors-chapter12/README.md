# Advanced Python Development Sensors

This is the data collection package that forms part of the running example
for the book [Advanced Python Development](https://advancedpython.dev).

## Usage

This installs a console script called `sensors` that returns a report on
various aspects of the system. The available sensors are:

* Python version
* IP Addresses
* CPU Usage
* RAM Available
* Battery charging state
* Ambient Temperature
* Ambient Humidity

There are no command-line options, to view the report run `sensors` on the
command line.

## Caveats

The Ambient Temperature and Ambient Humidity sensors are only available on
RaspberryPi hosts and assume that a DHT22 sensor is connected to pin `D20`.

The location and type of the sensor can be controlled by setting a pair of
environment variables: `APD_SENSORS_TEMPERATURE_BOARD` and
`APD_SENSORS_TEMPERATURE_PIN`.

If there is an entry in `/etc/hosts` for the current machine's hostname that
value will be the only result from the IP Addresses sensor.

## Installation

You can install with `pip3 install apd.sensors` under Python 3.7 or higher.

We recommend using pipenv to manage your environment, in which case you would
install using `pipenv --three install apd.sensors` and run the programme using 
`pipenv run sensors`.

## API server

There is an optional API server shipped with apd.sensors. To use this you
should install the `apd.sensors[webapp]` extra. The API can then be started
with the non-production quality wsgiref server using:

    python -m apd.sensors.wsgi.serve

or through Waitress (if installed) using:

    waitress-serve --call apd.sensors.wsgi:set_up_config

Other WSGI servers will also work, you should use set_up_config as a factory
function.

An environment variable is required to use the API server, `APD_SENSORS_API_KEY`
should be set to the API key required to gain access. One can be generated
using:

    python -c "import uuid; print(uuid.uuid4().hex)"

The following endpoints are supported:

* /v/3.0/sensors
* /v/3.0/sensors/sensorid
* /v/3.0/deployment_id

## Historical data

You can install optional functionality to periodically store sensor
values using the `apd.sensors[scheduled]` extra. In this case, 
`sensors --save` will store the recorded data to `sensor_data.sqlite`
in the current working directory.

The database connection can be specified with `--db sqlite:////var/sensors.sqlite`,
for example. It can also be specified with the `APD_SENSORS_DB_URI`
environment variable.

The database must be migrated to contain the correct data first. This can be
done by running `alembic upgrade head` with the following alembic.ini file
in the current working directory.

    [alembic]
    script_location = apd.sensors:alembic
    sqlalchemy.url = sqlite:///sensor_data.sqlite

### Historical data API

An API to extract historical data is also available if installed with `apd.sensors[webapp,scheduled,storedapi]`.

This provides the following three URIs, where start and end are a date/time in ISO format.

* /v/3.0/historical
* /v/3.0/historical/start
* /v/3.0/historical/start/end
