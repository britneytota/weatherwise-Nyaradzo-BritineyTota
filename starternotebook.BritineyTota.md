# Define parse_weather_question() and generate_weather_response() here
def parse_weather_question(question):
    question = question.lower()
    parsed = {'location': None, 'attribute': None, 'time_period': 'today'}

    if 'temperature' in question or 'temp' in question:
        parsed['attribute'] = 'temperature'
    elif 'rain' in question or 'precipitation' in question:
        parsed['attribute'] = 'precipitation'

    if 'tomorrow' in question:
        parsed['time_period'] = 'tomorrow'
    elif 'week' in question or '5 days' in question:
        parsed['time_period'] = '5-day'

    tokens = question.split()
    for i in range(len(tokens)):
        if tokens[i] in ['in', 'for', 'at'] and i + 1 < len(tokens):
            parsed['location'] = ' '.join(tokens[i + 1:]).capitalize()
            break
        elif i == 0 and tokens[i].isalpha() and parsed['location'] is None:
            parsed['location'] = tokens[i].capitalize()

    return parsed

def generate_weather_response(parsed_question, weather_data):
    attribute = parsed_question['attribute']
    time_period = parsed_question['time_period']
    location = parsed_question['location']

    if not weather_data:
        return f"Sorry, I couldn't retrieve the weather data for {location}."

    forecast_days = weather_data.get('forecast', {}).get('forecastday', [])
    if not forecast_days:
        return "No forecast data available."

    if time_period == 'today':
        if len(forecast_days) >= 1:
            day_data = forecast_days[0]['day']
            time_desc = "today"
        else:
            return "No data for today."
    elif time_period == 'tomorrow':
        if len(forecast_days) >= 2:
            day_data = forecast_days[1]['day']
            time_desc = "tomorrow"
        else:
            return "No data for tomorrow."
    elif time_period == '5-day':
        response = f"Here is the 5-day forecast for {location}:\n"
        for day in forecast_days:
            date = day['date']
            avg_temp = day['day'].get('avgtemp_c', 'N/A')
            rain_chance = day['day'].get('daily_chance_of_rain', 'N/A')
            response += f"{date}: Avg Temp: {avg_temp}°C, Rain: {rain_chance}%\n"
        return response
    else:
        return "Time period not recognized."

    if attribute == 'temperature':
        return f"The average temperature in {location} {time_desc} is {day_data.get('avgtemp_c', 'N/A')}°C."
    elif attribute == 'precipitation':
        return f"There is a {day_data.get('daily_chance_of_rain', 'N/A')}% chance of rain in {location} {time_desc}."
    else:
        return "I couldn't identify the weather detail you're asking for."

# Define the menu function outside of generate_weather_response
def main_menu():
    print("=== Welcome to Weather Advisor ===")
    last_location = None

    while True:
        print("\nWeather Advisor Menu")
        print("Please select one of the following:")
        choice = pyip.inputMenu(
            ['Ask a question', 'Show temperature chart', 'Show precipitation chart', 'Exit'],
            numbered=True
        )

        if choice == 'Ask a question':
            question = input("What would you like to know about the weather?\n> ")
            parsed = parse_weather_question(question)
            data = get_weather_data(parsed['location'])
            response = generate_weather_response(parsed, data)
            print("\n" + response + "\n")
            last_location = parsed['location']

        elif choice in ['Show temperature chart', 'Show precipitation chart']:
            if last_location:
                use_last = pyip.inputYesNo(f"Use last location ({last_location})? (yes/no): ")
                if use_last == 'yes':
                    location = last_location
                else:
                    location = pyip.inputStr("Enter a location: ")
                    last_location = location
            else:
                location = pyip.inputStr("Enter a location: ")
                last_location = location

            data = get_weather_data(location)
            if data:
                if choice == 'Show temperature chart':
                    create_temperature_visualisation(data)
                else:
                    create_precipitation_visualisation(data)

        elif choice == 'Exit':
            print("Goodbye!")
            break

## Include sample input/output for each function
def run_tests():
    print("\n--- Running Tests ---")
    sample_location = "London"
    data = get_weather_data(sample_location)
    assert data is not None, "Failed to get weather data."
    assert "forecast" in data, "Missing 'forecast' in response."
    assert "forecastday" in data["forecast"], "Missing 'forecastday' in forecast."
    assert "day" in data["forecast"]["forecastday"][0], "Missing 'day' in forecastday."
    print("All tests passed!")

# === Run the application ===
if __name__ == "__main__":
    run_tests()
    menu()
