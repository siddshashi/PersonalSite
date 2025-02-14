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

Using an image diffusion model, I implemented an iterative denoising process and classifier-free guidance to generate prompt-based images. Sampling from the model, I performed image-to-image translation, editing and hallucination on top of hand-drawn images and inpainting in erased sections of images. I then implemented [visual anagrams][visanagrams], optical illusions where the generated images corresponded to different prompts depending on the viewing angle. Finally, I similarly implemented [factorized diffusion][facdiffusion], illusions where the viewing distance corresponds to different prompts. [GitHub][ghlink]

[ghlink]: https://github.com/siddshashi/HybridDiffusion
[visanagrams]: https://dangeng.github.io/visual_anagrams/
[facdiffusion]: https://arxiv.org/abs/2404.11615

## Setup

For my project, I explored the DeepFloyd IF diffusion model, a state-of-the-art text-to-image model trained by Stability AI. The project involved working with the two-stage DeepFloyd pipeline, where the first stage generated low-resolution images, and the second stage upsampled them to higher resolutions. Throughout the process, I generated images using provided and custom text prompts, evaluating how different inference step values affected image quality.

## Implementing Denoising

A major part of the project involved implementing and modifying the diffusion process. I first wrote a forward process function to simulate the addition of noise to an image over time, demonstrating how a clean image degrades into pure noise. Then, I applied classical Gaussian filtering techniques to denoise images, comparing their effectiveness to DeepFloyd’s pretrained denoising UNet. I also implemented iterative denoising using a strided timestep approach, which improved image reconstruction quality while reducing computational cost. Finally, I experimented with diffusion-based image generation, creating high-quality images from pure noise using the model’s iterative denoising capabilities. 

## Classifier-free Guidance (CFG)

In the previous section, some of the generated images lacked coherence or produced unrealistic results. To significantly improve image quality while sacrificing some diversity, I employed a technique known as Classifier-Free Guidance (CFG). This approach helps steer the image generation process toward more faithful and detailed results. To implement this, I modified my iterative denoising function to incorporate CFG by running the diffusion model twice—once with the text prompt and once with an empty prompt. 

## Sampling

Following the [SDEdit Algorithm][sdedit], I took an original test image, noised it a little, and forced it back onto the image manifold without any conditioning. For the prompt to follow, I provided "a high quality photo." Effectively, this gave me an image that is similar to the test image (with a low-enough noise level). I then tried using different guidance prompts like "a rocket ship", and saw the guidance level vary depending on the noise. 

I also experimented with starting with hand-drawn images, projecting some quick sketches onto the natural image manifold. With more noise, there was evidently more hallucination, and I found that with too much the generated image would look nothing like what I intended. Finally, I implemented the results of the [RePaint Paper][repaint]. That is, given an image `x`, and a binary mask `m`, I creatd a new image that has the same content where `x` is 0, but new content wherever `m` is 1.

[sdedit]: https://sde-image-editing.github.io/
[repaint]: https://arxiv.org/abs/2201.09865 

## Visual Anagrams

For creating a visual anagram, I found that results turned out well when I tried to create an image that looks like "an oil painting of people around a campfire", but when flipped upside down reveals "an oil painting of an old man".

To do this, I denoised an image `xt` at step `t` normally with the prompt "an oil painting of an old man", to obtain noise estimate `e1`. But at the same time, I flipped `xt` upside down, and denoised with the prompt "an oil painting of people around a campfire", to get noise estimate `e2`. I then flipped `xt` back, to make it right-side up, and averaged the two noise estimates. Finally, I then performed a reverse/denoising diffusion step with the averaged noise estimate.

## Hybrid Images

For creating hybrid images, I took a similar approach to visual anagrams by combining two types of noise into one estimate. To obtain `e1`, I used a low pass filter on the image, and to obtain `e2` I used a high pass. Because these noises correspond to different frequencies, I just added them together to get the total estimate. The prompts "a lithograph of a skull" and "a lithograph of waterfalls" turned out really cool! 