# video-streaming-with-Rtsp-and-Rtp
from flask import Flask, Response
import cv2
app = Flask(__name__)
@app.route('/video')
def video_feed():
    cap = cv2.VideoCapture('https://www.youtube.com/watch?v=OxAvxsL08Cc')  # Replace with your RTSP URL
    if not cap.isOpened():
        return Response("Error: Could not open video stream.", status=500)
    def generate():
        while cap.isOpened():
            ret, frame = cap.read()
            if not ret:
                break
            ret, jpeg = cv2.imencode('.jpg', frame)
            frame_bytes = jpeg.tobytes()
            yield (b'--frame\r\n'
                   b'Content-Type: image/jpeg\r\n\r\n' + frame_bytes + b'\r\n')
    return Response(generate(), mimetype='multipart/x-mixed-replace; boundary=frame')
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
