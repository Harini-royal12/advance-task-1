import RPi.GPIO as GPIO
import time
import requests
import Adafruit_ADS1x15  # ADC module for analog sensors

# Sensor Configuration
ADC = Adafruit_ADS1x15.ADS1115()
AIR_QUALITY_CHANNEL = 0  # ADC Channel for MQ135 Sensor

# ThingSpeak API Configuration (Optional - For Cloud Monitoring)
THINGSPEAK_API_KEY = "your_thingspeak_api_key"
THINGSPEAK_URL = "https://api.thingspeak.com/update"

# Threshold for air quality (adjustable based on sensor calibration)
AIR_QUALITY_THRESHOLD = 400  # Example threshold value

# Setup GPIO
GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)

def read_air_quality():
    air_quality_value = ADC.read_adc(AIR_QUALITY_CHANNEL, gain=1)
    return air_quality_value

def send_to_cloud(air_quality):
    payload = {
        'api_key': THINGSPEAK_API_KEY,
        'field1': air_quality
    }
    try:
        response = requests.get(THINGSPEAK_URL, params=payload)
        if response.status_code == 200:
            print("Data sent to ThingSpeak successfully!")
        else:
            print("Failed to send data. Response code:", response.status_code)
    except Exception as e:
        print("Error sending data to cloud:", e)

print("IoT-Based Air Quality Monitoring System Active")
try:
    while True:
        air_quality = read_air_quality()
        print(f"Air Quality Level: {air_quality}")
        if air_quality > AIR_QUALITY_THRESHOLD:
            print("Warning: Poor air quality detected!")
        send_to_cloud(air_quality)
        time.sleep(10)  # Check every 10 seconds
except KeyboardInterrupt:
    print("Shutting down...")
    GPIO.cleanup()
