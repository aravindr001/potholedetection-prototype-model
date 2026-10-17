# Pothole Detection with YOLOv8s

Object-detection model that locates **potholes** in road-surface images using
[Ultralytics YOLOv8s](https://docs.ultralytics.com), trained end-to-end on a
Roboflow dataset (1 class).

## Results

Evaluated on the held-out **test** split (123 images, 460 instances) using the
best checkpoint from training.

| Metric | Value |
| --- | --- |
| Precision | 0.696 |
| Recall | 0.567 |
| mAP@50 | 0.650 |
| mAP@50-95 | 0.317 |

Reference — best epoch (66) on the **validation** split:

| Precision | Recall | mAP@50 | mAP@50-95 |
| --- | --- | --- | --- |
| 0.646 | 0.570 | 0.625 | 0.304 |

Training curves, confusion matrices and validation batches:
`runs/detect/yolov8s_pothesis100/`

## Dataset

Roboflow export: `pothole detection using yolov8.v2i.yolov8/`

| Split | Images | Labels |
| --- | --- | --- |
| train | 869 | 869 |
| valid | 248 | 248 |
| test | 123 | 123 |

- Classes: 1 (`0` — pothole)
- License: CC BY 4.0

> The original `data.yaml` ships with Roboflow-style relative paths
> (`../train/images`) that do not resolve in this layout. A corrected copy,
> `data_local.yaml`, points at the actual split directories. Original dataset
> files were not modified.

## Project structure

```
.
├── data_local.yaml                     # corrected dataset config (used for training)
├── pothole detection using yolov8.v2i.yolov8/
│   ├── data.yaml                       # original Roboflow config
│   ├── train/ valid/ test/             # images + labels
├── runs/detect/
│   ├── yolov8s_pothesis100/            # training run (100 epochs)
│   │   ├── weights/best.pt             # best checkpoint
│   │   ├── weights/last.pt
│   │   ├── results.csv, results.png
│   │   └── confusion_matrix*.png, Box*_curve.png
│   ├── yolov8s_pothesis100_test/       # test-split evaluation
│   └── predict_test/                   # annotated prediction outputs
└── train_log.txt                       # raw training log
```

## Training

- Model: `yolov8s.pt` (pretrained)
- Epochs: 100, image size 640, batch size: auto (`batch=-1`)
- Device: Apple MPS (Apple M1)
- Total time: 10.78 h

```bash
yolo detect train \
  model=yolov8s.pt \
  data=data_local.yaml \
  epochs=100 imgsz=640 batch=-1 \
  device=mps \
  project=runs/detect name=yolov8s_pothesis100
```

## Evaluation (test split)

```bash
yolo detect val \
  model=runs/detect/yolov8s_pothesis100/weights/best.pt \
  data=data_local.yaml split=test \
  project=runs/detect name=yolov8s_pothesis100_test
```

## Inference

```bash
yolo detect predict \
  model=runs/detect/yolov8s_pothesis100/weights/best.pt \
  source=path/to/images conf=0.25 \
  project=runs/detect name=predict_test save=True
```

Annotated examples are saved in `runs/detect/predict_test/`.

## Environment

- Ultralytics 8.4.142
- PyTorch 2.14.0
- Python 3.11

```bash
pip install ultralytics
```

## Hardware notes (8 GB Apple M1)

- Auto device selection falls back to CPU on this build; MPS must be requested
  explicitly (`device=mps`).
- `batch=-1` AutoBatch only supports CUDA and therefore used the default batch
  of 16, which exceeded available unified memory and caused heavy swap. The run
  used `batch=8` instead, which fit comfortably and roughly tripled throughput.
- `caffeinate -i` was used during training to prevent macOS idle sleep.

## License

Dataset: CC BY 4.0 (see `README.dataset.txt` / `README.roboflow.txt`).
