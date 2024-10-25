---
title: Neumorphism in UI Design and How to Add Inner Shadows in Flutter
date: 2024-10-23 18:23:00 -500
categories: [Flutter, gist, UI Designs]
tags: [flutter, for beginners, recommendations, neumorphism]
image: "https://raw.githubusercontent.com/cristina22999/cristina22999.github.io/refs/heads/main/assets/img/neumorphic-ui-4.png"
---


## What is Neumorphism?
Neumorphism (or "new skeuomorphism") is the newest UI design trend. Through the use of shadows and highlights, it blends flat designs with a 3D aesthetic. It’s supposed to be satisfying. I call it the ASMR of UI Design - satisfying if you’re into it.

It is most commonly used on buttons, and evolved from skeumorphism to make digital elements come to life and be subtly 3D. This trend took off in 2020 as designers searched for fresh, minimalist aesthetics that you could nearly reach out and feel.

Several apps and platforms use neumorphism to create a visually soothing, user-friendly experience. You may have already encountered such designs:

![Desktop View](/assets/img/neumorphic-ui.png){: width="280vw" height="420vw" }{: .left}
![Desktop View](/assets/img/neumorphic-ui-2.png){: width="280vw" height="420vw" }{: .right}


---

## Implementing Neumorphism in Flutter with the Neumorphic Package

Flutter offers a Neumorphic package that makes it relatively easy to add neumorphic styles to your app. The package allows you to create soft shadows and highlights, add rounded edges, and build out raised or inset UI elements. You can adjust properties like `depth`, `lightSource`, and `color` to fine-tune the effect, achieving the classic neumorphic look.

This package is an excellent starting point if you’re aiming for a neumorphic aesthetic in your Flutter app. Here’s an example: 

```dart 
import 'package:flutter_neumorphic/flutter_neumorphic.dart'; 
NeumorphicButton( 
onPressed: () {}, 
style: NeumorphicStyle( 
depth: 8, 
color: Colors.grey[200], 
lightSource: LightSource.topLeft, 
), 
child: Text("Press Me"), 
)
```

## The Challenge of Adding Inner Shadows in Flutter

While the Neumorphic package is powerful, Flutter doesn’t have a straightforward way to add inner shadows. Inner shadows are essential for creating realistic "pressed" effects in neumorphic designs, but Flutter lacks a built-in method for this effect. Here is the workaround I usually use to achieve this effect:

![Desktop View](/assets/img/inner_shadow.png){: width="280vw" height="420vw" }

```dart
// Custom widget for adding inner shadow
class InnerShadow extends SingleChildRenderObjectWidget {
  const InnerShadow({
    Key? key,
    this.blur = 5, // Default blur value for shadow
    this.color = Colors.black38, // Default shadow color
    this.offset = const Offset(50, 50), // Default shadow offset
    Widget? child,
  }) : super(key: key, child: child);

  final double blur; // Amount of blur for the shadow
  final Color color; // Color of the shadow
  final Offset offset; // Offset of the shadow

  @override
  RenderObject createRenderObject(BuildContext context) {
    // Create and return the custom render object
    final renderObject = _RenderInnerShadow();
    updateRenderObject(context, renderObject);
    return renderObject;
  }

  @override
  void updateRenderObject(
      BuildContext context, _RenderInnerShadow renderObject) {
    // Update the render object with the new values of blur, color, and offset
    renderObject
      ..color = color
      ..blur = blur
      ..dx = offset.dx
      ..dy = offset.dy;
  }
}

// Custom render object class for inner shadow effect
class _RenderInnerShadow extends RenderProxyBox {
  double blur = 5; // Default blur for shadow
  Color color = Color(0xFFF15923); // Default color for shadow
  double dx = 5; // Default x-axis translation for shadow
  double dy = 5; // Default y-axis translation for shadow

  @override
  void paint(PaintingContext context, Offset offset) {
    // Ensure there is a child to paint
    if (child == null) return;

    // Calculate the outer and inner bounds of the shadow
    final Rect outerRect = offset & size;
    final Rect innerRect =
        outerRect.deflate(blur); // Adjust the inner rect for shadow

    final Canvas canvas = context.canvas;

    // Draw the child widget
    canvas.saveLayer(outerRect, Paint());
    context.paintChild(child!, offset);

    // Create the paint for the shadow layer
    final Paint shadowPaint = Paint()
      ..blendMode = BlendMode.srcATop // Apply the shadow over the widget
      ..colorFilter =
          ColorFilter.mode(color, BlendMode.srcOut) // Set shadow color
      ..imageFilter =
          ImageFilter.blur(sigmaX: blur, sigmaY: blur); // Apply blur

    // Draw the shadow layer on the canvas
    canvas.saveLayer(outerRect, shadowPaint);
    context.paintChild(child!, offset);

    // Restore the canvas stack
    canvas.restore();
    canvas.restore();
  }
}
```

Check out my full solution in my [gist](https://gist.github.com/cristinaponcela/3053d28b0ba280e2b61da5e34b8e5203).
