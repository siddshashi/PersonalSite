+++
authors = ["Lone Coder"]
title = "Messing with Image Diffusion"
date = "2024-10-05"
description = "Sample article showcasing basic Markdown syntax and formatting for HTML elements."
tags = [
    "hugo",
    "markdown",
    "css",
    "html",
]
categories = [
    "theme demo",
    "syntax",
]
series = ["Theme Demo"]
aliases = ["migrate-from-jekyl"]
+++

Using an image diffusion model, I implemented an iterative denoising process and classifier-free guidance to generate prompt-based images. Sampling from the model, I performed image-to-image translation, editing and hallucination on top of hand-drawn images and inpainting in erased sections of images. I then implemented [visual anagrams][visanagrams], optical illusions where the generated images corresponded to different prompts depending on the viewing angle. Finally, I similarly implemented [factorized diffusion][facdiffusion], illusions where the viewing distance corresponds to different prompts. 

[visanagrams]: https://dangeng.github.io/visual_anagrams/
[facdiffusion]: https://arxiv.org/abs/2404.11615

## Setup

For my project, I explored the DeepFloyd IF diffusion model, a state-of-the-art text-to-image model trained by Stability AI. The project involved working with the two-stage DeepFloyd pipeline, where the first stage generated low-resolution images, and the second stage upsampled them to higher resolutions. Throughout the process, I generated images using provided and custom text prompts, evaluating how different inference step values affected image quality.

## Implementing Denoising

A major part of the project involved implementing and modifying the diffusion process. I first wrote a forward process function to simulate the addition of noise to an image over time, demonstrating how a clean image degrades into pure noise. Then, I applied classical Gaussian filtering techniques to denoise images, comparing their effectiveness to DeepFloyd’s pretrained denoising UNet. I also implemented iterative denoising using a strided timestep approach, which improved image reconstruction quality while reducing computational cost. Finally, I experimented with diffusion-based image generation, creating high-quality images from pure noise using the model’s iterative denoising capabilities. 

## Classifier-free Guidance (CFG)

## Sampling

## Visual Anagrams

## Hybrid Images