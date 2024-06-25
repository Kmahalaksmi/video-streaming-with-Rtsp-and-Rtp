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














 #frontend
 <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Video Streaming with RTSP and RTP</title>
</head>
<body>
    <h1>Video Streaming with RTSP and RTP</h1>
    <video id="video-stream" controls autoplay></video>
    
    <script>
        const videoElement = document.getElementById('video-stream');
        
        // Function to fetch video stream from backend
        function fetchVideoStream() {
            fetch('/video')
                .then(response => {
                    if (!response.ok) {
                        throw new Error('Network response was not ok');
                    }
                    return response.body;
                })
                .then(body => {
                    const reader = body.getReader();
                    let decoder = new TextDecoder();
                    let partialData = '';
                    let chunks = [];

                    reader.read().then(function processText({ done, value }) {
                        if (done) {
                            return;
                        }
                        let chunk = partialData + decoder.decode(value, { stream: true });
                        partialData = '';
                        let lines = chunk.split('\r\n');
                        for (let i = 0; i < lines.length - 1; i++) {
                            if (lines[i] === '--frame') {
                                if (chunks.length > 0) {
                                    let blob = new Blob(chunks, { type: 'image/jpeg' });
                                    const url = URL.createObjectURL(blob);
                                    videoElement.src = url;
                                    chunks = [];
                                }
                            } else {
                                chunks.push(new Uint8Array(Buffer.from(lines[i] + '\r\n', 'utf-8')));
                            }
                        }
                        partialData = lines[lines.length - 1];
                        return reader.read().then(processText);
                    });
                })
                .catch(error => console.error('Error fetching video stream:', error));
        }

        // Call fetchVideoStream function when page loads
        window.onload = fetchVideoStream;
    </script>
</body>
</html>

