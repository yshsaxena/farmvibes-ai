# ml/segment_anything/prompt_segmentation

```yaml

name: prompt_segmentation
sources:
  input_raster:
  - sam_inference.input_raster
  input_geometry:
  - sam_inference.input_geometry
  input_prompts:
  - ingest_points.user_input
sinks:
  segmentation_mask: sam_inference.segmentation_mask
parameters:
  model_type: vit_b
  spatial_overlap: 0.5
tasks:
  ingest_points:
    workflow: data_ingestion/user_data/ingest_geometry
  sam_inference:
    op: prompt_segmentation
    op_dir: segment_anything
    parameters:
      model_type: '@from(model_type)'
      spatial_overlap: '@from(spatial_overlap)'
edges:
- origin: ingest_points.geometry
  destination:
  - sam_inference.input_prompts
description:
  short_description: Runs Segment Anything Model over a Sentinel-2 raster with points
    and/or bounding boxes as prompts.
  long_description: This workflow splits the input raster into chips of 1024x1024
    pixels with an overlap defined by `spatial_overlap`. Chips intersecting with prompts
    are processed by SAM's image encoder, followed by prompt encoder and mask decoder.
    Before running the workflow, make sure the model has been imported into the cluster
    by running `scripts/export_sam_models.py`. The script will download the desired
    model weights from SAM repository, export the image encoder and mask decoder to
    ONNX format, and add them to the cluster. For more information, refer to the [FarmVibes.AI
    troubleshooting](https://microsoft.github.io/farmvibes-ai/docfiles/markdown/TROUBLESHOOTING.html)
    page in the documentation.
  sources:
    input_raster: Sentinel-2 raster to be segmented.
    input_geometry: Geometry of interest for the segmentation.
    input_prompts: ExternalReferences to the point and/or bounding box prompts. These
      are GeoJSON with coordinates, label (foreground/background) and prompt id (in
      case, the raster contains multiple entities that should be segmented in a single
      workflow run).
  sinks:
    segmentation_mask: Output segmentation mask.
  parameters:
    model_type: Type of Visual Transformer (ViT) used as backbone architecture for
      SAM's image encoder, among 'vit_h', 'vit_l', or 'vit_b'. Make sure the desired
      model has been imported into the cluster before running the workflow.
    spatial_overlap: Spatial overlap between chips in the range of [0.0, 1.0).


```

```{mermaid}
    graph TD
    inp1>input_raster]
    inp2>input_geometry]
    inp3>input_prompts]
    out1>segmentation_mask]
    tsk1{{ingest_points}}
    tsk2{{sam_inference}}
    tsk1{{ingest_points}} -- geometry/input_prompts --> tsk2{{sam_inference}}
    inp1>input_raster] -- input_raster --> tsk2{{sam_inference}}
    inp2>input_geometry] -- input_geometry --> tsk2{{sam_inference}}
    inp3>input_prompts] -- user_input --> tsk1{{ingest_points}}
    tsk2{{sam_inference}} -- segmentation_mask --> out1>segmentation_mask]
```