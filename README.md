
# Flutter Camera Image Encoding & Face Detection

This repository demonstrates how to open the camera in Flutter, configure the camera controller, perform face detection, and encode image formats. The project covers essential topics such as Flutter camera setup, image preview, face-detection in Flutter, and handling camera formats like NV21 and YUV_422_888.

## Features
1. Open camera in Flutter
2. Configure camera controller
3. Perform face detection
4. Encode image format

## Getting Started

### Prerequisites
- Flutter SDK
- Dart
- A compatible IDE (e.g., Android Studio, VS Code)

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/mostafizurrahman/flutter-camera-image-encoding.git
   cd flutter-camera-image-encoding-face-detection
   ```

2. **Install dependencies:**
   ```bash
   flutter pub get
   ```

### Running the App

1. **Connect a device or start an emulator.**
2. **Run the app:**
   ```bash
   flutter run
   ```

## Implementation Details

### 1. Open Camera in Flutter

Add the `camera` package to your `pubspec.yaml`:
```yaml
dependencies:
  flutter:
    sdk: flutter
  camera: ^0.10.0+4
  flutter_ml_vision: ^3.0.0
  image: ^3.0.1
```

### 2. Configure Camera Controller

Create a camera controller in your Dart code:

```dart
import 'package:camera/camera.dart';
import 'package:flutter/material.dart';

class CameraPage extends StatefulWidget {
  @override
  _CameraPageState createState() => _CameraPageState();
}

class _CameraPageState extends State<CameraPage> {
  CameraController? _controller;
  List<CameraDescription>? cameras;

  @override
  void initState() {
    super.initState();
    initializeCamera();
  }

  Future<void> initializeCamera() async {
    cameras = await availableCameras();
    _controller = CameraController(
      cameras![0],
      ResolutionPreset.high,
    );
    await _controller?.initialize();
    setState(() {});
  }

  @override
  void dispose() {
    _controller?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    if (_controller == null || !_controller!.value.isInitialized) {
      return Center(child: CircularProgressIndicator());
    }
    return CameraPreview(_controller!);
  }
}
```

### 3. Perform Face Detection

Use the `flutter_ml_vision` package for face detection:

```dart
import 'package:flutter_ml_vision/flutter_ml_vision.dart';

Future<void> detectFaces() async {
  final File imageFile = // Your image file
  final FirebaseVisionImage visionImage = FirebaseVisionImage.fromFile(imageFile);
  final FaceDetector faceDetector = FirebaseVision.instance.faceDetector();
  final List<Face> faces = await faceDetector.processImage(visionImage);

  for (Face face in faces) {
    final Rect boundingBox = face.boundingBox;
    // Do something with the detected face
  }
}
```

### 4. Encode Image Format

Use the `image` package to encode image formats:

```dart
import 'dart:io';
import 'package:image/image.dart' as img;

void encodeImage(File imageFile) {
  img.Image image = img.decodeImage(imageFile.readAsBytesSync())!;
  List<int> jpeg = img.encodeJpg(image, quality: 85);
  File('encoded_image.jpg')..writeAsBytesSync(jpeg);
}
```

## Example

Here’s an example that ties everything together:

```dart
import 'package:camera/camera.dart';
import 'package:flutter/material.dart';
import 'package:flutter_ml_vision/flutter_ml_vision.dart';
import 'dart:io';
import 'package:image/image.dart' as img;

class CameraApp extends StatefulWidget {
  @override
  _CameraAppState createState() => _CameraAppState();
}

class _CameraAppState extends State<CameraApp> {
  CameraController? _controller;
  List<CameraDescription>? cameras;

  @override
  void initState() {
    super.initState();
    initializeCamera();
  }

  Future<void> initializeCamera() async {
    cameras = await availableCameras();
    _controller = CameraController(
      cameras![0],
      ResolutionPreset.high,
    );
    await _controller?.initialize();
    setState(() {});
  }

  Future<void> captureAndDetectFaces() async {
    final imageFile = await _controller!.takePicture();
    final visionImage = FirebaseVisionImage.fromFile(File(imageFile.path));
    final faceDetector = FirebaseVision.instance.faceDetector();
    final List<Face> faces = await faceDetector.processImage(visionImage);

    for (Face face in faces) {
      final boundingBox = face.boundingBox;
      // Do something with the detected face
    }

    encodeImage(File(imageFile.path));
  }

  void encodeImage(File imageFile) {
    img.Image image = img.decodeImage(imageFile.readAsBytesSync())!;
    List<int> jpeg = img.encodeJpg(image, quality: 85);
    File('encoded_image.jpg')..writeAsBytesSync(jpeg);
  }

  @override
  void dispose() {
    _controller?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    if (_controller == null || !_controller!.value.isInitialized) {
      return Center(child: CircularProgressIndicator());
    }
    return Scaffold(
      appBar: AppBar(title: Text('Flutter Camera App')),
      body: CameraPreview(_controller!),
      floatingActionButton: FloatingActionButton(
        onPressed: captureAndDetectFaces,
        child: Icon(Icons.camera),
      ),
    );
  }
}

void main() => runApp(MaterialApp(home: CameraApp()));
```

## Contributing

Feel free to fork this repository and contribute by submitting a pull request.

## License

This project is licensed under the MIT License.
