# Comfy Workflows (Animagine XL 4.0)

`txt2img` / `img2img` 워크플로 저장소. 모델 바이너리는 포함하지 않음.

## 워크플로

| 파일 | 흐름 |
| --- | --- |
| `workflows/txt2img.json` | `EmptyLatent(832×1216)` → `KSampler 1` → `LatentUpscaleBy(×1.5)` → `KSampler 2` → `VAE Decode` → `FaceDetailer(얼굴)` → `FaceDetailer(손)` → `Save` |
| `workflows/img2img.json` | `LoadImage` → `VAEEncode` → `KSampler 1(denoise 0.6)` → 이후 txt2img와 동일(Hires.fix + Detailer ×2) → `Save` |

최종 해상도 약 1248×1824.

## 필수 노드 (ComfyUI Manager에서 설치)

* `ComfyUI-Impact-Pack` — `FaceDetailer`
* `ComfyUI-Impact-Subpack` — `UltralyticsDetectorProvider`

## 모델

### 체크포인트

* `animagine-xl-4.0.safetensors` → `ComfyUI/models/checkpoints/`

### 추가한 검출 모델 (이번에 넣은 것)

| 파일 | 위치 |
| --- | --- |
| `face_yolov8s.pt` | `ComfyUI/models/ultralytics/bbox/` |
| `hand_yolov8n.pt` | `ComfyUI/models/ultralytics/bbox/` |

같은 폴더에 기존 `face_yolov8m.pt`, `hand_yolov8s.pt` 공존 가능. `segm/person_yolov8m-seg.pt`는 `models/ultralytics/segm/`에 유지.
출처: ComfyUI Manager Model Manager에서 `ultralytics` 검색, 또는 `Bingsu/adetailer` HuggingFace 저장소.

## Detailer 설정값

| 설정 | 얼굴·머리 | 손 |
| --- | --- | --- |
| 검출기 | `bbox/face_yolov8s.pt` | `bbox/hand_yolov8n.pt` |
| `bbox_threshold` | 0.4 | 0.25 |
| `bbox_dilation` | 20 | 6 |
| `bbox_crop_factor` | 2.2 | 2.2 |
| `guide_size` / `max_size` | 640 / 1024 | 512 / 896 |
| `steps` / `CFG` | 20 / 6.5 | 20 / 6.5 |
| `denoise` | 0.6 | 0.6 |
| `noise_mask` / `force_inpaint` | 켬 / 끔 | 켬 / 끔 |
| `guide_size_for` | `bbox` | `bbox` |
| 샘플러 | `euler_ancestral` / `karras` | 동일 |

두 Detailer 모두 기본 체크포인트의 `model`/`clip`/`vae`와 기본 positive/negative를 재사용.

## Hires.fix 설정값

* `LatentUpscaleBy`: `bislerp`, `1.5`
* `KSampler 2`: 1차와 같은 steps/샘플러, `CFG 8`, `denoise 0.6`
* 변화가 크면 0.45~0.50으로, 밋밋하면 0.60까지 조정.

## 중간 미리보기

* `VAE Decode` 출력 → `PreviewImage` (Hires.fix 직후, Detailer 전)
* `FaceDetailer(얼굴)` 출력 → `PreviewImage`
* 각 Detailer의 `cropped_refined` → `PreviewImage` (보정 크롭 확대 확인용)
* 최종(손 Detailer 출력)은 `Save Image`에서 확인.

## 사용법

1. 위 노드·모델을 ComfyUI에 설치.
2. `workflows/*.json`을 ComfyUI에 드래그하거나 `user/default/workflows/`에 복사.
