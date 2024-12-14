#import libraries
import google.generativeai as genai
import pyttsx3
import speech_recognition as sr
import requests
import random
import webbrowser
import datetime
import wikipedia
import os
#from transformers import AutoModelForCausalLM, AutoTokenizer
import pyautogui  # Import pyautogui for controlling the laptop
import requests
import json
import customtkinter as ctk
import threading
import keyboard
import time



# Initialize text-to-speech engine
engine = pyttsx3.init()

# Load pre-trained model and tokenizer
"""model_name = "microsoft/DialoGPT-medium"
tokenizer = AutoTokenizer.from_pretrained(model_name, padding_side="right")
tokenizer.add_special_tokens({'pad_token': tokenizer.eos_token})
model = AutoModelForCausalLM.from_pretrained(model_name)

# Set the model to evaluation mode
model.eval()"""

# Function to convert text to speech
import pyttsx3

def speak(text, excitement=True, male_voice=True):
    # Initialize the engine
    engine = pyttsx3.init()
    
    # Get available voices
    voices = engine.getProperty('voices')
    
    # Select male or female voice
    if male_voice:
        engine.setProperty('voice', voices[0].id)  # Typically, voices[0] is male
    else:
        engine.setProperty('voice', voices[1].id)  # Typically, voices[1] is female
    
    # Set speaking rate and volume
    if excitement:
        engine.setProperty('rate', 180)  # Increased speaking rate
        engine.setProperty('volume', 1.0)  # Maximum volume
    else:
        engine.setProperty('rate', 150)  # Normal speaking rate
        engine.setProperty('volume', 0.9)  # Normal volume
    
    # Speak the text
    engine.say(text)
    engine.runAndWait()


# Function to get personalized greeting based on time of day
def get_greeting():
    current_hour = datetime.datetime.now().hour
    if 5 <= current_hour < 12:
        return "Good morning"
    elif 12 <= current_hour < 18:
        return "Good afternoon!"
    else:
        return "Good evening!"



# Function for speech recognition
def listen():
    r = sr.Recognizer()
    with sr.Microphone() as source:
        audio = r.listen(source)
        print("Listening...")
    try:
        user_input = r.recognize_google(audio)
        print("You:", user_input)
        return user_input
    except sr.UnknownValueError:
        speak("Sorry, I couldn't understand what you said.")
        return ""
    except sr.RequestError as e:
        speak(f"Could not request results from Google Speech Recognition service; {e}")
        return ""
    except Exception as e:
        speak(f"An error occurred: {e}")
        return ""



# Function to perform web search
def search_web(query):
    url = f"https://www.google.com/search?q={query}"
    webbrowser.open(url)
    return "I've opened the search results in your web browser."

# Function to calculate mathematical expressions
def calculate(expression):
    try:
        result = eval(expression)
        return f"The result is {result}."
    except Exception as e:
        return f"Error in calculation: {str(e)}"

# Function to tell a joke
def tell_joke():
    jokes = [
        "Why don't scientists trust atoms? Because they make up everything!",
        "Did you hear about the mathematician who’s afraid of negative numbers? He’ll stop at nothing to avoid them!",
        "Why did the scarecrow win an award? Because he was outstanding in his field!",
        "Why did the bicycle fall over? Because it was two-tired!"
    ]
    return random.choice(jokes)

# Function to fetch news headlines
def get_news():
    news_api_key = "abfc5096e19845f6bd33c07d79a8e0a9"  # Replace with your actual news API key
    url = f"https://newsapi.org/v2/top-headlines?country=us&apiKey=abfc5096e19845f6bd33c07d79a8e0a9"
    response = requests.get(url)
    data = response.json()
    if data["status"] == "ok":
        articles = data["articles"][:5]  # Get top 5 headlines
        headlines = [article["title"] for article in articles]
        return "Here are the top news headlines: " + " | ".join(headlines)
    else:
        return "Sorry, I couldn't retrieve the news."

