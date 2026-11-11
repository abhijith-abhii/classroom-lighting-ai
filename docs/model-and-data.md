# Model and dataset card

YOLO11n weights are used as pretrained; no detector weights were trained or fine-tuned. ONNX exports use static 320 and 640 input sizes, float32, opset 17, CPU execution, two inference threads, argmax class selection and per-class NMS at IoU 0.45. Thresholds 0.15, 0.25, 0.40 and 0.55 were compared on 42 validation images; 0.25 was selected before evaluating 78 test images. Resolution is the measured model improvement; temporal and sensor policies are evaluated separately.

Nine of ten synthetic framed-photo composites still produced person candidates. The target for lighting is a physically present person, but COCO labels and the pretrained detector represent visual people; they do not establish liveness. Configured depiction zones mark ambiguity, and cannot discover every poster or screen. Tracking can also track a poster. No real classroom poster, seated-person, bags, crowd or screen-specific accuracy was measured. Image splits lack recording IDs; crowd annotations are excluded under a documented nonstandard metric.

## Evaluation boundaries

Model confidence is not accuracy. Detector scores use visual object annotations. Classroom physical presence and waste composition need different labels. A real-video evaluation must hold out complete recordings, sites and acquisition sessions; this project has no labeled video test set. Synthetic trajectories test tracker mechanics only.

The detector dataset contains 120 metadata-selected COCO val2017 images: 40 people, 40 vehicle and 40 negative-stratum images, partitioned into 42 validation and 78 test images. COCO val2017 is familiar to model developers, so this is a reproducible local comparison rather than an independent deployment benchmark. Crowds are excluded from matching and may still produce false positives. See the manifest and per-class JSON.

The waste baseline uses a 48-feature HSV histogram with a trained linear classifier. The final uses 576 pretrained neural features with a newly trained head. Both use the same data split. Threshold selection is rule tuning; tracking and fusion are control changes; neither trains network weights. Float32 inference was used; no quantization was claimed.

## Sources and rights

- YOLO11: https://docs.ultralytics.com/models/yolo11/ and https://github.com/ultralytics/ultralytics (AGPL-3.0).
- Original weights: https://github.com/ultralytics/assets/releases/download/v8.3.0/yolo11n.pt
- COCO: https://cocodataset.org/#download and https://cocodataset.org/#termsofuse . Annotations are CC BY 4.0; individual photos retain their Flickr licenses. This package does not redistribute COCO photographs. Sample manifests retain original source URLs and license IDs.
- TrashNet by Gary Thung and Mindy Yang: https://github.com/garythung/trashnet and author-hosted https://huggingface.co/datasets/garythung/trashnet (MIT). Six small credited examples are included in the waste package.
- MobileNetV3 implementation: https://docs.pytorch.org/vision/stable/models/generated/torchvision.models.mobilenet_v3_small.html . Torchvision code uses BSD-3-Clause; pretrained ImageNet weights have separate dataset provenance. Download through the official setup tool; no backbone weights are redistributed in source packages.

Model hashes and frozen selections are retained under evidence. Original decoder results are retained separately from corrected results; the decoder bug was corrected without changing selected hyperparameters, and the same held-out evaluation was rerun. This test-set reuse is disclosed in FAILURES.md.
