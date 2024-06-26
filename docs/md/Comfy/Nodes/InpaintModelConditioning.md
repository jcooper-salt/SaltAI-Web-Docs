---
tags:
- Conditioning
---

# InpaintModelConditioning
## Documentation
- Class name: `InpaintModelConditioning`
- Category: `conditioning/inpaint`
- Output node: `False`

This node is designed for conditioning in the context of inpainting tasks, where it likely processes and prepares inputs such as images and masks for inpainting, integrating with VAE models for generating or modifying image content. The node's specific functionality, including how it handles conditioning and interacts with other components like VAEs, remains undefined due to the lack of detailed information in the provided context.
## Input types
### Required
- **`positive`**
    - Represents the positive conditioning information required for the inpainting process, influencing the generation or modification of image content.
    - Comfy dtype: `CONDITIONING`
    - Python dtype: `torch.Tensor`
- **`negative`**
    - Represents the negative conditioning information, which likely serves to guide the inpainting process by specifying undesired attributes or areas in the generated or modified image content.
    - Comfy dtype: `CONDITIONING`
    - Python dtype: `torch.Tensor`
- **`vae`**
    - Specifies the VAE model used in the inpainting process, crucial for encoding or decoding image data.
    - Comfy dtype: `VAE`
    - Python dtype: `object`
- **`pixels`**
    - The input image pixels that are subject to inpainting, serving as the primary content for the node's operation.
    - Comfy dtype: `IMAGE`
    - Python dtype: `torch.Tensor`
- **`mask`**
    - The mask indicating areas to inpaint, guiding the node in identifying which parts of the image require modification or generation.
    - Comfy dtype: `MASK`
    - Python dtype: `torch.Tensor`
## Output types
- **`positive`**
    - Comfy dtype: `CONDITIONING`
    - The processed positive conditioning output, ready for further steps in the inpainting pipeline.
    - Python dtype: `torch.Tensor`
- **`negative`**
    - Comfy dtype: `CONDITIONING`
    - The processed negative conditioning output, indicating attributes or areas to avoid in the final image content.
    - Python dtype: `torch.Tensor`
- **`latent`**
    - Comfy dtype: `LATENT`
    - The latent representation of the image, potentially modified or generated by the VAE model as part of the inpainting process.
    - Python dtype: `torch.Tensor`
## Usage tips
- Infra type: `GPU`
- Common nodes: unknown


## Source code
```python
class InpaintModelConditioning:
    @classmethod
    def INPUT_TYPES(s):
        return {"required": {"positive": ("CONDITIONING", ),
                             "negative": ("CONDITIONING", ),
                             "vae": ("VAE", ),
                             "pixels": ("IMAGE", ),
                             "mask": ("MASK", ),
                             }}

    RETURN_TYPES = ("CONDITIONING","CONDITIONING","LATENT")
    RETURN_NAMES = ("positive", "negative", "latent")
    FUNCTION = "encode"

    CATEGORY = "conditioning/inpaint"

    def encode(self, positive, negative, pixels, vae, mask):
        x = (pixels.shape[1] // 8) * 8
        y = (pixels.shape[2] // 8) * 8
        mask = torch.nn.functional.interpolate(mask.reshape((-1, 1, mask.shape[-2], mask.shape[-1])), size=(pixels.shape[1], pixels.shape[2]), mode="bilinear")

        orig_pixels = pixels
        pixels = orig_pixels.clone()
        if pixels.shape[1] != x or pixels.shape[2] != y:
            x_offset = (pixels.shape[1] % 8) // 2
            y_offset = (pixels.shape[2] % 8) // 2
            pixels = pixels[:,x_offset:x + x_offset, y_offset:y + y_offset,:]
            mask = mask[:,:,x_offset:x + x_offset, y_offset:y + y_offset]

        m = (1.0 - mask.round()).squeeze(1)
        for i in range(3):
            pixels[:,:,:,i] -= 0.5
            pixels[:,:,:,i] *= m
            pixels[:,:,:,i] += 0.5
        concat_latent = vae.encode(pixels)
        orig_latent = vae.encode(orig_pixels)

        out_latent = {}

        out_latent["samples"] = orig_latent
        out_latent["noise_mask"] = mask

        out = []
        for conditioning in [positive, negative]:
            c = node_helpers.conditioning_set_values(conditioning, {"concat_latent_image": concat_latent,
                                                                    "concat_mask": mask})
            out.append(c)
        return (out[0], out[1], out_latent)

```
