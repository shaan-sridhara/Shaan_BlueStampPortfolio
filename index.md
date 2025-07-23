# Smart Glasses

The Smart Glasses are a wearable vision system that detects specific objects in real time using a Raspberry Pi 5 and a camera module. Users can type in the name of an object they want to track, and the glasses will alert them with a buzzer when that object is confidently recognized by a TensorFlow Lite model. To improve power efficiency, a mechanical switch made from a piece of balsam wood is connected to the hinge of the glasses. When the glasses are closed, the switch presses a button that powers the system on, and when opened, it turns the system off. This ensures that the glasses only use power when they are being worn. The camera is mounted securely on the frame to provide a stable forward-facing view, making the Smart Glasses accurate, efficient, and user-friendly.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Shaan S | Bret Harte | Biomedical Engineering | Incoming 8th Grader

![Headstone Image](image_2025-06-17_161621492.png)
  
# 3rd Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/9kyJQXgnGEQ?si=9zepytVPRucwKMnC" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Modification 2

![Modification 2](IMG_0286.jpg)

I use the google.generativeai library to connect to Gemini, Google's AI model that can understand both images and text. I authenticate with my API key and use the "gemini-1.5-flash" model because it's the current supported version. When I press the button, my Pi Camera takes a photo, and I send that image to Gemini with a prompt asking it to solve any problems shown. Gemini reads the image, processes the prompt, and returns the final answers, which I print out in the terminal.

### Code

```Python
import os
import time
from datetime import datetime
from gpiozero import Button
from PIL import Image
from picamera2 import Picamera2
import cv2
import google.generativeai as genai

# === Gemini API Setup ===
GEMINI_API_KEY = "AIzaSyAjytjD0_9yrHoKdUKJqnM17XhO4nbNbdc"
genai.configure(api_key=GEMINI_API_KEY)
model = genai.GenerativeModel("gemini-1.5-flash")

# === Setup Output Directory ===    
save_dir = "Pictures"
os.makedirs(save_dir, exist_ok=True)

# === Setup PiCamera2 ===
picam2 = Picamera2()
camera_config = picam2.create_preview_configuration(main={"size": (640, 480)})
picam2.configure(camera_config)
picam2.start()
time.sleep(2)

# === Button Setup ===
button = Button(20, pull_up=True, bounce_time=0.3)

# === Picture & Gemini Response Function ===
def take_picture():
    try:
        print("[INFO] Button pressed. Capturing image...")
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        filename = os.path.join(save_dir, f"image_{timestamp}.png")

        image = picam2.capture_array()
        image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
        cv2.imwrite(filename, image)
        print(f"[INFO] Picture saved to: {filename}")

        with Image.open(filename) as img:
            prompt = (
                "Solve the problem in this image. Only give the final answer. "
                "If there are multiple problems separate the answers by a comma. "
                "If there isn't a problem in the image just tell me what you see."
            )
            response = model.generate_content([prompt, img])
            result = response.text.strip()
            print(f"[Gemini Answer] {result}")
            os.system('amixer set Master 150%')
            os.system(f'espeak -a 200 \"{result}\" &')

        time.sleep(1)
    except Exception as e:
        print(f"[ERROR] {e}")

# === Bind Button to Function ===
button.when_pressed = take_picture

# === Main Loop ===
print("[READY] Waiting for button press on GPIO 20...")
while True:
    time.sleep(1)

```

This code sets up a Raspberry Pi with a camera and button to take a photo when the button (on GPIO 20) is pressed. The captured image is sent to Google's Gemini AI, which analyzes the image and responds with a short answer or description. The result is then spoken aloud using the espeak text-to-speech engine.

### Challenges

Some challenges I faced while creating this code included setting up the connection between Gemini and my computer. It was difficult because I had to install and configure several libraries in VS Code to get the code to communicate properly with Gemini. Another issue was that the text-to-speech (TTS) output was very quiet after a picture was taken. Making the audio louder was tricky and required experimenting with different system settings and volume controls before it finally worked.

### Next Steps

