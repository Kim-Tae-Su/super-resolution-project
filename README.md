# Super Resolution

## 1. 프로젝트 개요

- 프로젝트명: Super Resolution
- 프로젝트 소속: 고영테크놀러지
- 프로젝트 기간: 2024.09 ~ 2025.01
- 프로젝트 인원: 2명

### Super Resolution 개요
Super Resolution(SR)은 저해상도 이미지를 입력으로 받아  
고해상도 이미지를 복원하는 이미지 복원(Image Restoration) 기술이다.
<img src="images/new_example.png" width="600"/>

### 프로젝트 목표
고영테크놀러지의 반도체 검사 장비는
정확한 위치 제어 및 Motion 이동을 위해 Machine Calibration(Mcal) 이미지를 활용한다.

기존 Calibration 이미지 생성 방식은
고해상도 이미지를 직접 측정해야 하므로 생성 시간이 오래 걸리고,
이로 인해 장비 운영 효율 및 생산성이 저하되는 문제가 있었다.

본 프로젝트에서는 저해상도(LR) Calibration 데이터를 기반으로
고해상도(HR) 이미지를 복원하는 딥러닝 기반 Super-Resolution 모델을 개발하여,
Calibration 이미지 생성 시간 단축, 장비 Calibration 정확도 향상, 생산 효율 및 장비 운영 효율 개선을 목표로 하였다.


### 기술 선정 배경
반도체 검사 장비 특성상 미세한 패턴과 Edge 정보를 정밀하게 복원해야 하므로,
단순 Interpolation(Bicubic 등) 기반 이미지 확대 방식만으로는
필요한 수준의 정확도를 확보하기 어려웠다.

이에 따라 저해상도 이미지로부터 고주파 특징(High-Frequency Feature)과
세부 패턴을 효과적으로 복원할 수 있는
딥러닝 기반 Super-Resolution 기술을 도입하였다.

다만 딥러닝 기반 SR 모델은 복원 품질이 향상되는 대신
모델 복잡도 증가에 따른 추론 속도(Inference Time) 및 연산 비용 증가라는 Trade-off가 존재하였다.

정확도 향상뿐 아니라 실시간 처리 성능과 장비 운영 효율도 중요했기 때문에,
복원 품질(PSNR, SSIM)과 추론 속도(Inference Time) 간 균형을 고려하여 모델을 최적화하였다.


---

## 2. 담당 역할
### 데이터 로딩
- 데이터 수집
- 자사CRM 크롤링 스크립트 작성 

### 데이터 로딩
- 시퀀스 기반 데이터 로더 설계
- 이미지 파일을 NumPy 배열 형태로 변환

### 데이터 전처리
- 저해상도(LR) 및 고해상도(HR) 이미지 생성
- 노이즈 제거 및 Scaling 처리

### 모델 설계 및 구현
- Super-Resolution 모델 후보 조사 및 선정
- 관련 논문 분석
- SR 모델 구조 구현
<img src="images/srcnn.png" width="600"/>

### 모델 학습 및 튜닝
- 학습 / 검증 / 테스트 데이터셋 분할
- 하이퍼파라미터 튜닝
- 최적화 기법 적용

### 모델 검증 및 테스트
- PSNR, SSIM 지표를 활용한 복원 성능 평가
- 복원된 HR 이미지와 실제 장비 오차 이미지 비교 분석
- 실제 장비 적용 가능성 검증
  
### 파이프라인
<img src="images/pipeline.png" width="800"/>

---

## 3. 기술적 문제

### 문제 원인
Calibration 이미지는 일반적인 RGB 이미지와 달리,  
**이미지마다 픽셀 값의 분포 범위가 크게 상이한 특성**을 가진다  
(예: `[-50, 50]`, `[-200, 200]`).

이러한 특성을 고려하지 않고 **전체 데이터셋 기준 Min-Max 정규화**를 적용할 경우,  
분포 범위가 좁은 이미지의 픽셀 값은 과도하게 축소되고,  
범위가 넓은 이미지의 픽셀 값은 상대적으로 과장되는 문제가 발생하였다.

그 결과 모델이 **특정 이미지 분포에 편향(bias)** 되어 학습되었으며,  
일부 샘플에 대해 복원 성능이 급격히 저하되는 현상이 발생하였다.


### 해결 방법
위 문제를 해결하기 위해 **이미지 단위(Image-wise) Min-Max Scaling** 방식을 적용하였다.  
각 이미지를 독립적으로 정규화하여,  
모든 입력 이미지를 동일한 `[0, 1]` 범위로 변환함으로써  
데이터 분포 차이에 따른 학습 편향을 제거하였다.

#### 정규화 수식
X_norm = (X - X_min) / (X_max - X_min)

#### 복원 수식
X_restore = X_pred * (X_max - X_min) + X_min

---

## 4. 결과 및 성과

- Mcal 오차 이미지 파일 생성 시간 약 90% 단축
- 고정밀 장비 보정 정확도 향상
- 총 5종의 Super-Resolution 모델 설계 및 시뮬레이션 수행
- 최적 SR 모델 선별
- SR 기술 기반 실시간 보정 시스템 초기 기반 구축
- 기술 제안 → 설계 → 구현 → 적용까지 End-to-End 프로세스 수행


### Error Vector 기반 검증 결과

<p align="center">
  <img src="images/result.png" width="400"/>
  <img src="images/error_vector.png" width="400"/>
</p>

- 좌측: Super-Resolution 기반 Calibration 이미지 복원 결과
- 우측: Error Vector 개념도  
  (Neural Network 예측값과 실제 Offset 간 차이를 벡터로 표현)
- SR 적용 후 Error Vector 분포가 감소하여 장비 보정 정확도 향상을 확인함
---

## 5. 사용 기술

- Language
  - Python
- Deep Learning
  - Keras
  - TensorFlow
- Data Processing
  - NumPy
  - OpenCV

---

## 6. 프로젝트 의의

본 프로젝트는 Super-Resolution 모델 구현에 그치지 않고,  
실제 장비 환경의 데이터 특성과 물리적 해석 가능성을 고려하여  
현업 적용이 가능한 딥러닝 기반 Calibration 시스템을 구축하였다.
