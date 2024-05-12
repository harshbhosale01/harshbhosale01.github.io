---
layout: page
title: Image processing on FPGA
description: Implemented JPEG image compression algorithms on FPGA.
img: assets/img/image_processing_on_fpga.png
importance: 1
category: work
related_publications: false
---

In this project we take YCbCr color space image and apply JPEG compression algorithm to it. We apply algorithms in two stages, the first stage is the encoding stage. In this stage algorithms like Discrete cosine transform and quantization are applied and then we save it in memory in an encoded format. In the second stage we take the encoded string from memory and apply the decoding algorithms like dequantization and inverse DCT to it. This achieves 70 percent compression with little perceptible loss in image quality.

For more details regarding the project, view this [document](https://docs.google.com/document/d/10EYbFGkmdOjBmTeEX9F7WU6JgaQeKMwrnvfq3rCaGn0/edit?usp=sharing).
For code and other details visit [github](https://github.com/harshbhosale01/image-processing-fpga).

Following is the input uncomprssed image and output compressed image:
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Original-img.jpeg" title="Input Image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/70-compressed-img.png" title="Output Image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

    