My next step is to design and develop a website that will serve as a central platform for users to easily access and interact with the software used by the smart glasses. This website will streamline the user experience by allowing quick access to features like image capture, AI analysis, and system settings, all in one place. It will also provide helpful documentation, updates, and support to ensure users can operate the glasses without needing to interact directly with the code. By creating this web interface, I hope to make the technology more user-friendly and accessible to a wider audience.

## Modification 1

![Headstone Image](image_2025-07-10_131349179.png)

I implemented a mechanical power management solution for my smart glasses by incorporating a tactile button on the side frame. A custom-cut piece of balsam wood was affixed to the hinge mechanism such that, upon closing the glasses, the wood actuates the button, thereby powering the device on. When the glasses are opened, the button is released, cutting power and placing the device in a low-energy state. This mechanical activation mechanism effectively prevents continuous operation, significantly reducing unnecessary power consumption. Given the limited capacity of the portable power bank supplying the device, this design optimizes energy usage and extends operational longevity by ensuring the system remains active only when physically engaged.

### Code

```Python
import os
import subprocess
import RPi.GPIO as GPIO
import time
import ast

button_pin = 27
buzzer_pin = 21

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)
GPIO.setup(button_pin, GPIO.IN, pull_up_down=GPIO.PUD_UP)
GPIO.setup(buzzer_pin, GPIO.OUT)

target_label = input("Enter the object label to detect (e.g., laptop): ").strip().lower().replace(" ", "_")

project_dir = '/home/shaan/rpi-vision'
python_bin = '/home/shaan/Documents/env/bin/python'
env = os.environ.copy()
env['PYTHONPATH'] = project_dir
os.chdir(project_dir)

print("Current directory:", os.getcwd())
print(f"Will detect: {target_label}")
print("Hold the button to start detection...")

CONFIDENCE_THRESHOLD = 0.6
model_process = None
detection_active = False

try:
    while True:
        button_pressed = GPIO.input(button_pin) == GPIO.LOW

        if button_pressed and model_process is None:
            print("🟢 Button held — starting model...")
            model_process = subprocess.Popen(
                [python_bin, 'tests/pitft_labeled_output.py', '--tflite'],
                env=env,
                stdout=subprocess.PIPE,
                stderr=subprocess.STDOUT,
                text=True
            )

        elif not button_pressed and model_process is not None:
            print("🔴 Button released — stopping model...")
            model_process.terminate()
            model_process.wait()
            model_process = None
            detection_active = False
            GPIO.output(buzzer_pin, GPIO.LOW)

        if model_process:
            line = model_process.stdout.readline()
            if not line:
                continue

            print("📤 MODEL:", line.strip())

            if line.startswith("INFO:root:[('"):
                try:
                    detections_str = line.split("INFO:root:")[1].strip()
                    detections = ast.literal_eval(detections_str)

                    for wnid, label, conf in detections:
                        print(f"🔍 Detected: {label} with confidence {conf:.2f}")

                        if label.lower() == target_label and conf >= CONFIDENCE_THRESHOLD:
                            print("✅ Match found — buzzing!")
                            GPIO.output(buzzer_pin, GPIO.HIGH)
                            time.sleep(0.2)
                            GPIO.output(buzzer_pin, GPIO.LOW)
                            detection_active = True
                            break
                        else:
                            print("⛔ Not a match or confidence too low")

                except Exception as e:
                    print("⚠️ Error parsing detection line:", e)

        time.sleep(0.05)

except KeyboardInterrupt:
    print("🛑 Stopped by user.")
    if model_process:
        model_process.terminate()
        model_process.wait()

finally:
    GPIO.cleanup()
    print("✅ GPIO cleaned up.")
```

This code uses a button on GPIO 27 to control when a TensorFlow Lite object detection model runs on a Raspberry Pi, and a buzzer on GPIO 21 to signal when the specified object (e.g., "laptop") is detected with high confidence. When the button is held, the model starts; when released, the model stops and the buzzer turns off. If the detected object's label matches the user's input and its confidence is above 0.6, the buzzer briefly activates to signal a match.

### Challenges 

One challenge I faced during my first modification was attaching the button securely to the glasses. The materials involved didn’t bond well with most adhesives, making it difficult to find a reliable solution. Eventually, I used Clear Adhesive Sealant, which worked extremely well and kept the button firmly in place. Another challenge was setting up the TensorFlow model so that it could be turned on and off through the code. This required figuring out how to control the model's execution based on external input. After some troubleshooting, I was able to get it working as intended.

