# iot.py
import time
import requests

# IoT/cloud configuration
API_URL = "https://example.com/api/energy"

# Meter configuration
VOLTAGE = 230.0          # Volts
COST_PER_KWH = 6.50      # Example tariff in ₹/kWh

total_energy_kwh = 0.0
previous_time = time.time()


def read_current():
    """
    Replace this with code for your actual current sensor.
    Example: ACS712, INA219, PZEM-004T, etc.
    """
    return 2.5  # Amperes


def calculate_power(voltage, current):
    return voltage * current


def calculate_energy(power_watts, elapsed_seconds):
    return (power_watts * elapsed_seconds) / 3_600_000


def send_to_cloud(power, energy, cost):
    data = {
        "voltage": VOLTAGE,
        "current": round(read_current(), 2),
        "power_watts": round(power, 2),
        "energy_kwh": round(energy, 4),
        "cost": round(cost, 2)
    }

    try:
        response = requests.post(API_URL, json=data, timeout=5)
        print("Cloud response:", response.status_code)
    except requests.RequestException as error:
        print("Cloud upload failed:", error)


while True:
    current = read_current()
    power = calculate_power(VOLTAGE, current)

    current_time = time.time()
    elapsed_time = current_time - previous_time
    previous_time = current_time

    total_energy_kwh += calculate_energy(power, elapsed_time)
    estimated_cost = total_energy_kwh * COST_PER_KWH

    print("--------------------------------")
    print(f"Voltage : {VOLTAGE:.1f} V")
    print(f"Current : {current:.2f} A")
    print(f"Power   : {power:.2f} W")
    print(f"Energy  : {total_energy_kwh:.4f} kWh")
    print(f"Cost    : ₹{estimated_cost:.2f}")

    send_to_cloud(power, total_energy_kwh, estimated_cost)

    time.sleep(10)
