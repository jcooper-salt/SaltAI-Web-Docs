---
tags:
- Searge
---

# Searge's Magic Box for SDXL
## Documentation
- Class name: `SeargeMagicBox`
- Category: `Searge/Magic`
- Output node: `False`

SeargeMagicBox orchestrates a complex pipeline for image generation and manipulation, leveraging a series of specialized processing stages. It dynamically selects and executes various stages such as data preprocessing, checkpoint loading, image styling, and resolution enhancement, facilitating a customizable and modular approach to generating high-quality images.
## Input types
### Required
- **`stage`**
    - Specifies the current processing stage to be executed, determining the specific operation or transformation to be applied to the image data.
    - Comfy dtype: `COMBO[STRING]`
    - Python dtype: `str`
- **`input_from`**
    - Indicates the source of input for the current stage, potentially allowing for custom input alongside standard pipeline data.
    - Comfy dtype: `COMBO[STRING]`
    - Python dtype: `str`
- **`output_to`**
    - Specifies the destination for the stage's output, enabling custom handling or integration into the standard data stream.
    - Comfy dtype: `COMBO[STRING]`
    - Python dtype: `str`
### Optional
- **`data`**
    - The data object that carries the image and related information through the processing pipeline, serving as the input and output medium for each stage.
    - Comfy dtype: `SRG_DATA_STREAM`
    - Python dtype: `dict`
- **`custom_input`**
    - Provides a mechanism for supplying custom input data to a stage, enabling specialized processing beyond the standard pipeline content.
    - Comfy dtype: `SRG_STAGE_INPUT`
    - Python dtype: `dict`
## Output types
- **`data`**
    - Comfy dtype: `SRG_DATA_STREAM`
    - The updated data object, reflecting the modifications and additions made by the current processing stage.
    - Python dtype: `dict`
- **`custom_output`**
    - Comfy dtype: `SRG_STAGE_OUTPUT`
    - Custom output generated by the current stage, available for use in subsequent stages or for external processing.
    - Python dtype: `dict`
## Usage tips
- Infra type: `CPU`
- Common nodes: unknown


