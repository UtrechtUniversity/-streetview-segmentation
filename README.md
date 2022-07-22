# Streetview segmentation
Semantic image segmentation with Facebook's Mask2Former models.

## Description
This script performs semantic segmentation on images, using models from Facebook's collection of Mask2Former 
models. Output consists of a CSV-file detailing, for each input image, the number of pixels in each semantic
class in that image. Optionally, it can save a copy of each input image overlaid with the semantic segmentation.
If the input images are 360° photo's, the script provides the possibility of tranforming them by projecting them
onto a cube, resulting in six images per input image.

This script is specifically designed to run on a computer without a GPU. Some of the underlying libraries
require the presence of CUDA-drivers to run, even if the actual device is absent. As it can be problematic to
install such drivers on a computer without an actual GPU, the program is packaged as Docker-container, based on
an official NVIDIA-image, which comes with pre-installed drivers.

## Building the docker container
Check out this repository, and run to build:
```bash
docker build -t my_tag:latest -f Dockerfile .
```

## Getting model files

Models consist of two parts: configuration files, and a file with weights. The configuration files are located in
the Mask2Former repository, which is included in the container at build time. Weights files have not been built
into the container, and must be downloaded and made available to the script in the container.

After selecting the appropriate model configuration files and downloading the corresponding file with model weights,
the location of both needs to registered in a configuration file. For example:

```json
{
    "path_model_cfg" : "/configs/coco/panoptic-segmentation/swin/maskformer2_swin_large_IN21k_384_bs16_100ep.yaml",
    "path_model_weights" : "/data/model/maskformer2_swin_large_IN21k_384_bs16_100ep/model_final_f07440.pkl"
}
```

Save this to a configuration file, which will be passed to the script using the `--config` parameter.

Please note that the software has been tested with a limited number of models, specifically one from the Mapillary Vistas
Panoptic Segmentation collection (model_id 49189528_0), one from the COCO Panoptic Segmentation collection (47429163_0),
and one from the Cityscapes set (48318254_2). Generally, the Swin-L based models seem to work properly.

### Model configuration
Go the [Mask2Model Model Zoo](https://github.com/facebookresearch/Mask2Former/blob/main/MODEL_ZOO.md) and pick
a model to use. Find the path of the model's configuration file by clicking the 'Mask2Former' link for the appropriate
model in the Model Zoo-table. This leads to the corresponding configuration file in the Mask2Former repository.
Extract the path of that file relative to the repository's root, as it would be when the repository were checked out.

Example: the first model from the COCO Model Zoo, Panoptic Segmentation-table (model_id 47430278_4) links to [its configuration
in the Mask2Former repo](https://github.com/facebookresearch/Mask2Former/blob/main/configs/coco/panoptic-segmentation/maskformer2_R50_bs16_50ep.yaml),
the path of which, when checked out, would be:

`Mask2Former/configs/coco/panoptic-segmentation/maskformer2_R50_bs16_50ep.yaml`

This would be entered into the config file as:
```json
"path_model_cfg" : "/configs/coco/panoptic-segmentation/maskformer2_R50_bs16_50ep.yaml",
```


### Weights-file
A file with model weights needs to be downloaded and made available in the container. Click the appropriate 'model'-link in the
Model Zoo-table to download the pickle-file containing the model. In the example above, this is `model_final_94dc52.pkl`.

Place the file in a folder that will be mapped into the container, and edit the config file accordingly:

```json
"path_model_weights" : "/data/model/model_final_94dc52.pkl",
```
Note that this should be the path as it is 'seen' from inside the container.


## Running a job
```bash
docker run -v /local/path/to/data:/data --rm -it my_tag:latest \
	--config "/data/model/config.json" \
	--input "/data/images" \
	--transform360 \
	--transform360exclude "5" \
	--save-segmentation-images \
	--suppress-warnings
```
The parameter `-v` maps a host directory to one inside the container, allowing the container acceess to files on the host computer. Mappings have the form of `<path on host>:<path in container>`; it's advised to always use '/data' for the second part.

Parameters are as follows:

+ **config**: path to a JSON-file containing the model paths (mandatory).
+ **input**: folder with images (mandatory). The folder is read non-recursively.
+ **transform360**: whether the program has to tranform the photo's from 360° to six cube projections (default: False). If your images should be processed _as is_, skip this flag. Transformed images are written to a subdirectory for each 360° image.
+ **transform360exclude**: comma-separated list of sides to exclude from the transformation. sides: 0 = left most, 1 = middle left, 2 = middle right, 3 = right most, 4 = top, 5 = bottom. for instance, `--transform360exclude "4,5"` outputs four projected images, omitting the cube's top and bottom.
+ **save-segmentation-images**: whether the program has to save a copy of each image with the segmentation as overlay (default: False). These images are written to a subfolder called 'segmentations' amd can be useful for checking results at a glance. Numerical output is always written to a CSV-file in the input image folder.
+ **suppress-warnings**: suppresses some of the warnings generated by the Mask2Former software (default: False).

## Used libraries 
+ [Mask2Former](https://github.com/facebookresearch/Mask2Former)
+ [detectron2](https://github.com/facebookresearch/detectron2)
+ [three60cube](https://pypi.org/project/three60cube/)