### Next Steps

The next phase of my project will focus on implementing my second modification, which uses Google's Gemini AI to analyze and solve math problems directly from images. When a user presses a button, the camera will capture a photo of the problem, and Gemini will interpret the content, calculate the correct answers, and return the solution. This modification adds powerful educational functionality to the smart glasses, turning them into a hands-free math assistant. It’s a big step toward making the device more interactive and helpful in real-world learning environments.

# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/KUQzLhIjmnM?si=WQE3oEMpKa3BfVY6" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Since my first milestone, I have made significant and meaningful progress on both the hardware and software components of my project. One of the major improvements was physically attaching the Raspberry Pi Camera to the glasses in a secure and well-aligned position. This step was crucial for making the system wearable and functional in real-world use. On the software side, I successfully set up a live video stream from the camera, which included solving technical issues like fixing color distortion and configuring the camera settings through the PiCamera2 library. I also began working on the object detection aspect by preparing to integrate a TensorFlow Lite model, which will allow the system to identify objects in real time. These improvements have moved me much closer to my ultimate goal of developing an AI-powered wearable vision system, and they provide a strong foundation for the next phase of my work.

This is a diagram of my Second Milestone

![Second Milestone Setup](image_2025-06-26_153834715.png)

## Challenges

One of the main challenges I faced during this phase of the project was figuring out the correct mount for attaching the camera securely to the glasses. It took several attempts to find a position that was both stable and aligned well enough to capture a clear forward-facing view without interfering with the user's vision. Another major challenge was getting the object detection code to work properly. Setting up the TensorFlow Lite model involved dealing with compatibility issues, understanding how to preprocess frames correctly, and making sure the model could run efficiently on the Raspberry Pi without slowing down the video stream. Troubleshooting these problems required a lot of trial and error, but ultimately helped me better understand how real-time computer vision systems function.

