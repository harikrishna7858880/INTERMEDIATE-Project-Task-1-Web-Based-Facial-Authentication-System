INTERMEDIATE Project:
Task 1 Web-Based Facial Authentication System

Description: The web face detector can also be designed to view faces in a video call. We need to get started with this project using an OpenCV and a webcam. OpenCV is a real-time computer vision tool. This project can be extended for use cases like user authentication in meetings, exams, police force, face unlock feature of phones, etc


To build a Web-Based Facial Authentication System with OpenCV, here’s how you can structure and execute this project:

Plan for the Project
Tech Stack:

Frontend: HTML, CSS, and JavaScript.
Backend: Python with Flask or FastAPI.
Libraries: OpenCV (for facial detection), dlib (for advanced facial features), and face_recognition (for comparing faces).
Basic Workflow:

Capture live video frames from a webcam.
Detect faces in the video stream.
Match detected faces with a pre-registered dataset for authentication.
Project Extensions:

Real-time face matching for video calls or meetings.
Secure storage and management of user face embeddings.
Step 1: Install Required Libraries
Install the necessary Python libraries:

bash
Copy code
pip install opencv-python opencv-python-headless face-recognition flask
Step 2: Create the Backend
Use Flask to create a backend server for handling face recognition logic.

Backend Code (server.py):
python
Copy code
import os
import cv2
import numpy as np
import face_recognition
from flask import Flask, request, jsonify

app = Flask(__name__)

# Path to store registered user face encodings
KNOWN_FACES_DIR = "known_faces"
os.makedirs(KNOWN_FACES_DIR, exist_ok=True)

def load_known_faces():
    known_faces = []
    known_names = []

    for filename in os.listdir(KNOWN_FACES_DIR):
        if filename.endswith(".jpg") or filename.endswith(".png"):
            image_path = os.path.join(KNOWN_FACES_DIR, filename)
            image = face_recognition.load_image_file(image_path)
            encoding = face_recognition.face_encodings(image)[0]
            known_faces.append(encoding)
            known_names.append(os.path.splitext(filename)[0])

    return known_faces, known_names

@app.route("/register", methods=["POST"])
def register_face():
    name = request.form.get("name")
    if not name:
        return jsonify({"error": "Name is required!"}), 400

    file = request.files.get("file")
    if not file:
        return jsonify({"error": "Image file is required!"}), 400

    # Save the image and process it
    image_path = os.path.join(KNOWN_FACES_DIR, f"{name}.jpg")
    file.save(image_path)

    return jsonify({"message": f"User {name} registered successfully!"})

@app.route("/authenticate", methods=["POST"])
def authenticate_face():
    file = request.files.get("file")
    if not file:
        return jsonify({"error": "Image file is required!"}), 400

    # Load known faces and names
    known_faces, known_names = load_known_faces()

    # Read uploaded image
    uploaded_image = face_recognition.load_image_file(file)
    uploaded_encoding = face_recognition.face_encodings(uploaded_image)

    if len(uploaded_encoding) == 0:
        return jsonify({"error": "No face detected in the uploaded image!"}), 400

    uploaded_encoding = uploaded_encoding[0]
    matches = face_recognition.compare_faces(known_faces, uploaded_encoding)
    if True in matches:
        matched_index = matches.index(True)
        return jsonify({"message": f"Authenticated as {known_names[matched_index]}!"})

    return jsonify({"error": "Authentication failed!"}), 401

if __name__ == "__main__":
    app.run(debug=True)
Step 3: Frontend for Capturing Webcam Input
Create a simple frontend interface to capture images from the webcam using JavaScript and send them to the backend.

Frontend Code (index.html):
html
Copy code
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Facial Authentication</title>
</head>
<body>
    <h1>Facial Authentication System</h1>
    <div>
        <video id="webcam" autoplay width="640" height="480"></video>
        <canvas id="snapshot" style="display: none;"></canvas>
        <button id="capture">Capture & Authenticate</button>
    </div>
    <script>
        const video = document.getElementById('webcam');
        const canvas = document.getElementById('snapshot');
        const captureButton = document.getElementById('capture');
        const context = canvas.getContext('2d');

        // Start webcam
        navigator.mediaDevices.getUserMedia({ video: true })
            .then(stream => {
                video.srcObject = stream;
            })
            .catch(err => {
                console.error('Error accessing webcam:', err);
            });

        // Capture image and send to server
        captureButton.addEventListener('click', async () => {
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            context.drawImage(video, 0, 0, canvas.width, canvas.height);

            const imageBlob = await new Promise(resolve => canvas.toBlob(resolve));
            const formData = new FormData();
            formData.append("file", imageBlob);

            fetch('/authenticate', { method: 'POST', body: formData })
                .then(response => response.json())
                .then(data => alert(data.message || data.error))
                .catch(err => console.error('Error:', err));
        });
    </script>
</body>
</html>
Step 4: Run the Application
Start the Flask Server:

bash
Copy code
python server.py
Open the Frontend: Serve index.html on a local HTTP server or directly open it in a browser.

Test the System:

Register a face by uploading an image through the backend /register route.
Use the webcam to authenticate against the registered faces.
Extensions and Use Cases
Real-Time Detection:

Use OpenCV to process live webcam streams for real-time facial detection and authentication.
Security Enhancements:

Encrypt and secure stored face encodings.
Add a timeout or session-based restrictions for repeated failed authentication attempts.
Integration:

Integrate with video call platforms (e.g., Zoom, MS Teams).
Use WebRTC for direct browser-based video stream processing.
Multi-Factor Authentication:

Combine facial recognition with OTP or password-based authentication for increased security.
