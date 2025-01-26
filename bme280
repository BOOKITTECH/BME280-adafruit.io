import time
import smbus2
import bme280
import os
import pytz
from Adafruit_IO import Client, Feed, Data, RequestError

# BME280 sensor address (default address)
address = 0x76

# Initialize I2C bus
bus = smbus2.SMBus(1)

# Load calibration parameters
calibration_params = bme280.load_calibration_params(bus, address)

# Check if the file exists before opening it in 'a' mode (append mode)
file_exists = os.path.isfile('sensor_readings_bme280.txt')

# Open the file in append mode
file = open('sensor_readings_bme280.txt', 'a')

# Write the header to the file if the file does not exist
if not file_exists:
    file.write('Time and Date, temperature (ºC), temperature (ºF), humidity (%), pressure (hPa)\n')

def celsius_to_fahrenheit(celsius):
    return (celsius * 9/5) + 32

# Set to your Adafruit IO key.
ADAFRUIT_IO_KEY = 'your-key'
ADAFRUIT_IO_USERNAME = 'your-username'

# Create an instance of the REST client once, outside the loop
aio = Client(ADAFRUIT_IO_USERNAME, ADAFRUIT_IO_KEY)

try:
    while True:
        try:
            # Read sensor data
            data = bme280.sample(bus, address, calibration_params)

            # Extract temperature, pressure, and humidity
            temperature_celsius = data.temperature
            pressure = data.pressure
            humidity = data.humidity
            timestamp = data.timestamp

            # Convert temperature to Fahrenheit
            temperature_fahrenheit = celsius_to_fahrenheit(temperature_celsius)

            # Send data to Adafruit IO
            aio.send_data('temperature', temperature_fahrenheit)
            aio.send_data('humidity', humidity)
            aio.send_data('pressure', pressure)

            # Adjust timezone
            desired_timezone = pytz.timezone('Pacific/Honolulu')  # Replace with your desired timezone

            # Convert the timestamp to the desired timezone
            timestamp_tz = timestamp.replace(tzinfo=pytz.utc).astimezone(desired_timezone)

            # Print the readings
            print(f"{timestamp_tz.strftime('%H:%M:%S %d/%m/%Y')} Temp={temperature_celsius:0.1f}ºC, Temp={temperature_fahrenheit:0.1f}ºF, Humidity={humidity:0.1f}%, Pressure={pressure:0.1f} hPa")

            # Save time, date, temperature, humidity, and pressure in .txt file
            file.write(f"{timestamp_tz.strftime('%H:%M:%S %d/%m/%Y')}, {temperature_celsius:0.2f}, {temperature_fahrenheit:0.2f}, {humidity:0.2f}, {pressure:0.2f}\n")
            # Wait for a few seconds before the next reading
            time.sleep(300)

        except KeyboardInterrupt:
            print('Program stopped')
            break
        except Exception as e:
            print(f'An unexpected error occurred: {str(e)}')
            break

finally:
    # Make sure to close the file when the program is done
    file.close()