## Code
```Python
from flask import Flask, Response, render_template_string
from picamera2 import Picamera2
import cv2

app = Flask(__name__)

# Set up camera
picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(
    main={"format": "RGB888", "size": (640, 480)}
))
picam2.start()
picam2.set_controls({"AwbMode": 1})  # Enable auto white balance

# HTML for the camera stream
HTML = """
<!doctype html>
<title>Pi Camera Stream</title>
<h1>Live Stream from Raspberry Pi Camera</h1>
<img src="/video_feed">
"""

def generate_frames():
    while True:
        frame = picam2.capture_array()
        #frame = cv2.cvtColor(frame, cv2.COLOR_RGB2BGR)  # Convert to correct format
        ret, buffer = cv2.imencode('.jpg', frame)
        if not ret:
            continue
        frame = buffer.tobytes()
        yield (b'--frame\r\n'
               b'Content-Type: image/jpeg\r\n\r\n' + frame + b'\r\n')

@app.route('/')
def index():
    return render_template_string(HTML)

@app.route('/video_feed')
def video_feed():
    return Response(generate_frames(), mimetype='multipart/x-mixed-replace; boundary=frame')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

This code establishes a live video stream that captures real-time footage using the Raspberry Pi Camera and displays it through a web browser. By setting up a Flask server and using OpenCV to process each video frame, the stream can be accessed over a network, making it easy to monitor the camera feed remotely. This functionality is essential for testing and developing computer vision applications, as it provides a continuous and accessible view of what the camera is seeing. It also serves as the foundation for integrating more advanced features like object detection.

### Next Steps

My next steps involve implementing two major modifications to improve the functionality of the system. The first is adding a power-saving feature that allows the model to be turned on and off with the press of a button, which will help avoid unnecessary detections and conserve energy when not in use. The second modification integrates Gemini AI to automatically solve math problems by analyzing a picture taken with the camera. Once the problem is solved, the system will use text-to-speech to read the answer out loud, making it more interactive and useful in real-time scenarios. These enhancements will make the device smarter, more efficient, and easier to use.

# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/hwgU-7iSydI?si=aCjqJ5J-OpynQL5M" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

My project is called Smart Glasses. The goal is to help people, especially those who can’t see well, by using AI to detect objects around them. I’m building it using a Raspberry Pi 5 and a camera that attaches to it. These parts will go on a glasses frame. The camera takes video of what’s in front of the person, and the AI will figure out what the objects are. Then, the glasses will say what it sees out loud using a speaker so the person knows what’s around them.

So far, I was able to take a picture using the Raspberry Pi and the camera. At first, I tried to set it up without using a monitor or keyboard, but it was really hard and didn’t work for me. Instead, I decided to just plug it into a monitor and use a keyboard and mouse, which made it easier to work on.

My plan now is to find a database of pictures to help train the AI to recognize different objects. Once I get that working, I’ll connect it to a text-to-speech program so the glasses can tell the person what it sees. That way, the Smart Glasses will be able to help people understand what’s around them without needing to see it.

This is my first milestone video:


![Milestone_Setup](Screenshot 2025-06-20 161814.png)

## Challenges

One of the main challenges I faced during Milestone 1 was setting up the Raspberry Pi for a headless configuration, which means accessing and controlling the Pi remotely without a monitor or keyboard attached. This setup is very convenient because it allows you to program the Raspberry Pi from another computer over a network. However, due to slow and unstable Wi-Fi, I had trouble maintaining a consistent connection, which made the setup process difficult and sometimes impossible to complete. This experience taught me how important a reliable network is for remote device management and pushed me to troubleshoot connectivity issues while learning more about networking and remote access tools like SSH.

## Next Steps

The next steps for this project are to get the camera module working so it can detect and recognize different objects by using a database of known items. This means setting up software that can compare what the camera sees with stored information to identify objects accurately. At the same time, I plan to set up a continuous video stream so that the camera feed can be viewed and analyzed in real time. Having this live video combined with object detection will let the system recognize and classify objects as they come into view, making it more interactive and useful. To do this, I’ll need to explore different machine learning models, figure out how to run them efficiently on the Raspberry Pi, and make sure the camera, detection program, and database all work smoothly together. These steps will really push the project forward and help build a smart vision system that can respond to what it sees.

## Code
```Python
from picamera2 import Picamera2, Preview
import time
import cv2
picam2 = Picamera2()
camera_config = picam2.create_still_configuration(main={"size": (1920, 1080)},
lores={"size": (640, 480)}, display="lores")
picam2.configure(camera_config)
#picam2.start_preview(Preview.QTGL) #Comment this out if not using desktop interface
picam2.start()
time.sleep(2)
im = picam2.capture_array()
im = cv2.cvtColor(im, cv2.COLOR_BGR2RGB)
cv2.imwrite('file.png', im)
```
This code initializes the Raspberry Pi Camera using the Picamera2 library, configures it to capture a still image at a resolution of 1920x1080, and starts the camera. After a brief delay to allow the camera to adjust, it captures an image, converts its color format from BGR to RGB using OpenCV, and saves the image as "file.png".

[comment]: <> (# Schematics)

[comment]: <> (work in progress)

# Code

```Python
import os
import cv2
import time
import tempfile
import threading
import subprocess
from flask import Flask, Response, request, jsonify
from gtts import gTTS
import google.generativeai as genai
from datetime import datetime
from PIL import Image
import speech_recognition as sr
from picamera2 import Picamera2

# Configure your Gemini API key here
GEMINI_API_KEY = "AIzaSyAjytjD0_9yrHoKdUKJqnM17XhO4nbNbdc"
genai.configure(api_key=GEMINI_API_KEY)
model = genai.GenerativeModel("gemini-1.5-flash")

# Initialize camera
picam2 = Picamera2()
picam2.configure(picam2.create_preview_configuration(main={"size": (640, 480)}))
picam2.start()
time.sleep(2)

# Initialize Flask app
app = Flask(__name__)

# Initialize speech recognizer and mic
recognizer = sr.Recognizer()
mic = sr.Microphone(device_index=1)  # Change device_index as needed

save_dir = "Pictures"
os.makedirs(save_dir, exist_ok=True)