# Function to give personalized suggestions
def suggest_recommendations(category):
    suggestions = {
        "movies": ["Inception", "The Matrix", "Interstellar"],
        "books": ["1984 by George Orwell", "To Kill a Mockingbird by Harper Lee", "The Great Gatsby by F. Scott Fitzgerald"],
        "music": ["Bohemian Rhapsody by Queen", "Hotel California by Eagles", "Stairway to Heaven by Led Zeppelin"]
    }
    return random.choice(suggestions.get(category, ["I don't have suggestions for that category."]))

# Function to give feedback based on user's activities
def give_feedback(activity):
    feedback = {
        "work": ["Keep up the good work!", "You're doing great!", "Don't forget to take breaks."],
        "exercise": ["Nice job staying active!", "Keep pushing yourself!", "Remember to stay hydrated."],
        "study": ["Keep up the hard work!", "You're making great progress!", "Stay focused and you'll achieve your goals."]
    }
    return random.choice(feedback.get(activity, ["I don't have feedback for that activity."]))


def get_weather(city):
    """
    Fetches the current weather for a specified city using OpenWeatherMap API.
    
    Args:
        city (str): Name of the city.
        
    Returns:
        str: A message containing the weather details or an error message.
    """
    api_key = "e03744c9889ba583f17b9fa549bbab3c"  # Replace with your API key
    base_url = "http://api.openweathermap.org/data/2.5/weather"

    try:
        # API request
        params = {
            "q": city,
            "appid": api_key,
            "units": "metric"  # For temperature in Celsius
        }
        response = requests.get(base_url, params=params)
        data = response.json()

        if response.status_code == 200:
            # Extract relevant data
            city_name = data["name"]
            temp = data["main"]["temp"]
            feels_like = data["main"]["feels_like"]
            weather_description = data["weather"][0]["description"]
            humidity = data["main"]["humidity"]
            wind_speed = data["wind"]["speed"]

            # Format response
            weather_info = (
                f"Weather in {city_name}:\n"
                f"- Temperature: {temp}°C\n"
                f"- Feels Like: {feels_like}°C\n"
                f"- Description: {weather_description.capitalize()}\n"
                f"- Humidity: {humidity}%\n"
                f"- Wind Speed: {wind_speed} m/s"
            )
            return weather_info
        elif data["cod"] == "404":
            return f"City '{city}' not found. Please check the name and try again."
        else:
            return f"Error: {data.get('message', 'Unable to fetch weather data')}"

    except requests.exceptions.RequestException as e:
        return f"Error: Unable to connect to the weather service. {e}"


def chat(prompt):
    # Set up the generative AI API
    genai.configure(api_key="AIzaSyBxYKQedL4vvgmK-yxpbrXVp-Fa1HSS6DI")
    model = genai.GenerativeModel("gemini-1.5-flash")
    
    # Generate a response
    response = model.generate_content(prompt)
    
    # Limit the length of the response manually
    truncated_response = response.text.strip()[:300]  # Truncate to 150 characters 
    return truncated_response


# Initialize the app
app = ctk.CTk()
app.attributes("-fullscreen", True)
app.title("Voice-Controlled AI Assistant")

# Themes
light_theme = {
    "bg_color": "#F5F5F5",
    "text_color": "#1d1f1f",
    "bubble_user_color": "#59eff7",
    "bubble_ai_color": "#03f2ff",
    "progress_color": "#4682B4",
}
dark_theme = {
    "bg_color": "#2B2B2B",
    "text_color": "#FFFFFF",
    "bubble_user_color": "#2A2A2A",
    "bubble_ai_color": "#1E1E1E",
    "progress_color": "#FFA500",
}
current_theme = dark_theme  # Start with dark mode


