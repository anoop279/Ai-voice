# Ai-voice
This is my first git repository on AI voice chat 
<br>
Author :Anoop Kumar
<br>
Code:
<br>
from pydub import AudioSegment
from scipy.io.wavfile import write
import numpy as np
import noisereduce as nr
import pyttsx3
import speech_recognition as sr
import datetime
import os
import cv2
import wikipedia
import spotipy
from spotipy.oauth2 import SpotifyOAuth
from requests import get
import webbrowser
import pywhatkit as kit
import sys

# Spotify API setup
SPOTIFY_CLIENT_ID = "fe4c044352da41ef828f202e99317583"
SPOTIFY_CLIENT_SECRET = "ecc017389f28497ea6a4cfa528afed95"
SPOTIFY_REDIRECT_URI = "https://open.spotify.com/"

spotify = spotipy.Spotify(auth_manager=SpotifyOAuth(
    client_id=SPOTIFY_CLIENT_ID,
    client_secret=SPOTIFY_CLIENT_SECRET,
    redirect_uri=SPOTIFY_REDIRECT_URI,
    scope="user-read-playback-state,user-modify-playback-state,playlist-read-private"))

# Initialize Text-to-Speech
engine = pyttsx3.init('sapi5')
voices = engine.getProperty('voices')
engine.setProperty('voice', voices[0].id)  # Use the first voice

# Text-to-Speech Function
def speak(audio):
    engine.say(audio)
    print(audio)
    engine.runAndWait()

# Wish the user
def wish():
    hour = int(datetime.datetime.now().hour)
    if 0 <= hour <= 12:
        speak("Good morning!")
    elif 12 < hour < 18:
        speak("Good afternoon!")
    else:
        speak("Good evening! , SIR")
    speak("I am AI  assistant. Please tell me how I can help you.")

# Noise Reduction Function
def reduce_noise(input_file, output_file):
    try:
        # Load the audio file
        audio = AudioSegment.from_file(input_file, format="wav")
        audio_samples = np.array(audio.get_array_of_samples())
        sample_rate = audio.frame_rate

        # Convert stereo to mono if necessary
        if audio.channels > 1:
            audio_samples = audio_samples.reshape((-1, audio.channels)).mean(axis=1).astype(audio_samples.dtype)

        # Estimate noise using the first second of audio
        noise_sample = audio_samples[:sample_rate]

        # Apply noise reduction
        reduced_noise = nr.reduce_noise(y=audio_samples, sr=sample_rate, y_noise=noise_sample)

        # Save the denoised audio
        write(output_file, sample_rate, reduced_noise.astype(np.int16))
        print(f"Denoised audio saved to {output_file}")
    except Exception as e:
        print(f"Error during noise reduction: {e}")

# Take voice command with noise reduction
def takecommand():
    r = sr.Recognizer()
    with sr.Microphone() as source:
        print("Calibrating microphone for ambient noise...")
        r.adjust_for_ambient_noise(source, duration=2)
        print("Listening...")
        try:
            audio = r.listen(source, timeout=5, phrase_time_limit=10)

            # Save the raw audio for noise reduction
            with open("raw_audio.wav", "wb") as f:
                f.write(audio.get_wav_data())

            # Apply noise reduction
            reduce_noise("raw_audio.wav", "clean_audio.wav")

            # Load the denoised audio
            with sr.AudioFile("clean_audio.wav") as clean_source:
                clean_audio = r.record(clean_source)
                query = r.recognize_google(clean_audio, language='en-in')

            print(f"User said: {query}")
            return query.lower()

        except sr.UnknownValueError:
            speak("Sorry, I couldn't understand that.")
            return "none"
        except sr.RequestError:
            speak("There seems to be an issue with the speech recognition service.")
            return "none"
        except Exception as e:
            print(f"Error in recognition: {e}")
            speak("An error occurred while recognizing your voice.")
            return "none"
# Play Spotify Music (Updated for Free users)
def play_spotify_music(song_name):
    try:
        search_query = song_name.replace(" ", "+")
        spotify_url = f"https://open.spotify.com/search/{search_query}"
        webbrowser.open(spotify_url)
        speak(f"Opening {song_name} on Spotify.")
    except Exception as e:
        print(f"Error opening Spotify: {e}")
        speak("There was an issue opening Spotify. Please check your setup.")

# Main Execution
if name == 'main':
    wish()
    while True:
        query = takecommand()

        if "open notepad" in query:
            npath = "C:\\Windows\\notepad.exe"
            os.startfile(npath)

        elif "open command prompt" in query:
            os.system("start cmd")

        elif "open camera" in query:
            cap = cv2.VideoCapture(0)
            while True:
                ret, img = cap.read()
                cv2.imshow("Webcam", img)
                if cv2.waitKey(50) == 27:
                    break
            cap.release()
            cv2.destroyAllWindows()

        elif "play music on spotify" in query:
            speak("Which song would you like me to play?")
            song_name = takecommand()
            if song_name != "none":
                play_spotify_music(song_name)

        elif "ip address" in query:
            ip = get("https://api.ipify.org").text
            speak(f"Your IP Address is {ip}.")

        elif "wikipedia" in query:
            speak("Searching Wikipedia...")
            query = query.replace("wikipedia", "")
            results = wikipedia.summary(query, sentences=10)
            speak("According to Wikipedia:")
            speak(results)

        elif "open youtube" in query:
            webbrowser.open("https://www.youtube.com/")

        elif "open google" in query:
            speak("Sir, what should I search for?")
            cm = takecommand()
            webbrowser.open(f"https://www.google.com/search?q={cm}")

        elif "play songs on youtube" in query:
            kit.playonyt("see you again")

        elif "no thanks" in query:
            speak("Thanks for using me, sir. Have a good day.")
            sys.exit()

        speak("Sir, do you have any other work?")
