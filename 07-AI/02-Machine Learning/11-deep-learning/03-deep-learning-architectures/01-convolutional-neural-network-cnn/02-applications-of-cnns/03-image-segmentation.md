---
tags: ['ai', 'roadmap']
---

## Summary

**Image Segmentation** is a fundamental computer vision task that involves partitioning a digital image into multiple segments (sets of pixels), effectively assigning a specific label to every pixel in the image. Unlike **Image Classification**, which assigns a single label to an entire image, or **Object Detection**, which identifies objects with bounding boxes, segmentation provides a pixel-perfect understanding of the scene. It is critical for applications requiring high precision, such as medical diagnostics, autonomous driving, and satellite image analysis.

## Detailed Explanation

Image segmentation can be broadly categorized into three types, each serving different levels of granularity and use cases:

### 1. **Semantic Segmentation**
Semantic segmentation classifies each pixel into a predefined category without distinguishing between individual object instances. 
- **Concept**: All pixels belonging to the "car" class are colored the same, regardless of how many cars are in the image.
- **Key Architecture**: **Fully Convolutional Networks (FCN)**. FCNs replace the final fully connected layers of traditional CNNs with convolutional layers and upsampling techniques (like transposed convolutions) to produce an output map with the same spatial dimensions as the input.

### 2. **Instance Segmentation**
Instance segmentation goes a step further by identifying and delineating each individual object instance within a class.
- **Concept**: If there are three cars in an image, instance segmentation will provide three distinct masks, one for each car.
- **Key Architecture**: **Mask R-CNN**. This architecture extends the Faster R-CNN object detector by adding a parallel branch for predicting pixel-level masks on each Region of Interest (RoI).

### 3. **Panoptic Segmentation**
Panoptic segmentation unifies semantic and instance segmentation.
- **Concept**: It assigns both a semantic label (e.g., "sky", "road", "car") and an instance ID to every pixel. 
- **Granularity**: It treats "stuff" (amorphous regions like grass or sky) semantically and "things" (countable objects like cars or pedestrians) as instances.

### **Common Architectures and Techniques**

#### **U-Net**
Originally developed for medical image segmentation, U-Net is a symmetric **encoder-decoder** architecture:
- **Encoder (Contracting Path)**: A typical CNN that captures context and high-level features through successive convolutions and pooling.
- **Decoder (Expanding Path)**: Uses transposed convolutions to upsample the feature maps back to the original image size, enabling precise localization.
- **Skip Connections**: These are the hallmark of U-Net. They concatenate high-resolution features from the encoder directly to the decoder, helping the network recover spatial details lost during downsampling.

#### **DeepLab**
A series of models from Google that utilize **Atrous Convolution** (dilated convolution) to capture multi-scale context without increasing the number of parameters or losing spatial resolution. It often uses **Atrous Spatial Pyramid Pooling (ASPP)** to robustly segment objects at multiple scales.

## Interview Questions

**Q: What is the main difference between Semantic and Instance Segmentation?**
**A:** Semantic segmentation focuses on identifying the class of each pixel (e.g., all "tree" pixels get one label), while Instance segmentation identifies each individual object instance (e.g., each "tree" gets a unique ID and its own mask).

**Q: Why are "Skip Connections" crucial in the U-Net architecture?**
**A:** During the downsampling process in the encoder, spatial information is often lost in favor of high-level semantic features. Skip connections transfer the fine-grained spatial details from the early encoder layers directly to the decoder, allowing for more accurate boundary reconstruction during upsampling.

**Q: How does a Fully Convolutional Network (FCN) differ from a standard CNN used for classification?**
**A:** A standard CNN ends with fully connected (dense) layers that flatten the spatial information into a single class probability vector. An FCN replaces these with 1x1 convolutions and upsampling layers, allowing the network to output a 2D spatial map (mask) where each "pixel" represents a class prediction.

**Q: What is the role of the "Mask Branch" in Mask R-CNN?**
**A:** While the base Faster R-CNN architecture identifies object classes and bounding boxes, the Mask Branch is a small FCN applied to each detected object. It predicts a binary mask in a pixel-to-pixel manner, defining exactly which pixels inside the bounding box belong to the object.
