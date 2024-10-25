---
title: Map APIs and How To Add Custom Markers 
date: 2024-10-18 09:42:00 -500
categories: [APIs, gist]
tags: [flutter, for beginners, recommendations]
image: "https://raw.githubusercontent.com/cristina22999/cristina22999.github.io/refs/heads/main/assets/img/emoji_markers.png"
---


There are so many applications out there that, in one way or another, make use of the map functionality. And when it comes to adding maps to a Flutter app, the number of API options available may surprise you; from Flutter Maps to Google Maps, and even OpenStreetMap integrations. Despite its costs, I prefer Google Maps, but let’s see a breakdown.

## Flutter Maps vs. Google Maps vs. Other Map APIs

### Flutter Maps (`flutter_map`)
Flutter Maps is a versatile, open-source map API that works well for basic or small-scale. It supports a range of providers like OpenStreetMap, Mapbox, and Bing Maps. The biggest benefit? It’s free (and easy to use). However, advanced features like custom markers, real-time traffic data, or smooth animations, which make your app look professional and well-rounded, are unfortunately not supported.

### Google Maps
Google Maps is, by far, one of the most robust and widely used map APIs for Flutter. It offers real-time traffic updates, street view, and satellite imagery. It is easy to use and has the best and most detailed global data accuracy. 

Most importantly, it allows you to add custom markers with a lot of flexibility—whether you're adding images, emojis, or even custom text to your markers. Unfortunately, Flutter doesn’t offer a direct method to do this easily, so keep reading for my workaround and example code.

The downside? You’ll need to set up billing since it’s a paid service after a certain threshold - the Maps SDK offers 100,000 free map loads per month, which in my experience is enough for small-scale projects.

> Top Tip: it is also cross-platform, so the performance across iOS and Android makes the cost worthwhile. For this, make sure to use [platform_maps_flutter](https://pub.dev/documentation/platform_maps_flutter/latest/). You can also pair it up with the [OpenCage Geocode API](https://opencagedata.com/api#quickstart) to allow users to easily search for places, and to convert coordinates to a String {city}, {country}.
{: .prompt-tip }


### Other Map APIs
Mapbox is another popular alternative, offering beautiful visual styling and customization options. It’s ideal if you need highly customizable map tiles or specific geographical data. 


![Desktop View](/assets/img/emoji_markers.png.png){: width="14vw" height="21vw" }
![Desktop View](/assets/img/pics_markers.png.png){: width="14vw height="21vw" }
![Desktop View](/assets/img/more_pics_markers.png.png){: width="14vw height="21vw" }
![Desktop View](/assets/img/black_pics_markers.png.png){: width="14vw height="21vw" }


## How To Add Custom Markers to Google Maps in Flutter

Now, let’s get to the fun part: customizing your markers! Whether you want to add emojis, images, or even custom text, here’s a solution that will get the job done.

```dart
import 'package:flutter/material.dart';
import 'dart:ui' as ui;
import 'package:flutter/services.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';

// Function to generate BitmapDescriptor without text overlay
Future<BitmapDescriptor> textOrImageToBitmapDescriptorWithText(
  // Dynamic input allws us to paint image, emoji and String
  dynamic input, {
  TextStyle? textStyle,
  int width = 300,
  int height = 300,
}) async {
  final ui.PictureRecorder recorder = ui.PictureRecorder();
  final Canvas canvas = Canvas(recorder);

  // Check if the input is an image path (supports .png, .jpg, .jpeg)
  if (input is String &&
      (input.endsWith('.png') ||
          input.endsWith('.jpg') ||
          input.endsWith('.jpeg'))) {
    final ByteData data = await rootBundle.load(input);
    final Uint8List bytes = data.buffer.asUint8List();

    final ui.Codec codec = await ui.instantiateImageCodec(bytes,
        targetWidth: width, targetHeight: height);
    final ui.FrameInfo frameInfo = await codec.getNextFrame();
    final ui.Image img = frameInfo.image;

    // Draw the image onto the canvas
    canvas.drawImageRect(
      img,
      Rect.fromLTWH(0, 0, img.width.toDouble(), img.height.toDouble()),
      Rect.fromLTWH(0, 0, width.toDouble(), height.toDouble()),
      Paint(),
    );
  } else if (input is String) {
    // Assume input is text (e.g. an emoji or String)
    final TextPainter painter = TextPainter(
      textDirection: TextDirection.ltr,
      textAlign: TextAlign.center,
    );
    painter.text = TextSpan(
      text: input,
      style: textStyle ?? TextStyle(fontSize: 150.0, color: Colors.black),
    );
    painter.layout();
    painter.paint(canvas,
        Offset((width - painter.width) / 2, (height - painter.height) / 2));
  }

  // Convert the canvas into an image and create the BitmapDescriptor
  final ui.Image image = await recorder.endRecording().toImage(width, height);
  final ByteData? byteData =
      await image.toByteData(format: ui.ImageByteFormat.png);
  final Uint8List uint8List = byteData!.buffer.asUint8List();

  return BitmapDescriptor.bytes(uint8List);
}
```


For the full runnable example, visit my [gist](https://gist.github.com/cristinaponcela/6284430066db94ca9c733fc26ffdad59).