# Function to apply theme to the app
def apply_theme(theme):
    global current_theme
    current_theme = theme
    # Apply colors to main components
    frame.configure(fg_color=theme["bg_color"])
    chat_frame.configure(fg_color=theme["bg_color"])
    status_frame.configure(fg_color=theme["bg_color"])
    title_label.configure(text_color=theme["text_color"])
    status_label.configure(text_color=theme["text_color"])
    progress_bar.configure(progress_color=theme["progress_color"])

    # Update all existing chat bubbles
    for widget in chat_frame.winfo_children():
        if isinstance(widget, ctk.CTkFrame):
            bubble_text = widget.winfo_children()[0].cget("text")
            bg_color = (
                theme["bubble_user_color"]
                if bubble_text.startswith("You:")
                else theme["bubble_ai_color"]
            )
            widget.configure(fg_color=bg_color)


# Function to toggle between light and dark themes
def toggle_theme():
    apply_theme(light_theme if current_theme == dark_theme else dark_theme)


# Function to handle exit
def exit_app():
    speak("Goodbye!", excitement=True)
    app.destroy()


# Function to update status with an animated progress bar
def update_status_with_progress(status_text, duration=3):
    def animate_progress():
        progress = 0
        increment = 100 / (duration * 20)
        while progress < 100:
            progress_bar.set(progress / 100)
            time.sleep(0.05)
            progress += increment
        progress_bar.set(1.0)

    update_status(status_text)
    progress_thread = threading.Thread(target=animate_progress)
    progress_thread.start()


# Function to update status
def update_status(status_text):
    status_label.configure(text=status_text)
    app.update()


# Authorization function
def authorize():
    while True:
        update_status_with_progress("Listening for authorization password...")
        speak("Please say the authorization password.", excitement=True)

        # Simulate voice input
        audio = listen()  # Replace with actual audio capture logic

        if audio.lower() == "password":  # Replace "password" with your chosen keyword
            update_status("Authorization successful. Welcome!")
            speak("Authorization successful. Booting up the system.")
            break
        else:
            update_status("Authorization failed. Waiting for password...")
            speak("Authorization failed. Please try again.")


# Function to display messages in chat bubbles
def display_chat_bubble(message, sender="You"):
    bubble_frame = ctk.CTkFrame(
        chat_frame,
        fg_color=current_theme["bubble_user_color"] if sender == "You" else current_theme["bubble_ai_color"],
        corner_radius=15,
    )
    bubble_frame.pack(anchor="e" if sender == "You" else "w", pady=5, padx=10, fill="x")

    bubble_label = ctk.CTkLabel(
        bubble_frame,
        text=f"{sender}: {message}",
        font=("Arial", 14),
        wraplength=600,
        text_color=current_theme["text_color"],
    )
    bubble_label.pack(anchor="e" if sender == "You" else "w")


# Function to process user input
def process_input():
    update_status_with_progress("Listening...")
    speak("Listening...")
    user_input = listen()  # Replace with actual audio capture logic
    if not user_input:
        update_status("Waiting for input...")
        return

    display_chat_bubble(user_input, sender="You")

    update_status_with_progress("Processing...")

    response = ""

    if "calculate" in user_input.lower():
        expression = user_input.replace("calculate", "")
        response = calculate(expression)
    elif "weather" in user_input:
        city = user_input.split("weather in")[1].strip()
        response = get_weather(city)
    elif "joke" in user_input:
        response = tell_joke()
    elif "news" in user_input:
        response = get_news()
    elif "search" in user_input:
        query = user_input.replace("search", "")
        response = search_web(query)
    elif "recommend" in user_input:
        category = user_input.split("recommend ")[-1]
        response = f"Here's a recommendation for {category}: {suggest_recommendations(category)}."
    elif "open Notepad" in user_input:
        response = "Opening Notepad."
        os.system("notepad.exe")  
    elif "time" in user_input.lower():
        current_time = datetime.datetime.now().strftime("%H:%M")
        response = f"The current time is {current_time}."
    elif "date" in user_input.lower():
        current_date = datetime.datetime.now().strftime("%Y-%m-%d")
        response = f"The current date is {current_date}."
    elif user_input.lower() == "quit":
        exit_app()
    else:
        response = chat(user_input)

    display_chat_bubble(response, sender="Assistant")
    speak(response)
    update_status("Waiting for input...")


