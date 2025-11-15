---
title: "Open Source AI Models to Enhance Your Product"
date: 2025-10-02 08:42:00 -500
categories: [StreamYard, AI, QoX]
tags: [production, open source, cost, my journey]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/"
---

StreamYard used to have a feature to create thumbnails using AI. This is available for scheduled posts, and appears through a button in the modal to create a broadcast. However, the past implementation used the API-based client ClipDrop, which cost us around $1k per month. 

![Desktop View](/assets/img/StreamYard/AIThumbnails/ai-thumbnails-button.png){: .normal}

The feature allowed users to automatically remove backgrounds from images to create professional-looking thumbnails. It was quite well liked, but was by far not profitable or justified its cost based on usage. Thus, we recently reimplemented this feature, but I moved from an expensive API-based solution to a cost-effective browser-based approach.

## Overview

One of my colleagues came up with an initial solution for a new implementation - to call the Hugging Face API, which would download the modal for each user and process the image. However, this was suboptimal both in terms of waiting time and structure - it added latency due to network requests and scaled costs with usage. So my job was to find a way to do this free of cost, and a compromise between model quality and processing time client-side.

What I found to be a good solution was using [onnxruntime](https://github.com/microsoft/onnxruntime/tree/main) ((Open Neural Network Exchange) Runtime), a package that allows you to run open-source web worker, machine learning models directly and entirely in the user's browser, so that, if you serve your model of choice from your CDN, no API calls are needed. By looking at the [onnx Community](https://huggingface.co/onnx-community), you can get an idea of the vast range of models they have, of different sizes, pixel inputs and outputs, and thus quality. I tested about half a dozen models, and found that anything above 100MB of weight took too long to load for the user (about 30+ seconds), but anything below doesn't have the best quality. Also note that a lot of the models are trained with human data only, so the background removal for images containing objects, landscapes, etc only produced pretty bad results.

Either way, this solution is completely free.

See how fast it is too!

<iframe class="embed-video" loading="lazy" src="/assets/img/StreamYard/AIThumbnails/ai-thumbnails-demo.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

As you can see, I also took the chance to add the SY logo and make it a paid option to hide it. We initially thought this may drive conversions as the feature is super fun, but spoiler alert, in the first 10 days it only resulted in 3 conversions :(

<iframe class="embed-video" loading="lazy" src="/assets/img/StreamYard/AIThumbnails/sy-logo.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

This is cool: to generate the final thumbnail, we use the HTML Canvas API to composite all elements. The canvas is set to a specific resolution, and we draw layers in order: background image first, then any uploaded images, followed by the title text with custom styling, the date/time badge with a calendar icon, and finally the logo in the top-right corner. Each element is positioned based on the selected layout style (horizontal, left-aligned, slanted, etc.), with proper margins and padding. So it quite literally is like painting the components on a canvas!

```typescript
// Create XxY canvas
const canvas = document.createElement('canvas');
canvas.width = X;
canvas.height = Y;
const ctx = canvas.getContext('2d');

// Layer 1: Background
ctx.fillStyle = '#000000';
ctx.fillRect(0, 0, 1920, 1080);
await drawBackground(ctx, backgroundImage);

// Layer 2: Title text
drawTitle(ctx, 'My Broadcast Title', {
  fontSize: 120,
  color: '#FFFFFF',
  position: 'left'
});

// Layer 3: Date/Time badge
drawDateTime(ctx, 'Dec 15, 2024 • 3:00 PM', {
  fontSize: 48,
  position: 'bottom-right',
  icon: calendarIcon
});

// Layer 4: Logo
const logo = await loadImage(logoUrl);
ctx.drawImage(logo, 1920 - 280 - 30, 30, 280, logoHeight);

// Export as image
canvas.toBlob(blob => downloadThumbnail(blob), 'image/png');
```

### Hugging Face Integration

The AI thumbnail feature uses a pre-trained background removal model from Hugging Face, an open-source platform for sharing pre-trained AI models. They have proven performance, active community maintenance and good docu, making the perfect candidate as a free solution to our above issue.

The models are in ONNX format, the universal format for AI models that can run anywhere, including browsers, and hosted on our CDN. When a user uploads an image, their browser downloads and caches the model, then processes everything locally using ONNX Runtime Web. This is the JavaScript package that runs ONNX models in the browser using WebAssembly. 

The processing runs in a Web Worker — a separate browser thread for heavy computation — so the UI stays responsive while the model works. The worker resizes the image to 512×512, runs inference, and sends progress updates back to the main thread. Since we have a clear pipeline for progress messages, I added logic in the UI so that users can see their percentage progress while the image is being processed, to encourage them to wait for this to finish, though it is typically done in under 10 seconds.

```typescript
// Main Thread: Send work to Web Worker
worker.postMessage({ type: 'init_model' });
worker.postMessage({ type: 'process_image', imageData });

// Main Thread: Receive progress updates
worker.onmessage = (event) => {
  if (event.data.type === 'model_loading') showProgress(event.data.progress);
  if (event.data.type === 'processing') showProgress(event.data.progress);
  if (event.data.type === 'success') displayResult(event.data.processedImage);
};

// Web Worker: Process on separate thread
self.onmessage = async (event) => {
  if (event.data.type === 'init_model') {
    model = await loadFromCDN(); // Browser's native HTTP caching caches this automatically
    postMessage({ type: 'model_loading', progress: 100 });
  }
  
  if (event.data.type === 'process_image') {
    const prepared = preprocessImage(event.data.imageData);
    const alphaMask = await model.run(prepared); // AI inference
    const result = applyMask(event.data.imageData, alphaMask);
    postMessage({ type: 'success', processedImage: result });
  }
};
```

And this provides the progress updates for the states of loading, processing and uploading.

The model then outputs an alpha mask: a grayscale image where white pixels are foreground and black pixels are background. By applying this mask to make the background transparent, we can leverage confidence-based blending to smooth edges. Think of it as edge detection by fading out pixels that we are confident contrast with our image:

```typescript
// Apply confidence-based alpha blending for smoother edges
for (let i = 0; i < imageData.width * imageData.height; i += 1) {
    const pixelIndex = i * 4;
    const maskValue = maskData[pixelIndex + 3];

    if (maskValue > 240) {
        // Very high confidence: keep fully opaque
        resultData[pixelIndex + 3] = 255;
    } else if (maskValue > 200) {
        // High confidence: slight enhancement for crisp edges
        resultData[pixelIndex + 3] = Math.round(maskValue * 1.06);
    } else if (maskValue > 100) {
        // Medium confidence: boost for smooth transitions
        resultData[pixelIndex + 3] = Math.round(maskValue * 1.2);
    } else if (maskValue > 50) {
        // Low confidence: reduce to fade out edge artifacts
        resultData[pixelIndex + 3] = Math.round(maskValue * 0.8);
    } else {
        // Very low confidence: fully transparent
        resultData[pixelIndex + 3] = 0;
    }
}
```

So the steps are:

```typescript
const processImage = async (imageData: Promise => {
    // Pre-processing
    const { inputData, inputShape } = preprocessImage(imageData);
    
    // Model inference
    const outputData = await runInference(inputData, inputShape);
    
    // Post-processing with smart alpha blending
    const alphaMask = postprocessOutput(
        outputData,
        imageData.width,
        imageData.height
    );
}
```


## Licensing and Legal Considerations

One interesting and unexpected skill I learned during this task was handling licenses and compliance. A lot of the models offered by Hugging Face, especially the RMBD ones, do not actually allow distribution. Thus, I had limited options to test.

Onnxruntime, as most npm packages (and open-source frameworks), offers an `MIT` license. This offers commercial use, modification, distribution and private use. So for most intents and purposes, you are crystal clear using this. However, it does have a slightly obscure requirement: that you include the license and copyright notice in your codebase. While not client-facing, this step is important for full compliance and something I was unaware of. But creating a `licenses.json` in your root directory and naming the packages and the license does the trick.

Less popular, I found the models to offer an `Apache 2.0` license. While slightly stricter, it is still pretty versatile for a monetized product. Again, it offers commercial use, modification, distribution and even patent use. And as `MIT`, it requires a license and copyright notice inclusion, but also to document any state changes and to preserve notices.

So anything I found with either license was golden and good to go, in order to use the technology commercially, modify it for our needs and distribute it to our users while maintaining legal compliance and scaling without licensing concerns.


## Conclusion

By moving from ClipDrop's API to a browser-based solution, I created a more sustainable, cost-effective, and user-friendly AI thumbnail feature. WHile it wasn't massively impactful, it was certainly well received - within the first hour, 1k users had interacted with the feature, and within the first 5 days the number rose to 36K. As a dev, this is super fun to see and a privilege to work at a firm that lets us ship features fast and have agency to see tasks through from start to finish.

This also demonstrates the power of open source packages and services - I really believe that nowadays, if you look hard enough, you can find a free and open source solution for 90% of whatever features you want to build.
