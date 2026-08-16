# The-Jarvis-virtual-assistant-


##Theory about Jarvis

**Jarvis is a virtual assistant designed to interact with users through voice or text commands.
It can listen to the user's instructions and perform different tasks automatically.
A virtual assistant can use speech recognition to convert spoken words into text.
It can then process the user's command and determine the required action.
Jarvis can answer questions and provide useful information to the user.
It can also open applications, websites, files, or other programs.
Text-to-speech technology can be used to give responses through voice.
A virtual assistant can also perform simple tasks such as telling the time or date.
It can be connected with APIs to access additional information and services.
Python is commonly used to develop basic virtual assistant projects.
Libraries such as SpeechRecognition and pyttsx3 can be used for voice interaction.
Artificial intelligence can make the assistant more capable of understanding commands.
The assistant can be designed to respond to specific keywords or wake words.
Jarvis can be extended with new commands and features according to user requirements.
Overall, a virtual assistant provides an interactive way to control tasks using natural language and voice commands.**







***python cocde***

```
import speech_recognition as sr
import webbrowser
from gtts import gTTS
import pygame
import os
import tempfile
import uuid
import subprocess
import urllib.parse
import datetime
import random
import warnings
import sys
import time
import pyautogui
import pygetwindow as gw

# ============ SUPPRESS WARNINGS ============
warnings.filterwarnings("ignore")
warnings.simplefilter("ignore")

# Suppress pygame welcome message
os.environ['PYGAME_HIDE_SUPPORT_PROMPT'] = "hide"

# Suppress deprecation warnings
if not sys.warnoptions:
    import warnings

    warnings.simplefilter("ignore")

# ============ CONFIGURATION ============
# Google Gemini API Key (Free from https://aistudio.google.com/)

#put here your api key:
#(api key)

# Contacts Dictionary - Add your contacts here
CONTACTS = {
    "mom": "Mother",
    "dad": "Father",
    "brother": "Brother",
    "sister": "Sister",
    "friend": "Friend",
    "wife": "Wife",
    "husband": "Husband",
    "boss": "Boss",
    "colleague": "Colleague"
}

# ======================================

recognizer = sr.Recognizer()

# Suppress pygame mixer init messages
pygame.mixer.pre_init()
pygame.mixer.init()
open_processes = {}

# Initialize Gemini
if GEMINI_API_KEY != (your api key wanted in this bracket)
    genai.configure(api_key=GEMINI_API_KEY)
    model = genai.GenerativeModel('gemini-1.5-flash')
    ai_available = True
else:
    ai_available = False

# AI Conversation History
conversation_history = []
MAX_HISTORY = 10

# Random responses for casual talk
casual_responses = {
    "how are you": [
        "I'm doing great! Thanks for asking. How can I assist you today?",
        "I'm functioning perfectly! Ready to serve you, sir.",
        "I'm excellent! Always happy to chat with you."
    ],
    "what is your name": [
        "I'm Jarvis, your personal AI assistant. Created to help you with various tasks.",
        "My name is Jarvis, at your service!",
        "I'm Jarvis. Think of me as your friendly AI companion."
    ],
    "who created you": [
        "I was created by a talented developer who wanted to build an intelligent personal assistant.",
        "A passionate programmer brought me to life!",
        "My creator is a tech enthusiast who coded me to be helpful and conversational."
    ],
    "thank you": [
        "You're welcome! Happy to help.",
        "My pleasure, sir! Anything else you need?",
        "Anytime! That's what I'm here for."
    ],
    "good morning": [
        "Good morning, sir! Hope you have a wonderful day ahead.",
        "Morning! Ready to tackle the day?",
        "Good morning! How can I make your day better?"
    ],
    "good night": [
        "Good night, sir! Sleep well. I'll be here when you need me.",
        "Rest well! See you tomorrow.",
        "Good night! Sweet dreams."
    ],
    "hello": [
        "Hello, sir! How can I assist you?",
        "Hi there! What can I do for you?",
        "Hey! Ready to help you with anything."
    ],
    "what time is it": [
        f"The current time is {datetime.datetime.now().strftime('%I:%M %p')}.",
        f"It's {datetime.datetime.now().strftime('%I:%M %p')} right now.",
        f"Time is {datetime.datetime.now().strftime('%I:%M %p')}."
    ],
    "what is the date": [
        f"Today is {datetime.datetime.now().strftime('%B %d, %Y')}.",
        f"It's {datetime.datetime.now().strftime('%A, %B %d, %Y')}.",
        f"The date is {datetime.datetime.now().strftime('%d/%m/%Y')}."
    ],
    "tell me a joke": [
        "Why don't scientists trust atoms? Because they make up everything!",
        "What do you call a fake noodle? An impasta!",
        "Why did the scarecrow win an award? Because he was outstanding in his field!"
    ],
    "what's up": [
        "Not much, just waiting for your command!",
        "Everything's running smoothly! What's up with you?",
        "Just here ready to help! How can I assist?"
    ],
    "good": [
        "Glad to hear that! How can I help you today?",
        "That's great! What would you like me to do?",
        "Awesome! Ready for your next command?"
    ]
}


def speak(text):
    """Convert text to speech"""
    filename = f"{tempfile.gettempdir()}/{uuid.uuid4()}.mp3"
    try:
        tts = gTTS(text=text, lang="en", tld="co.uk")
        tts.save(filename)
        pygame.mixer.music.load(filename)
        pygame.mixer.music.play()
        while pygame.mixer.music.get_busy():
            pygame.time.Clock().tick(10)
        pygame.mixer.music.unload()
    except Exception as e:
        print(f"Speak error: {e}")
    finally:
        if os.path.exists(filename):
            try:
                os.remove(filename)
            except Exception:
                pass


def get_ai_response(text):
    """Get AI-powered response using Gemini"""
    # Check for casual responses first
    for key in casual_responses:
        if key in text.lower():
            return random.choice(casual_responses[key])

    # If AI is not available, use fallback
    if not ai_available:
        # Use casual responses only
        for key in casual_responses:
            if key in text.lower():
                return random.choice(casual_responses[key])
        return "I'm here to help! What would you like me to do?"

    try:
        # Build conversation context
        context = "\n".join(conversation_history[-MAX_HISTORY:])
        prompt = f"""You are Jarvis, a friendly and helpful AI assistant. 
        Respond naturally like a human would in casual conversation.
        Keep responses concise and friendly.

        Previous conversation:
        {context}

        User: {text}
        Jarvis: """

        # Use Gemini
        response = model.generate_content(prompt)
        reply = response.text.strip()

        # Add to conversation history
        conversation_history.append(f"User: {text}")
        conversation_history.append(f"Jarvis: {reply}")

        # Keep history manageable
        if len(conversation_history) > MAX_HISTORY * 2:
            conversation_history[:MAX_HISTORY] = []

        return reply

    except Exception as e:
        print(f"AI Error: {e}")
        # Fallback to casual responses
        for key in casual_responses:
            if key in text.lower():
                return random.choice(casual_responses[key])
        return "I'm sorry, I couldn't process that. Can you please rephrase?"


def close_by_process_name(name):
    try:
        subprocess.run(["taskkill", "/F", "/IM", name], capture_output=True)
        return True
    except Exception as e:
        print(e)
        return False


def close_youtube_tab():
    """Close the current YouTube tab in browser"""
    try:
        # Method 1: Use pyautogui to close current tab (Ctrl+W)
        pyautogui.hotkey('ctrl', 'w')
        time.sleep(0.5)
        return True
    except:
        pass

    try:
        # Method 2: Try to find and close YouTube window
        windows = gw.getWindowsWithTitle('YouTube')
        if windows:
            for window in windows:
                window.close()
            return True
    except:
        pass

    return False


def close_youtube_app():
    """Close YouTube application (if installed as app)"""
    try:
        # Try to close YouTube app if it's running as a separate app
        result = subprocess.run(["tasklist", "/FI", "IMAGENAME eq YouTube.exe"], capture_output=True, text=True)
        if "YouTube.exe" in result.stdout:
            subprocess.run(["taskkill", "/F", "/IM", "YouTube.exe"], capture_output=True)
            return True
    except:
        pass

    try:
        # Try to close YouTube in browser using alt+f4
        pyautogui.hotkey('alt', 'f4')
        time.sleep(0.5)
        return True
    except:
        pass

    return False


def close_whatsapp():
    """Close WhatsApp Desktop"""
    try:
        # Close WhatsApp app
        result = subprocess.run(["tasklist", "/FI", "IMAGENAME eq WhatsApp.exe"], capture_output=True, text=True)
        if "WhatsApp.exe" in result.stdout:
            subprocess.run(["taskkill", "/F", "/IM", "WhatsApp.exe"], capture_output=True)
            return True
    except:
        pass
    return False


def find_whatsapp_path():
    """Find WhatsApp executable path"""
    try:
        # Common paths to check
        paths_to_check = [
            r"C:\Program Files\WindowsApps\WhatsAppDesktop_*_x64__cv1g1gvanyjgm\WhatsApp.exe",
            r"C:\Program Files\WindowsApps\WhatsAppDesktop_*_x86__cv1g1gvanyjgm\WhatsApp.exe",
            os.path.expanduser(r"~\AppData\Local\WhatsApp\WhatsApp.exe"),
            os.path.expanduser(r"~\AppData\Local\Programs\WhatsApp\WhatsApp.exe"),
        ]

        for path_pattern in paths_to_check:
            if "*" in path_pattern:
                import glob
                matches = glob.glob(path_pattern)
                if matches:
                    return matches[0]
            else:
                if os.path.exists(path_pattern):
                    return path_pattern
    except:
        pass

    return None


def open_whatsapp():
    """Open WhatsApp Desktop App"""
    try:
        # First try: Use Windows start command with whatsapp protocol
        subprocess.Popen("start whatsapp:", shell=True)
        time.sleep(2)
        speak("Opening WhatsApp Desktop")
        return True
    except:
        pass

    try:
        # Second try: Find and open WhatsApp executable
        whatsapp_path = find_whatsapp_path()
        if whatsapp_path:
            if whatsapp_path.endswith(".exe"):
                subprocess.Popen([whatsapp_path])
            else:
                subprocess.Popen("start whatsapp:", shell=True)
            time.sleep(2)
            speak("Opening WhatsApp Desktop")
            return True
    except:
        pass

    try:
        # Third try: Use Windows Start Menu
        subprocess.Popen("start shell:AppsFolder\\5319275A.51895FA4EA97F_cv1g1gvanyjgm!App", shell=True)
        time.sleep(2)
        speak("Opening WhatsApp Desktop")
        return True
    except:
        pass

    try:
        # Fourth try: Search in Program Files
        import glob
        whatsapp_files = glob.glob(r"C:\Program Files\WindowsApps\*WhatsAppDesktop*\WhatsApp.exe")
        if whatsapp_files:
            subprocess.Popen([whatsapp_files[0]])
            time.sleep(2)
            speak("Opening WhatsApp Desktop")
            return True
    except:
        pass

    speak("Sorry, I couldn't find WhatsApp on your system.")
    return False


def send_whatsapp_message_desktop(contact_name, message):
    """Send WhatsApp message using Desktop App"""
    try:
        # Open WhatsApp Desktop
        if not open_whatsapp():
            return False

        time.sleep(3)

        # Click on search/new chat button
        pyautogui.hotkey('ctrl', 'n')
        time.sleep(1)

        # Type contact name
        pyautogui.write(contact_name)
        time.sleep(2)

        # Press Enter to select first contact
        pyautogui.press('enter')
        time.sleep(2)

        # Type the message
        pyautogui.write(message)
        time.sleep(1)

        # Press Enter to send
        pyautogui.press('enter')
        time.sleep(1)

        speak(f"Message sent to {contact_name}")
        return True

    except Exception as e:
        print(f"WhatsApp error: {e}")
        speak(f"Sorry, I couldn't send message to {contact_name}")
        return False


def send_whatsapp_message_fallback(contact_name, message):
    """Fallback: Send using WhatsApp Web if Desktop app fails"""
    try:
        webbrowser.open("https://web.whatsapp.com")
        speak(f"Opening WhatsApp Web to send message to {contact_name}")
        time.sleep(5)

        # Find contact in search
        pyautogui.hotkey('ctrl', 'f')
        time.sleep(1)
        pyautogui.write(contact_name)
        time.sleep(2)
        pyautogui.press('enter')
        time.sleep(2)

        # Type and send message
        pyautogui.write(message)
        time.sleep(1)
        pyautogui.press('enter')
        time.sleep(1)

        speak(f"Message sent to {contact_name}")
        return True

    except Exception as e:
        print(f"WhatsApp Web error: {e}")
        return False


def processCommand(command):
    command = command.lower()

    # ===== YOUTUBE TAB CLOSING COMMANDS =====
    if "close the recent tab" in command and "youtube" in command:
        if close_youtube_tab():
            speak("Closed the recent YouTube tab")
        else:
            speak("Sorry, I couldn't find any YouTube tab to close")
        return

    elif "close youtube tab" in command or "close youtube" in command:
        if close_youtube_tab():
            speak("Closed YouTube tab")
        else:
            speak("Sorry, I couldn't close the YouTube tab")
        return

    elif "close youtube app" in command or "close youtube application" in command:
        if close_youtube_app():
            speak("Closed YouTube application")
        else:
            speak("Sorry, I couldn't close YouTube application")
        return

    elif "youtube close" in command or "close youtube completely" in command:
        # Try both methods
        tab_closed = close_youtube_tab()
        app_closed = close_youtube_app()
        if tab_closed or app_closed:
            speak("Closed YouTube")
        else:
            speak("Sorry, I couldn't close YouTube")
        return

    # ===== WHATSAPP COMMANDS =====
    if "whatsapp" in command and ("send" in command or "message" in command or "text" in command):
        # Extract contact name and message
        parts = command.split("to")
        if len(parts) > 1:
            contact_part = parts[1].strip()
            if "message" in contact_part or "text" in contact_part:
                contact_name = contact_part.split("message")[0].split("text")[0].strip()
                message_part = contact_part.split("message")[-1].split("text")[-1].strip()
            elif "say" in contact_part:
                contact_name = contact_part.split("say")[0].strip()
                message_part = contact_part.split("say")[-1].strip()
            else:
                contact_name = contact_part
                message_part = f"Hi! How are you? This is a message from Jarvis."

            contact_name = contact_name.replace("send", "").replace("message", "").replace("text", "").strip()

            if contact_name.lower() in CONTACTS:
                actual_name = CONTACTS[contact_name.lower()]
            else:
                actual_name = contact_name

            success = send_whatsapp_message_desktop(actual_name, message_part)
            if not success:
                speak("Trying WhatsApp Web as fallback...")
                send_whatsapp_message_fallback(actual_name, message_part)
        else:
            speak("Please tell me who to send the message to.")
        return

    elif "whatsapp" in command and "open" in command:
        open_whatsapp()
        return

    elif "close whatsapp" in command:
        if close_whatsapp():
            speak("Closed WhatsApp")
        else:
            speak("Could not close WhatsApp")
        return

    # ===== AI CONVERSATIONAL COMMANDS =====
    if "how are you" in command or "what's up" in command or "how's it going" in command:
        reply = get_ai_response(command)
        speak(reply)
        return

    elif "tell me a joke" in command or "make me laugh" in command:
        reply = get_ai_response(command)
        speak(reply)
        return

    elif "what is your name" in command or "who are you" in command:
        reply = get_ai_response(command)
        speak(reply)
        return

    elif "who created you" in command or "who made you" in command:
        reply = get_ai_response(command)
        speak(reply)
        return

    elif "thank you" in command or "thanks" in command:
        reply = get_ai_response(command)
        speak(reply)
        return

    elif "good morning" in command or "good afternoon" in command or "good evening" in command:
        reply = get_ai_response(command)
        speak(reply)
        return

    elif "good night" in command or "goodbye" in command or "bye" in command:
        reply = get_ai_response(command)
        speak(reply)
        return

    elif "what time is it" in command or "what's the time" in command:
        reply = get_ai_response(command)
        speak(reply)
        return

    elif "what is the date" in command or "today's date" in command:
        reply = get_ai_response(command)
        speak(reply)
        return

    elif "hello" in command or "hi" in command or "hey" in command:
        reply = get_ai_response(command)
        speak(reply)
        return

    # ===== COMMAND COMMANDS (Original Functionality) =====
    elif "search" in command and "youtube" in command:
        query = command.replace("search", "", 1).replace("on youtube", "", 1).replace("youtube", "", 1).strip()
        if query:
            url = f"https://www.youtube.com/results?search_query={urllib.parse.quote(query)}"
            webbrowser.open(url)
            speak(f"Searching YouTube for {query}")
        else:
            speak("What do you want me to search on YouTube?")

    elif "shorts" in command:
        query = command.replace("play", "", 1).replace("shorts", "", 1).replace("on youtube", "", 1).strip()
        if query:
            url = f"https://www.youtube.com/results?search_query={urllib.parse.quote(query)}+shorts"
            webbrowser.open(url)
            speak(f"Searching for {query} shorts")
        else:
            webbrowser.open("https://www.youtube.com/shorts")
            speak("Ok Sir!")

    elif "open youtube" in command:
        webbrowser.open("https://www.youtube.com")
        speak("Opening YouTube")

    elif "open google" in command:
        webbrowser.open("https://www.google.com")
        speak("Opening Google")

    elif "open facebook" in command:
        webbrowser.open("https://www.facebook.com")
        speak("Opening Facebook")

    elif "open linkedin" in command:
        webbrowser.open("https://www.linkedin.com")
        speak("Opening LinkedIn")

    elif "file manager" in command or "file explorer" in command or "open files" in command:
        try:
            subprocess.Popen("explorer", shell=True)
            speak("Opening File Manager")
        except Exception as e:
            print(e)
            speak("Sorry, I couldn't open file manager")

    elif "open ms word" in command or "open word" in command:
        try:
            subprocess.Popen("start winword", shell=True)
            speak("Opening Microsoft Word")
        except Exception as e:
            print(e)
            speak("Sorry, I couldn't open Word")

    elif "open notepad" in command:
        try:
            subprocess.Popen(["notepad.exe"])
            speak("Opening Notepad")
        except Exception as e:
            print(e)
            speak("Sorry, I couldn't open Notepad")

    elif "open anaconda prompt" in command:
        try:
            subprocess.Popen("start anaconda prompt", shell=True)
            speak("Opening Anaconda Prompt")
        except Exception as e:
            print(e)
            speak("Sorry, I couldn't open Anaconda Prompt")

    elif "open pycharm" in command:
        try:
            subprocess.Popen("start pycharm", shell=True)
            speak("Opening PyCharm")
        except Exception as e:
            print(e)
            speak("Sorry, I couldn't open PyCharm")

    elif "close word" in command:
        if close_by_process_name("winword.exe"):
            speak("Closed Word")
        else:
            speak("Could not close Word")

    elif "close notepad" in command:
        if close_by_process_name("notepad.exe"):
            speak("Closed Notepad")
        else:
            speak("Could not close Notepad")

    elif "close pycharm" in command:
        if close_by_process_name("pycharm64.exe"):
            speak("Closed PyCharm")
        else:
            speak("Could not close PyCharm")

    elif "close anaconda" in command:
        if close_by_process_name("cmd.exe"):
            speak("Closed Anaconda Prompt")
        else:
            speak("Could not close Anaconda Prompt")

    elif "close edge" in command:
        if close_by_process_name("msedge.exe"):
            speak("Closed Edge")
        else:
            speak("Could not close Edge")

    elif "search" in command:
        query = command.replace("search", "", 1).strip()
        if query:
            webbrowser.open(f"https://www.google.com/search?q={urllib.parse.quote(query)}")
            speak(f"Searching for {query}")
        else:
            speak("What do you want me to search?")

    else:
        # Try AI for unknown commands
        reply = get_ai_response(command)
        speak(reply)


if __name__ == "__main__":
    # Clear screen before starting
    os.system('cls' if os.name == 'nt' else 'clear')

    # Display only this message
    print("\n" + "=" * 50)
    print("     Hey sir have a nice day! 😊")
    print("=" * 50 + "\n")

    # Speak greeting
    speak("Hey sir have a nice day!")

    while True:
        try:
            try:
                with sr.Microphone() as source:
                    print("🎤 Listening...")
                    recognizer.adjust_for_ambient_noise(source, duration=0.5)
                    audio = recognizer.listen(source, timeout=5, phrase_time_limit=3)
            except OSError as e:
                print(f"Microphone error: {e}")
                pygame.time.wait(500)
                continue
            except sr.WaitTimeoutError:
                continue

            print("🔄 Recognizing...")

            try:
                word = recognizer.recognize_google(audio).lower()
            except sr.UnknownValueError:
                print("❌ Could not understand audio.")
                continue
            except sr.RequestError as e:
                print(f"Speech service error: {e}")
                continue

            print(f"🗣️ You said: {word}")

            if "hey jarvis" in word or "jarvis" in word:
                ready_responses = [
                    "Sir, Jarvis is ready for command! What's on your mind?",
                    "Ready and waiting! Give me your command.",
                    "Jarvis here! How can I help you today?",
                    "I'm listening, sir! What would you like me to do?"
                ]
                speak(random.choice(ready_responses))

                try:
                    with sr.Microphone() as source:
                        print("🎤 Listening for your command...")
                        recognizer.adjust_for_ambient_noise(source, duration=0.5)
                        audio = recognizer.listen(source, timeout=10, phrase_time_limit=8)
                except OSError as e:
                    print(f"Microphone error: {e}")
                    continue
                except sr.WaitTimeoutError:
                    print("⏰ Listening timed out.")
                    continue

                try:
                    command = recognizer.recognize_google(audio).lower()
                except sr.UnknownValueError:
                    speak("Sorry sir, I didn't get that. Could you please repeat?")
                    continue
                except sr.RequestError as e:
                    print(f"Speech service error: {e}")
                    continue

                print(f"📝 Command: {command}")
                processCommand(command)

        except KeyboardInterrupt:
            speak("Goodbye sir! It was nice chatting with you. See you soon!")
            print("\n👋 Exiting Jarvis...")
            break

        except Exception as e:
            print(f"❌ Error: {e}")
```
