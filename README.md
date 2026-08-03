# Face Liveness Detection (Anti-Spoofing)

I build this project to search and explore the filed of face anti-spoofing with various types of the anti-spoofing techniques [print, replay, 2D mask, 3D mask attacks], I saw that most of the projects about this field do tend to skip one or two of the different types of the anti-spoofing techniques, which derived me to test what could be a good baseline that cover all the most famous spoofing techniques 

## Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Dataset](#dataset)
- [Training Pipeline](#training-pipeline)
- [Results](#results)
- [Challenges & Debugging Journey](#challenges--debugging-journey)
- [Future Work](#future-work)
- [Acknowledgements](#acknowledgements)

## Overview

- Problem statement: Face recognition systems are increasingly used to authenticate users in banking, mobile devices, and access control systems. However, these systems are vulnerable to presentation attacks — attempts to fool the camera using a printed photo, a replayed video on a screen, or a 3D/2D mask of the legitimate user's face. A face recognition system that cannot tell a live person from a photo of that person is not actually secure, no matter how accurate its identity-matching is.

This project addresses that gap by building a face liveness detection (anti-spoofing) model that classifies a detected face as real or spoofed, and further distinguishes between spoof types (2D mask, 3D mask, printed photo, screen replay). The goal is to serve as a lightweight, fast pre-check that runs before identity verification, rejecting spoofed inputs before they ever reach the recognition stage
- Classes detected: 2D Mask, 3D Mask, Replay, Print, Real
- High-level approach: face detector using YuNet to crop the background into a single-frame CNN classifier 

## Architecture

- Backbone used MobileNetV3-Large
- I aimed to single frame CNN-Architecture to it's simplicity and ability to be deployed in edge machines and still provide decent accuracy compared to motion based classifiers, and choosed MobileNetV3-Large for its efficiency and lightweight based on the Depthwise separable convolutions and SE blocks, which fitted my objectves well 
- the pipeline: face detection → crop → preprocess → classifier 

## Dataset

- Source(s): I merged my dataset from multiple datasets mentioned in the [Acknowledgements](#acknowledgements) section. I merged different datasets to cover the different types of spoofing and used step frame of 3 and YuNet for face detection filtering and then performed under sampling on the training dataset
- Class distribution(before under sampling):
  | Class   |  count |
  |---------|--------|
  | 2DMask  |  5072  |
  | 3DMask  |  5939  |
  | Print   |  4979  |
  | Real    |  3154  |
  | Replay  |  4847  |
- Preprocessing pipeline:
  - Face detection using YuNet
  - i used 0.25 margin before cropping
  - standardization
- Dataset: https://drive.google.com/file/d/1tScS40vBZUYuGva5kyY1KMmwZvYavCCC/view?usp=sharing

## Training Pipeline

- Framework: PyTorch
- Key hyperparameters (final config): optimzier = SGD, lr = 7e-4, LR scheduler (step size = 7/gamma = 0.3), batch size = 256, epochs = 25, dropout = .75
- Loss function: cross-entropy 
- Augmentation strategy: (Resize, Random Rotation of 16 degrees, Random Horizontal Flip, Random Color Jitter of (brightness=0.25, contrast=0.20, saturation=0.15, hue=0.06)),
- Mid training: early stopping / best-checkpoint saving
- Hardware: GPU used (e.g. T4 via Kaggle)

## Results

- Final validation accuracy of 82% Compared against the face-antispoof-onnx pipline which scored 77% on the same validation dataset

## Challenges & Debugging Journey

-**Augmentation Limit**: any Augmentation further than the Augmentation mentioned did caused an underfitting and didn't really help the overfitting problem past the brightness of 0.2, contrast of 0.15, saturation of 0.15, and hue of 0.6
- **LR scheduler tuning**: initial 7 config caused a sharp plateau (too-early, too-aggressive decay); retuning step_size/gamma unlocked a big accuracy jump
- **Class imbalance**: tried Under sampling 
- **Overfitting past a val peak**: kept hitting really big Overfitting so i used in first a dropout of 0.2 and weight-decay of 1e-1 and kept increaseing them till a dropout of 0.75 and weight decay of 1e-2 which does decrease the overfitting noticeably but the divergence  was so slow, and actually the model couldn't push the validation accuracy past 77%, so i used only the dropout of 0.75 and no weight decay and worked well and caused the jump to 82%
- **ONNX export/conversion issues**: quantized model ops unsupported by conversion tooling, resolved by keeping quantized models in ONNX Runtime rather than converting back to PyTorch


## Future Work

- using auxiliary supervision such as FFT head, or/and depth head during training get the benefits of the information in the frequency domain and the depth information

## Acknowledgements

- face-antispoof-onnx :https://github.com/facenox/face-antispoof-onnx
- Datasets used:
  - Anti-Spoofing Dataset, 30,000 sets: https://www.kaggle.com/datasets/tapakah68/anti-spoofing
  - Liveness Detection Replay Attack Dataset - 5,000+: https://www.kaggle.com/datasets/axondata/liveness-detection-real-and-display-attacks-5k
  - Asian People - Liveness Detection Video Dataset: https://www.kaggle.com/datasets/trainingdatapro/asian-people-liveness-detection-video-dataset
  - iBeta 1 - 35,800 Liveness Detection Dataset: https://www.kaggle.com/datasets/trainingdatapro/ibeta-level-1-liveness-detection-dataset-part-1
  - Printed Photos Attacks: https://www.kaggle.com/datasets/trainingdatapro/printout
  - Web Camera Face Liveness Detection - Face Dataset: https://www.kaggle.com/datasets/trainingdatapro/web-camera-face-liveness-detection
- Team members:
  - Mohammed Abdulmajeed Algunaid 