def speak_text(text):
    try:
        tts = gTTS(text=text, lang='en')
        with tempfile.NamedTemporaryFile(delete=False, suffix=".mp3") as fp:
            tts.save(fp.name)
            subprocess.run(["mpg123", "-q", fp.name])
            os.remove(fp.name)
    except Exception as e:
        print(f"[TTS ERROR] {e}")

def take_picture():
    try:
        print("[INFO] Capturing image...")
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        filename = os.path.join(save_dir, f"image_{timestamp}.png")

        image = picam2.capture_array()
        image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
        cv2.imwrite(filename, image)
        print(f"[INFO] Picture saved to: {filename}")

        with Image.open(filename) as img:
            prompt = (
                "Solve the problem in this image. Only give the final answer. "
                "If there are multiple problems separate the answers by a comma. "
                "If there isn't a problem in the image just tell me what you see."
            )
            response = model.generate_content([prompt, img])
            result = response.text.strip()
            print(f"[Gemini Answer] {result}")

            os.system('amixer set Master 150%')
            speak_text(result)

        time.sleep(1)
    except Exception as e:
        print(f"[ERROR] {e}")

def gen_frames():
    while True:
        frame = picam2.capture_array()
        frame = cv2.cvtColor(frame, cv2.COLOR_RGB2BGR)
        ret, buffer = cv2.imencode('.jpg', frame)
        if not ret:
            continue
        yield (b'--frame\r\n'
               b'Content-Type: image/jpeg\r\n\r\n' + buffer.tobytes() + b'\r\n')

@app.route("/")
def index():
    return '''
    <html>
    <head>
        <title>Smart Glasses Control</title>
        <style>
            body { font-family: Arial, sans-serif; }
            #transcript { margin-top: 10px; font-weight: bold; }
        </style>
    </head>
    <body>
        <h1>Live Camera Feed</h1>
        <img src="/video_feed" width="640" height="480"><br><br>

        <h2>Send Audio Message</h2>
        <button id="record-btn">Hold to Record</button>
        <p id="status"></p>
        <p>Transcription:</p>
        <textarea id="transcript" rows="4" cols="50"></textarea><br>
        <button id="send-btn" disabled>Send Text</button>
        <button id="rerecord-btn" disabled>Re-record</button>

        <script>
            let mediaRecorder;
            let audioChunks = [];
            const recordBtn = document.getElementById('record-btn');
            const status = document.getElementById('status');
            const transcriptArea = document.getElementById('transcript');
            const sendBtn = document.getElementById('send-btn');
            const rerecordBtn = document.getElementById('rerecord-btn');

            recordBtn.addEventListener('mousedown', async () => {
                status.textContent = 'Recording...';
                audioChunks = [];
                transcriptArea.value = "";
                sendBtn.disabled = true;
                rerecordBtn.disabled = true;

                const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
                mediaRecorder = new MediaRecorder(stream);
                mediaRecorder.start();

                mediaRecorder.ondataavailable = e => {
                    audioChunks.push(e.data);
                };

                mediaRecorder.onstop = async () => {
                    status.textContent = 'Uploading...';
                    const audioBlob = new Blob(audioChunks, { type: 'audio/webm' });
                    const formData = new FormData();
                    formData.append('audio_data', audioBlob, 'recording.webm');

                    try {
                        const response = await fetch('/upload_audio', {
                            method: 'POST',
                            body: formData
                        });

                        if (!response.ok) {
                            const errorText = await response.text();
                            console.error("Upload error:", errorText);
                            status.textContent = 'Server Error: Unable to process audio.';
                            return;
                        }

                        const data = await response.json();
                        status.textContent = 'Done';
                        transcriptArea.value = data.transcript_original;
                        sendBtn.disabled = false;
                        rerecordBtn.disabled = false;
                    } catch (err) {
                        console.error(err);
                        status.textContent = 'Error: ' + err.message;
                    }
                };
            });

            recordBtn.addEventListener('mouseup', () => {
                if (mediaRecorder && mediaRecorder.state === 'recording') {
                    mediaRecorder.stop();
                }
            });

            recordBtn.addEventListener('mouseleave', () => {
                if (mediaRecorder && mediaRecorder.state === 'recording') {
                    mediaRecorder.stop();
                }
            });

            sendBtn.addEventListener('click', () => {
                const editedTranscript = transcriptArea.value.trim();
                if (editedTranscript) {
                    fetch('/send_text', {
                        method: 'POST',
                        headers: {'Content-Type': 'application/json'},
                        body: JSON.stringify({text: editedTranscript})
                    });
                    status.textContent = 'Text sent!';
                    sendBtn.disabled = true;
                    rerecordBtn.disabled = true;
                }
            });

            rerecordBtn.addEventListener('click', () => {
                transcriptArea.value = "";
                status.textContent = 'Re-record and press the button.';
                sendBtn.disabled = true;
                rerecordBtn.disabled = true;
            });
        </script>
    </body>
    </html>
    '''

