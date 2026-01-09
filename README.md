# ⚽ Soccer Object Detection

YOLOv8 기반 축구 경기 영상 내 객체 탐지 성능 최적화 분석 실험

## 1. 프로젝트 개요

본 프로젝트는 축구 경기 영상에서 선수, 공, 심판 등의 객체를 정확하게 탐지하기 위해 '모델 크기'와 '입력 해상도'가 성능에 미치는 영향을 실험적으로 '비교 분석'합니다.

## 2. 실험 목적

모델 (Yolov8n vs Yolov8s)과 이미지 해상도(640px vs 1280px)의 변화가 축구 경기 내 소형 이미지 객체(공, 선수)에 대한 탐지 정확도(mAP50)에 미치는 인과관계를 비교하여 분석합니다.

## 3. 실험 환경

### Dataset
- **출처**: SelectStar Fitogether 축구 데이터
- **데이터셋 링크** : https://open.selectstar.ai/ko/fitogether
- **데이터 파일** : fittogether.zip (약 3.4 GB)
- **파일 수**: *.json, *.jpg 파일 각각 11,150개
- **클래스 정의**:
  - `B` (Ball): 축구공
  - `Ta` (Team A): A팀 선수
  - `Tb` (Team B): B팀 선수
  - `R` (Referee): 심판
  - `O` (Others): 기타 객체

- **✍️ 사용한 데이터양**: 전체 데이터의 10%
- **✍️ 분할 비율** = train : valid : test = 7 : 1 : 2

### Local PC의 GPU 활용
- VScode에서 Google Colab 실행
- local PC의 장치 확인: cuda 설정 활용
- GPU 모델: NVIDIA GeForce RTX 4070 Laptop GPU

### Models & Configurations
| Model | Image Resolution | mAP50 result | Patience / Epochs |
|-------|-----------|------------|----------------|
| YOLOv8n | 640px | ~0.463 < 0.5 | 10 / 50|
| YOLOv8s | 640px | ~0.525 > 0.5 | 10 / 50|
| YOLOv8s | 1280px | ~0.623 > 0.5 | 10 / 50|

- 먼저 640 해상도 이미지를 활용하여, YOLOv8s 모델이 더 적합함을 확인함.
- 선택한 YOLOv8s 모델을 활용하여, 1280 해상도 이미지를 학습하여 mAP50 값이 더 개선되었음을 확인함. 

## 4. 실험 결과

### 🔎 정량적 성능 분석 : Performance Analysis

<img width="1400" height="800" alt="Step_Comparison_E50" src="https://github.com/user-attachments/assets/47b2741c-01c2-441e-a833-34c8ac55fe32" />


**✅ 주요 결과**:

1. **해상도의 결정적 영향**: 
   - `yolov8s_SZ1280` 모델이 mAP50 ~ 0.6 > 0.5 달성
   - 640 해상도 모델들은 mAP50 = 0.4 ~ 0.5 구간에서 수렴

2. **모델 규모의 영향**:
   - 이미지가 동일한 640 해상도에서 YOLOv8s가 YOLOv8n보다 **약 10% 높은 정확도** 를 가진다.

3. **수렴 속도**:
   - 고해상도 모델은 Epochs ~ 100 이내에 목표 성능(mAP50 = 0.5) 달성


### 🔎 정성적 탐지 결과 : Detection Results Comparison

<img width="768" height="979" alt="Resolution_Comparison_Result" src="https://github.com/user-attachments/assets/5ee30734-b3f0-4f34-acbd-57a64d7469c4" />


**✅ 시각적 분석**:

- **분해능 향상**: 1280 해상도에서 원거리 선수 및 소형 축구공 탐지 성능 대폭 개선
- **신뢰도 증가**: 고해상도 입력으로 인한 Confidence Score 전반적 상승
- **소형 객체 손실 방지**: 640에서 누락되던 원거리 객체들이 1280에서 정확히 분류

## 5. 정리

### 1. 해상도 임계값의 중요성
축구와 같은 **광각 촬영 영상**에서는 모델 크기를 늘리는 것보다 **입력 해상도를 1280 이상**으로 확보하는 것이 성능 향상에 더 지배적인 영향을 미칩니다.

### 2. 연산 효율성
YOLOv8s 모델은 Nano 대비 약간의 연산량 증가만으로도 유의미한 정확도 이득을 제공하므로, **실시간성이 보장되는 한 Small급 이상의 모델 사용 권장**

### 3. 향후 개선 방향
- 현재 ID 기반 임시 팀 분류(홀짝)를 **유니폼 색상 히스토그램 분석** 기반으로 전환
  -> `Ta`와 `Tb`의 오분류율 감소 예상
- 본 프로젝트에서는 전체 데이터셋의 10% 만 학습했으므로, 데이터셋 모두를 학습하여 추가 결과를 비교하는 과제가 가능하다.
  -> 특히, 더 많은 수의 1280 해상도 이미지를 사용하면, 상대적으로 작은 이미지 객체인 축구공(Ball)의 탐지에 대한 학습이 더 잘 될 것으로 예상한다.

## 6. 사용 방법

```bash
# 환경 설정
- pip install ultralytics pyyaml pandas matplotlib seaborn opencv-python 실행
- Data/fittogether 폴더 안에 raw files를 넣고 *.json, *.jpg 파일들이 각각 11150개가 있는 것을 확인

# 소스 코드
- SelectStar Fitogether.ipynb

# 상세 설명
- memo.txt

# 결과 출력 폴더 및 파일
- runs/Step1_yolov8n_SZ640_E50
- runs/Step1_yolov8s_SZ640_E50
- runs/Step2_Winner_yolov8s_SZ1280_E50
- Final_Comparison_Report.csv
- Resolution_Comparison_Result.png
- Step_Comparison_E50.png
etc

```

## 7. 참고 문헌

- [Ultralytics YOLOv8 Documentation](https://docs.ultralytics.com/)
- [SelectStar Fitogether Dataset](https://selectstar.ai/)

---