# Frontend layout
frame = ctk.CTkFrame(app, fg_color=current_theme["bg_color"])
frame.pack(fill="both", expand=True, pady=0, padx=0)

title_label = ctk.CTkLabel(
    frame,
    text="AI Assistant",
    font=("New York", 24, "bold"),
    fg_color="transparent",
    text_color=current_theme["text_color"],
)
title_label.pack(pady=20, anchor="n")

chat_frame = ctk.CTkScrollableFrame(frame)
chat_frame.pack(fill="both", expand=True, pady=10, padx=20)

status_frame = ctk.CTkFrame(app, fg_color=current_theme["bg_color"], height=50)
status_frame.pack(side="bottom", fill="x", pady=5)

status_label = ctk.CTkLabel(
    status_frame,
    text="Waiting for input...",
    font=("New York", 14, "bold"),
    fg_color="transparent",
    text_color=current_theme["text_color"],
    anchor="w",
)
status_label.pack(side="left", padx=10)

progress_bar = ctk.CTkProgressBar(
    status_frame,
    width=200,
    height=15,
    progress_color=current_theme["progress_color"],
)
progress_bar.pack(side="right", padx=10)
progress_bar.set(0.0)

# Theme toggle switch
theme_toggle = ctk.CTkSwitch(
    status_frame,
    text="Mode",
    command=toggle_theme,
    onvalue="Dark",
    offvalue="Light",
)
theme_toggle.pack(side="left", padx=10)

# Voice control function triggered by 's' key
def start_voice_control():
    while True:
        keyboard.wait("s")
        process_input()


# Run the app and voice control in parallel
speak(get_greeting())
authorize()
voice_thread = threading.Thread(target=start_voice_control, daemon=True)
voice_thread.start()

apply_theme(current_theme)  # Apply initial theme
app.mainloop()



from flask import Flask, request, jsonify
from flask_cors import CORS

app = Flask(__name__)
CORS(app)  # Allow React to communicate with the backend

@app.route("/chat", methods=["POST"])
def chat():
    user_input = request.json.get("message")

    # Replace this block with your chatbot logic
    if "hello" in user_input.lower():
        response = "Hi there! How can I assist you today?"
    elif "bye" in user_input.lower():
        response = "Goodbye! Have a great day!"
    
    elif "calculate" in user_input.lower():
        expression = user_input.replace("calculate", "")
        response = calculate(expression)
    elif "weather" in user_input:
        city = user_input.split("weather in")[1].strip()
        response = get_weather(city)
    elif "joke" in user_input:
        response = tell_joke()
    elif "news" in user_input:
        response = get_news()
    elif "search" in user_input:
        query = user_input.replace("search", "")
        response = search_web(query)
    elif "recommend" in user_input:
        category = user_input.split("recommend ")[-1]
        response = f"Here's a recommendation for {category}: {suggest_recommendations(category)}."
    elif "open Notepad" in user_input:
        response = "Opening Notepad."
        os.system("notepad.exe")  
    elif "time" in user_input.lower():
        current_time = datetime.datetime.now().strftime("%H:%M")
        response = f"The current time is {current_time}."
    elif "date" in user_input.lower():
        current_date = datetime.datetime.now().strftime("%Y-%m-%d")
        response = f"The current date is {current_date}."
    elif user_input.lower() == "quit":
        exit_app()
    else:
        response = chat(user_input)

    return jsonify({"response": response})  # Respond with JSON

if __name__ == "__main__":
    app.run(debug=True)
