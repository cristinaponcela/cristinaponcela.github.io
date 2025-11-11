---
title: "Open Source AI Models to Enhance Your Product"
date: 2025-10-02 08:42:00 -500
categories: [StreamYard, AI, QoX]
tags: [production, open source, cost, my journey]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/NPS/nps.png"
---

StreamYard used to have a feature to create thumbnails using AI. This is available for scheduled posts, and appears through a button in the modal to create a broadcast. However, the past implementation used the API-based client ClipDrop, which cost us around $1k per month. 

![Desktop View](/assets/img/StreamYard/AIThumbnails/ai-thumbnails-button.png){: .normal}

The feature allowed users to automatically remove backgrounds from images to create professional-looking thumbnails. It was quite well liked, but was by far not profitable or justified its cost based on usage. Thus, we recently reimplemented this feature, but I moved from an expensive API-based solution to a cost-effective browser-based approach.

## Overview

One of my colleagues came up with an initial solution for a new implementation - to call the Hugging Face API, which would download the modal for each user and process the image. However, this was suboptimal both in terms of waiting time and structure - it added latency due to network requests and scaled costs with usage. So my job was to find a way to do this free of cost, and a compromise between model quality and processing time client-side.

What I found to be a good solution was using [onnxruntime](https://github.com/microsoft/onnxruntime/tree/main) ((Open Neural Network Exchange) Runtime), a package that allows you to run open-source web worker, machine learning models directly and entirely in the user's browser, so that, if you serve your model of choice from your CDN, no API calls are needed. By looking at the [onnx Community](https://huggingface.co/onnx-community), you can get an idea of the vast range of models they have, of different sizes, pixel inputs and outputs, and thus quality. I tested about half a dozen models, and found that anything above 100MB of weight took too long to load for the user (about 30+ seconds), but anything below doesn't have the best quality. Also note that a lot of the models are trained with human data only, so the background removal for images containing objects, landscapes, etc only produced pretty bad results.

Either way, this solution is completely free.



### Hugging Face Integration

Hugging Face serves as our model source, offering:
- Pre-trained models with proven performance
- Active community maintenance
- Extensive documentation and examples
- Regular updates and improvements

The model pipeline:
1. Select model from Hugging Face Hub
2. Convert to ONNX format
3. Optimize for web deployment
4. Host on our CDN
5. Load and run in browser

### Open Source Model Benefits

Using open source models provides several advantages:
- Community-tested implementations
- Transparent architecture
- Regular improvements
- No vendor lock-in
- Cost-effective scaling

## Licensing and Legal Considerations

One interesting and unexpected skill I learned during this task was handling licenses and compliance. A lot of the models offered by Hugging Face, especially the RMBD ones, do not actually allow distribution. Thus, I had limited options to test.

Onnxruntime, as most npm packages (and open-source frameworks), offers an `MIT` license. This offers commercial use, modification, distribution and private use. So for most intents and purposes, you are crystal clear using this. However, it does have a slightly obscure requirement: that you include the license and copyright notice in your codebase. While not client-facing, this step is important for full compliance and something I was unaware of. But creating a `licenses.json` in your root directory and naming the packages and the license does the trick.

Less popular, I found the models to offer an `Apache 2.0` license. While slightly stricter, it is still pretty versatile for a monetized product. Again, it offers commercial use, modification, distribution and even patent use. And as `MIT`, it requires a license and copyright notice inclusion, but also to document any state changes and to preserve notices.

So anything I found with either license was golden and good to go, in order to use the technology commercially, modify it for our needs and distribute it to our users while maintaining legal compliance and scaling without licensing concerns.


### Loading the Model and Image Processing

Essentially, what we do is create a pipeline for a messaging system that updates with the percentage progress of the model loading. On the completion message, it triggers the image processing. The only interesting part of this logic is where we create an alpha mask for alpha blending - depending on the confidence level of the model output, we decide to blur or delete a part of the background to a greater or lesser degree.  

We do this in stages:

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

And then the smart edge detection:

We implemented sophisticated edge detection:

```typescript
// Apply confidence-based alpha blending for smoother edges
for (let i = 0; i < imageData.width * imageData.height; i += 1) {
    const pixelIndex = i * 4;
    const maskValue = maskData[pixelIndex + 3];

    if (maskValue > 240) {
        // Very high confidence: keep fully opaque
        resultData[pixelIndex + 3] = 255;
    } else if (maskValue > 200) {
        // High confidence: slight enhancement
        resultData[pixelIndex + 3] = Math.round(maskValue * 1.06);
    } else if (maskValue > 100) {
        // Medium confidence: smooth transitions
        resultData[pixelIndex + 3] = Math.round(maskValue * 1.2);
    } else if (maskValue > 50) {
        // Low confidence: fade out artifacts
        resultData[pixelIndex + 3] = Math.round(maskValue * 0.8);
    } else {
        // Very low confidence: fully transparent
        resultData[pixelIndex + 3] = 0;
    }
}
```

And again, the system provides the progress updates for the states of loading, processing and uploading.

## Benefits

1. **Cost Efficiency**:
   - Eliminated $1,000/month API costs
   - No per-request charges
   - Scales with user base at no additional cost

2. **Performance**:
   - No network latency for API calls
   - Immediate processing start
   - Progress tracking for better UX

3. **Reliability**:
   - Works offline
   - No API dependency
   - Consistent performance

## Conclusion

By moving from ClipDrop's API to a browser-based solution, we've created a more sustainable, cost-effective, and user-friendly AI thumbnail feature. This demonstrates how moving AI processing to the client can dramatically reduce costs while maintaining or improving functionality.