## Source code
```python
class SeargeMagicBox:
    # processing stages supported by the magic box
    NONE = "none - skip"
    PRE_PROCESS_DATA = "pre-process data"
    LOAD_CHECKPOINTS = "load checkpoints"
    APPLY_LORAS = "apply loras"
    PROMPT_STYLING = "prompt styling"
    CLIP_CONDITIONING = "clip conditioning"
    CLIP_MIXING = "clip mixing"
    APPLY_CONTROLNET = "apply controlnet"
    LATENT_INPUTS = "latent inputs"
    SAMPLING = "sampling"
    LATENT_DETAILER = "latent detailer"
    VAE_DECODE_SAMPLED = "vae decode sampled"
    HIGH_RESOLUTION = "high resolution"
    HIRES_DETAILER = "hires detailer"
    VAE_DECODE_HI_RES = "vae decode hi-res"
    UPSCALING = "upscaling"
    IMAGE_SAVING = "image saving"

    STAGES = [
        NONE,
        PRE_PROCESS_DATA,
        LOAD_CHECKPOINTS,
        APPLY_LORAS,
        PROMPT_STYLING,
        CLIP_CONDITIONING,
        CLIP_MIXING,
        APPLY_CONTROLNET,
        LATENT_INPUTS,
        SAMPLING,
        LATENT_DETAILER,
        VAE_DECODE_SAMPLED,
        HIGH_RESOLUTION,
        HIRES_DETAILER,
        VAE_DECODE_HI_RES,
        UPSCALING,
        IMAGE_SAVING,
    ]

    # option to take inputs from a custom stage instead of the data stream
    DATA = "data stream"
    CUSTOM_AND_DATA = "custom stage & data stream"
    INPUT_OUTPUT = [
        DATA,
        CUSTOM_AND_DATA,
    ]

    def __init__(self):
        self.stage_pre_process_data = None
        self.stage_load_checkpoints = None
        self.stage_apply_loras = None
        self.stage_prompt_styling = None
        self.stage_clip_conditioning = None
        self.stage_clip_mixing = None
        self.stage_latent_inputs = None
        self.stage_apply_controlnet = None
        self.stage_sampling = None
        self.stage_vae_decode_sampled = None
        self.stage_latent_detailer = None
        self.stage_high_resolution = None
        self.stage_hires_detailer = None
        self.stage_vae_decode_hi_res = None
        self.stage_upscaling = None
        self.stage_image_saving = None
        self.stage_ = None

    @classmethod
    def INPUT_TYPES(s):
        return {
            "required": {
                "stage": (s.STAGES, {"default": s.NONE},),
                "input_from": (s.INPUT_OUTPUT,),
                "output_to": (s.INPUT_OUTPUT,),
            },
            "optional": {
                "data": ("SRG_DATA_STREAM",),
                "custom_input": ("SRG_STAGE_INPUT",),
            },
        }

    RETURN_TYPES = ("SRG_DATA_STREAM", "SRG_STAGE_OUTPUT",)
    RETURN_NAMES = ("data", "custom_output",)
    FUNCTION = "process"

    CATEGORY = UI.CATEGORY_MAGIC

    def run_stage(self, stage, data, stage_input=None):
        stage_processor = None

        has_data = data is not None

        # clear old stage output from data stream
        stage_output = stage_input
        if has_data:
            data["stage_output"] = None

        # stage: "none - skip" - does nothing, skip processing in this magic box
        if stage == self.NONE:
            pass

        elif stage == self.PRE_PROCESS_DATA:
            if self.stage_pre_process_data is None:
                self.stage_pre_process_data = SeargePreProcessData()
            stage_processor = self.stage_pre_process_data

        elif stage == self.LOAD_CHECKPOINTS:
            if self.stage_load_checkpoints is None:
                self.stage_load_checkpoints = SeargeStageLoadCheckpoints()
            stage_processor = self.stage_load_checkpoints

        elif stage == self.APPLY_LORAS:
            if self.stage_apply_loras is None:
                self.stage_apply_loras = SeargeStageApplyLoras()
            stage_processor = self.stage_apply_loras

        elif stage == self.PROMPT_STYLING:
            if self.stage_prompt_styling is None:
                # print("TODO: implement stage " + stage)
                self.stage_prompt_styling = SeargeStage()
            stage_processor = self.stage_prompt_styling

        elif stage == self.CLIP_CONDITIONING:
            if self.stage_clip_conditioning is None:
                self.stage_clip_conditioning = SeargeStageClipConditioning()
            stage_processor = self.stage_clip_conditioning

        elif stage == self.CLIP_MIXING:
            if self.stage_clip_mixing is None:
                # print("TODO: implement stage " + stage)
                self.stage_clip_mixing = SeargeStage()
            stage_processor = self.stage_clip_mixing

        elif stage == self.APPLY_CONTROLNET:
            if self.stage_apply_controlnet is None:
                self.stage_apply_controlnet = SeargeStageApplyControlnet()
            stage_processor = self.stage_apply_controlnet

        elif stage == self.LATENT_INPUTS:
            if self.stage_latent_inputs is None:
                self.stage_latent_inputs = SeargeStageLatentInputs()
            stage_processor = self.stage_latent_inputs

        elif stage == self.SAMPLING:
            if self.stage_sampling is None:
                self.stage_sampling = SeargeStageSampling()
            stage_processor = self.stage_sampling

        elif stage == self.LATENT_DETAILER:
            if self.stage_latent_detailer is None:
                self.stage_latent_detailer = SeargeStageLatentDetailer()
            stage_processor = self.stage_latent_detailer

        elif stage == self.VAE_DECODE_SAMPLED:
            if self.stage_vae_decode_sampled is None:
                self.stage_vae_decode_sampled = SeargeStageVAEDecodeSampled()
            stage_processor = self.stage_vae_decode_sampled

        elif stage == self.HIGH_RESOLUTION:
            if self.stage_high_resolution is None:
                self.stage_high_resolution = SeargeStageHighResolution()
            stage_processor = self.stage_high_resolution

        elif stage == self.HIRES_DETAILER:
            if self.stage_hires_detailer is None:
                self.stage_hires_detailer = SeargeStageHiresDetailer()
            stage_processor = self.stage_hires_detailer

        elif stage == self.VAE_DECODE_HI_RES:
            if self.stage_vae_decode_hi_res is None:
                self.stage_vae_decode_hi_res = SeargeStageVAEDecodeHires()
            stage_processor = self.stage_vae_decode_hi_res

        elif stage == self.UPSCALING:
            if self.stage_upscaling is None:
                self.stage_upscaling = SeargeStageUpscaling()
            stage_processor = self.stage_upscaling

        elif stage == self.IMAGE_SAVING:
            if self.stage_image_saving is None:
                self.stage_image_saving = SeargeStageImageSaving()
            stage_processor = self.stage_image_saving

        else:
            print("WARNING: implementation for stage " + stage + " is missing!")

        # no stage processor exists, so no processing can happen and no result exists
        if stage_processor is None:
            return (data, None,)

        # get the stage input data that is relevant to the selected stage
        stage_input = stage_processor.get_input(data, stage_output)

        # process the selected stage
        stage_result = None
        if stage_input is not None:
            (data, stage_result) = stage_processor.process(data, stage_input)

        # if we got a result from this stage, put it on the data stream
        if has_data:
            data["stage_output"] = stage_result

        return (data, stage_result,)

    def process(self, stage, input_from, output_to, data=None, custom_input=None):
        if data is None:
            data = {}

        stage_input = None
        custom_output = None

        # input from custom stage ?
        if input_from == self.CUSTOM_AND_DATA:
            stage_input = custom_input

        # if no stage data is provided, the stage will take it from the data stream
        if PipelineAccess(data).is_pipeline_enabled():  # or stage == self.LOAD_CHECKPOINTS:
            (data, stage_result) = self.run_stage(stage, data, stage_input)
        else:
            stage_result = None

        # output to custom stage ?
        if output_to == self.CUSTOM_AND_DATA:
            custom_output = stage_result

        # the result will always be on the data stream, so even without custom output it will be passed on
        return (data, custom_output,)

```