@app.route('/video_feed')
def video_feed():
    return Response(gen_frames(), mimetype='multipart/x-mixed-replace; boundary=frame')

@app.route('/upload_audio', methods=['POST'])
def upload_audio():
    if 'audio_data' not in request.files:
        return jsonify({"error": "No audio file uploaded"}), 400

    audio_file = request.files['audio_data']
    with tempfile.NamedTemporaryFile(delete=False, suffix=".webm") as temp_audio:
        temp_audio.write(audio_file.read())
        temp_webm_path = temp_audio.name

    temp_wav_path = temp_webm_path.replace(".webm", ".wav")

    # Convert webm to wav for transcription
    try:
        subprocess.run(
            ["ffmpeg", "-y", "-i", temp_webm_path, temp_wav_path],
            stdout=subprocess.DEVNULL,
            stderr=subprocess.DEVNULL,
            check=True
        )
    except subprocess.CalledProcessError:
        os.remove(temp_webm_path)
        return jsonify({"error": "Audio conversion failed"}), 500

    # Transcribe audio (original language)
    try:
        r = sr.Recognizer()
        with sr.AudioFile(temp_wav_path) as source:
            audio = r.record(source)
        original_transcript = r.recognize_google(audio)
    except Exception as e:
        os.remove(temp_webm_path)
        os.remove(temp_wav_path)
        return jsonify({"error": f"Transcription error: {e}"}), 500

    # Translate silently to English using Gemini and play TTS
    english_translation = None
    try:
        translate_prompt = (
            f"Translate the following text to English ONLY, no extra commentary:\n"
            f"'''{original_transcript}'''"
        )
        response = model.generate_text(
            prompt=translate_prompt,
            temperature=0,
            max_output_tokens=256,
        )
        english_translation = response.text.strip()
    except Exception as e:
        print(f"[Gemini Translation Error] {e}")
        english_translation = None

    if english_translation:
        speak_text(english_translation)

    os.remove(temp_webm_path)
    os.remove(temp_wav_path)

    return jsonify({
        "transcript_original": original_transcript,
    })

@app.route('/send_text', methods=['POST'])
def send_text():
    data = request.get_json()
    text = data.get("text", "")
    if text:
        print(f"[Text sent from web UI] {text}")
        speak_text(text)
    return ('', 204)

def run_flask():
    app.run(host='0.0.0.0', port=5000)

if __name__ == "__main__":
    threading.Thread(target=run_flask, daemon=True).start()

    print("[READY] Say 'take a picture' to capture and analyze an image.")
    while True:
        with mic as source:
            recognizer.adjust_for_ambient_noise(source)
            print("🎤 Listening...")
            try:
                audio = recognizer.listen(source, timeout=10)
            except sr.WaitTimeoutError:
                continue

        try:
            command = recognizer.recognize_google(audio).lower()
            print(f"[HEARD] {command}")
            if "take a picture" in command:
                take_picture()
        except sr.UnknownValueError:
            print("[INFO] Could not understand audio.")
        except sr.RequestError as e:
            print(f"[ERROR] Google Speech API error: {e}")
        except Exception as e:
            print(f"[ERROR] {e}")
