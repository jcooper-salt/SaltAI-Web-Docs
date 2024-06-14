---
tags:
- Image
- ImageBlend
- ImageComposite
---

# Image Blending Mode
## Documentation
- Class name: `Image Blending Mode`
- Category: `WAS Suite/Image`
- Output node: `False`

This node is designed to blend two images together using a variety of blending modes. It allows for the creation of composite images by applying different mathematical operations to the pixel values of the input images based on the selected mode.
## Input types
### Required
- **`image_a`**
    - The first input image to be blended. It serves as the base layer in the blending operation.
    - Comfy dtype: `IMAGE`
    - Python dtype: `torch.Tensor`
- **`image_b`**
    - The second input image to be blended. It is combined with the first image according to the specified blending mode.
    - Comfy dtype: `IMAGE`
    - Python dtype: `torch.Tensor`
- **`mode`**
    - Specifies the blending mode to be used for combining the two images. Each mode applies a different mathematical operation, affecting the visual outcome of the blend.
    - Comfy dtype: `COMBO[STRING]`
    - Python dtype: `str`
- **`blend_percentage`**
    - Determines the intensity of the blend between the two images. A higher percentage results in a stronger influence of the second image on the final output.
    - Comfy dtype: `FLOAT`
    - Python dtype: `float`
## Output types
- **`image`**
    - Comfy dtype: `IMAGE`
    - The result of blending the two input images according to the specified mode and blend percentage. It represents the composite image generated by the blending operation.
    - Python dtype: `torch.Tensor`
## Usage tips
- Infra type: `GPU`
- Common nodes:
    - [PreviewImage](../../Comfy/Nodes/PreviewImage.md)
    - [VAEEncode](../../Comfy/Nodes/VAEEncode.md)
    - [Image Blending Mode](../../was-node-suite-comfyui/Nodes/Image Blending Mode.md)



## Source code
```python
class WAS_Image_Blending_Mode:
    def __init__(self):
        pass

    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "image_a": ("IMAGE",),
                "image_b": ("IMAGE",),
                "mode": ([
                    "add",
                    "color",
                    "color_burn",
                    "color_dodge",
                    "darken",
                    "difference",
                    "exclusion",
                    "hard_light",
                    "hue",
                    "lighten",
                    "multiply",
                    "overlay",
                    "screen",
                    "soft_light"
                ],),
                "blend_percentage": ("FLOAT", {"default": 1.0, "min": 0.0, "max": 1.0, "step": 0.01}),
            },
        }

    RETURN_TYPES = ("IMAGE",)
    RETURN_NAMES = ("image",)
    FUNCTION = "image_blending_mode"

    CATEGORY = "WAS Suite/Image"

    def image_blending_mode(self, image_a, image_b, mode='add', blend_percentage=1.0):

        # Install Pilgram
        if 'pilgram' not in packages():
            install_package("pilgram")

        # Import Pilgram module
        import pilgram

        # Convert images to PIL
        img_a = tensor2pil(image_a)
        img_b = tensor2pil(image_b)

        # Apply blending
        if mode:
            if mode == "color":
                out_image = pilgram.css.blending.color(img_a, img_b)
            elif mode == "color_burn":
                out_image = pilgram.css.blending.color_burn(img_a, img_b)
            elif mode == "color_dodge":
                out_image = pilgram.css.blending.color_dodge(img_a, img_b)
            elif mode == "darken":
                out_image = pilgram.css.blending.darken(img_a, img_b)
            elif mode == "difference":
                out_image = pilgram.css.blending.difference(img_a, img_b)
            elif mode == "exclusion":
                out_image = pilgram.css.blending.exclusion(img_a, img_b)
            elif mode == "hard_light":
                out_image = pilgram.css.blending.hard_light(img_a, img_b)
            elif mode == "hue":
                out_image = pilgram.css.blending.hue(img_a, img_b)
            elif mode == "lighten":
                out_image = pilgram.css.blending.lighten(img_a, img_b)
            elif mode == "multiply":
                out_image = pilgram.css.blending.multiply(img_a, img_b)
            elif mode == "add":
                out_image = pilgram.css.blending.normal(img_a, img_b)
            elif mode == "overlay":
                out_image = pilgram.css.blending.overlay(img_a, img_b)
            elif mode == "screen":
                out_image = pilgram.css.blending.screen(img_a, img_b)
            elif mode == "soft_light":
                out_image = pilgram.css.blending.soft_light(img_a, img_b)
            else:
                out_image = img_a

        out_image = out_image.convert("RGB")

        # Blend image
        blend_mask = Image.new(mode="L", size=img_a.size,
                               color=(round(blend_percentage * 255)))
        blend_mask = ImageOps.invert(blend_mask)
        out_image = Image.composite(img_a, out_image, blend_mask)

        return (pil2tensor(out_image), )

```