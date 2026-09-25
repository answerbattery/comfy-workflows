# Comfy Workflows (Animagine XL 4.0)

`txt2img` / `img2img` 워크플로 저장소. 모델 바이너리는 포함하지 않음.

## 워크플로

| 파일 | 흐름 |
| --- | --- |
| `workflows/txt2img.json` | `EmptyLatent(832×1216)` → `KSampler 1` → `LatentUpscaleBy(×1.5)` → `KSampler 2` → `VAE Decode` → `FaceDetailer(얼굴)` → `FaceDetailer(손)` → `Save` |
| `workflows/img2img.json` | `LoadImage` → `VAEEncode` → `KSampler 1(denoise 0.6)` → 이후 txt2img와 동일(Hires.fix + Detailer ×2) → `Save` |

| `workflows/turnaround_sheet.json` | 캐릭터 이미지 1장 + 5포즈 포즈맵 → 3648×2304 턴어라운드 한 장 (OpenPose ControlNet + IPAdapter + Hires.fix + FaceDetailer + AnimeSharp) |

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

두 Detailer 모두 기본 체크포인트의 `model`/`clip`/`vae`를 공유하지만, conditioning은 전용 프롬프트를 사용.

* 얼굴 Detailer: 얼굴·머리카락·눈 특화 positive/negative (적발·회안·모자·점 유지)
* 손 Detailer: 손·손가락 특화 positive/negative (맨손, 소매-커프스 유지, 장갑 금지)

## Hires.fix 설정값

* `LatentUpscaleBy`: `bislerp`, `1.5`
* `KSampler 2`: 1차와 같은 steps/샘플러, `CFG 8`, `denoise 0.6`
* 변화가 크면 0.45~0.50으로, 밋밋하면 0.60까지 조정.

## 중간 미리보기

* `VAE Decode` 출력 → `PreviewImage` (Hires.fix 직후, Detailer 전)
* `FaceDetailer(얼굴)` 출력 → `PreviewImage`
* 각 Detailer의 `cropped_refined` → `PreviewImage` (보정 크롭 확대 확인용)
* 최종(손 Detailer 출력)은 `Save Image`에서 확인.

## 턴어라운드 시트 (`turnaround_sheet.json`)

* 입력 2개: `CHARACTER` 노드에 내 캐릭터 정면 이미지, 포즈맵은 `input/pose_turnaround_5pose.png`로 저장.
* 흐름: `EmptyLatent(1216×768)` → `KSampler 1` → `LatentUpscaleBy(×1.5)` → `KSampler 2(CFG 8, denoise 0.6)` → `VAEDecodeTiled` → `FaceDetailer(얼굴)` → `AnimeSharp 4x` → `ImageScale(3648×2304)` → `Save`.
* 포즈맵 가로세로비에 맞춰 잠재 크기를 잡았으니 포즈맵을 바꾸면 `EmptyLatent`·포즈 `ImageScale` 크기도 함께 조정.

## 사용법

1. 위 노드·모델을 ComfyUI에 설치.
2. `workflows/*.json`을 ComfyUI에 드래그하거나 `user/default/workflows/`에 복사.