```

[comment]: <> (work in progress)

# Bill of Materials
| Part                      | Note                                              | Price | Link                                                                                                     |
|---------------------------|---------------------------------------------------|-------|----------------------------------------------------------------------------------------------------------|
| Raspberry Pi 5            | Latest Raspberry Pi model                         | $75   | [Buy here](https://www.raspberrypi.org/products/raspberry-pi-5/)                                         |
| Pi Camera Module          | Official camera for Raspberry Pi                 | $30   | [Buy here](https://www.raspberrypi.org/products/camera-module-v3/)                                       |
| Non-prescription Glasses  | Black rectangular fashion glasses                | $12   | [Buy here](https://www.amazon.com/GQUEEN-201512-Fashion-Rectangular-Glasses/dp/B00ZRD1MEI/)              |
| Push Button x2            | Tactile buttons for triggering actions            | $1.50 | [Buy here](https://www.adafruit.com/product/367)                                                         |
| Jumper Wires (Male-Female)| For GPIO connections to buttons, camera, etc.     | $3.00 | [Buy here](https://www.adafruit.com/product/1956)                                                        |
| Balsa Wood (Small Sheet)  | Lightweight material for mounting components      | $4.00 | [Buy here](https://www.amazon.com/dp/B000BQW55C/)                                                        |


[comment]: <> (# Other Resources/Examples)

[comment]: <> (work in progress)

# Starter Project
This is the retro arcade console I built for this project, using the VOGURTIME Electronic Soldering Practice Kit. The kit came with pre-manufactured components, including a mini circuit board, joystick, LED display, and tactile buttons. The goal was to assemble a fully functional mini arcade game system by carefully soldering all the components to the PCB. Once completed, the arcade console lights up, plays sound, and allows simple interactive gameplay, making it a fun and practical introduction to basic electronics.

This is a [link](https://www.amazon.com/Electronic-Soldering-Practice-Comfortable-VOGURTIME/dp/B094QRRHC2/ref=sr_1_3?asc_source=01H2RCFWNNZMQFGXGXS3RMXVE5&crid=12C0SOV36FG6M&dib=eyJ2IjoiMSJ9.Prj06eg0mzBHrfW8zuFr43Ott4t2wUOVBo8A8bYw0PqFZRlOEmgR5YwhMy7jXrdI2HlBjVttnEyYLz5CP684SzJyHmVMBp25vNna9o8wjV-df55ilTgj0xMy1CiRwkcnu6xqacZ3JUPlq8C3mQJwmEtoeokndNqpwpdkZBQMplM9vg3M-cfB0xM_nXdjeqHQ3bB707ehrzX6Llp-Euu3CTFzF8wgEqhPwo6RCvzbo5M.yyrFg8EXJr9BL5cOgZF551-8cIl91p0MSy8nGiilcpU&dib_tag=se&keywords=arcade%2Bsolder%2Bproject&qid=1717994267&sprefix=arcade%2Bsolder%2Bprojec%2Caps%2C147&sr=8-3&tag=snxs3-20&th=1) for the Retro Arcade Console:

This is a video of me demoing the Retro Arcade Console:

<iframe width="560" height="315" src="https://www.youtube.com/embed/FWgXDVm8kqY?si=U474cKpsJ89gVsbU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Challenges
One of the biggest challenges I faced was soldering the components precisely. Many of the soldering points were small and tightly packed, which required a steady hand and a lot of focus. It took some trial and error to get the hang of it, but I eventually learned that good soldering comes down to patience, attention to detail, and proper technique. Overall, this was a great project for learning hands-on skills in electronics through a fun and engaging experience.

## Next Steps
My next steps for this project involve beginning the initial phase of my intensive project. This first milestone will focus on assembling and configuring the core hardware components: a Raspberry Pi 5 and the official Pi Camera Module. I will begin by carefully connecting the camera to the Raspberry Pi using the appropriate ribbon cable and ensuring the hardware is properly secured and recognized by the system.

Once the hardware is set up, I will move on to the software configuration. This includes enabling the camera interface through the Raspberry Pi OS settings and installing any necessary libraries or dependencies required to operate the camera. I will be using Visual Studio Code as my development environment to write and test Python scripts that allow the camera to capture still images.

The goal of this milestone is to successfully capture a still photo using the Pi Camera, laying the groundwork for more advanced features later in the project. This step is critical as it ensures that both the hardware and software components are functioning correctly before progressing to more complex parts of my project.

