# 조선왕조 어진 복원 AI 프로젝트 (Ujin Reconstruction 2026)

열성어진·선원보감 삽화처럼 **열화된 전승 이미지**를 입력으로 받아, 가능한 한 **공식 어진 화풍**에 가깝게 복원하는 생성형 AI 프로젝트입니다.

## 1) 문제 정의

조선왕조 공식 어진은 임진왜란·정묘호란·병자호란·화재 등의 사건으로 상당수가 소실되었습니다. 반면 선원보감/열성어진에는 더 많은 왕의 도상이 전하지만, 복식·비례·묘사 품질이 일정하지 않고 원본과 오차가 큽니다.

이 프로젝트의 목표는 다음과 같습니다.

- 입력: 선원보감/열성어진 기반의 저품질 도상
- 출력: 현존 공식 어진 화풍을 최대한 반영한 고해상도 복원 이미지
- 핵심 가정: 태조·영조·철종처럼 **공식 어진–열성어진(혹은 선원보감) 쌍**이 남아 있는 사례를 학습하면, 화풍/비례/복식 왜곡 보정을 일반화할 수 있다.

## 2) 데이터 전략

### 2.1 데이터 그룹

1. **Paired Set (핵심 학습셋)**
   - 공식 어진 ↔ 열성어진/선원보감 1:1 쌍
   - 예시: 태조, 영조, 철종 (추가 발굴 시 확장)
2. **Unpaired Set (보조 학습셋)**
   - 공식 어진 단독 이미지
   - 열성어진/선원보감 단독 이미지
3. **메타데이터**
   - 왕명, 제작 연대, 복식 유형(면복/곤룡포), 좌상/전신, 배경 요소

### 2.2 전처리

- 고해상도 스캔(가능하면 600dpi 이상)
- 정렬(눈·코·입 랜드마크 + 왕좌/신체 기준점)
- 색 표준화(조명 편차 보정)
- 손상/오염 마스크 생성(균열·번짐 영역 분리)

## 3) 모델 설계 (권장)

### 3.1 2단계 파이프라인

- **Stage A: 구조 복원 모델**
  - 목적: 비례·실루엣·복식 형태 교정
  - 후보: U-Net 기반 image-to-image, ControlNet 조건부 변환
- **Stage B: 화풍/세부 묘사 모델**
  - 목적: 채색, 필선 질감, 얼굴 디테일 복원
  - 후보: Diffusion 기반 super-resolution + style adapter

### 3.2 손실 함수

- 재구성 손실: L1/L2
- 지각 손실: LPIPS / VGG perceptual loss
- 화풍 정합 손실: style loss (Gram matrix)
- 얼굴/복식 영역 가중 손실: ROI weighted loss
- 선택: 적대적 손실(GAN loss)로 질감 향상

## 4) 학습 전략

- 쌍 데이터로 supervised pretraining
- unpaired 데이터로 domain adaptation (Cycle consistency 또는 contrastive approach)
- 데이터 증강:
  - 선화 열화, 색 바램, 종이 질감 노이즈
  - 랜덤 기하 변형(과도한 변형은 금지)
- k-fold 혹은 leave-one-king-out 평가로 과적합 점검

## 5) 평가 지표

- 정량:
  - PSNR / SSIM (paired 검증셋)
  - LPIPS / DISTS (지각 품질)
- 정성:
  - 한국 회화사/복식사 전문가 블라인드 평가
  - 평가 항목: 얼굴 유사성, 복식 고증도, 필선 자연성, 역사적 개연성
- 신뢰도:
  - 모델 불확실성 맵(uncertainty map) 동시 제공

## 6) 산출물 정책 (중요)

이 모델 출력은 **원본의 대체물이 아니라 연구용 추정 복원안**으로 취급합니다.

- 모든 결과물에 워터마크/메타데이터 삽입: `AI-assisted reconstruction`
- 전시/출판 시 “사료 기반 추정치” 문구 의무 표기
- 버전 관리로 복원 근거(학습 데이터, 파라미터, 평가 결과) 추적 가능화

## 7) 구현 로드맵

### Phase 1 — 데이터 구축 (1~2개월)
- 소장처 협력 및 스캔 파이프라인 확정
- paired/unpaired 분류 및 메타데이터 스키마 설계

### Phase 2 — 베이스라인 모델 (2개월)
- pix2pix/ControlNet 베이스라인 학습
- 기본 지표 리포트 자동화

### Phase 3 — 고도화 (2~3개월)
- diffusion 기반 세부 복원
- 복식/얼굴 특화 손실 튜닝

### Phase 4 — 전문가 검증 및 공개 (1개월)
- 사학·미술사 자문단 블라인드 평가
- 결과 리포트/데모 아카이브 공개

## 8) 최소 실행 예시 (개념)

```bash
# 1) paired 학습
python train.py --config configs/paired_baseline.yaml

# 2) unpaired adaptation
python train_adapt.py --config configs/domain_adapt.yaml

# 3) 추론
python infer.py --input data/yeolseong/test_yeongjo.png --output outputs/yeongjo_recon.png
```

---

## 프로젝트 한 줄 요약

> “남아 있는 공식 어진–열성어진/선원보감 쌍의 오차를 학습해, 소실된 공식 어진의 시각적 특성을 확률적으로 복원하는 역사문화 AI 연구 프로젝트”
