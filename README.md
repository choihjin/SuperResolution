# Dash cam License Plate Restoration

## 프로젝트 개요
블랙박스 영상에서 흐릿한 자동차 번호판을 고화질로 복원하는 딥러닝 기반 화질 개선 프로젝트입니다.

## 주요 기능
- DeepLabV3 기반 번호판 세그멘테이션
- FlowFormer++를 활용한 Optical Flow 추정
- 시공간적 일관성을 고려한 화질 개선
- CLAHE를 통한 대비 향상

## 기술 스택
- Python
- PyTorch
- DeepLabV3
- FlowFormer++
- OpenCV

## 프로젝트 구조
```
.
├── Data/               # 입력/출력 데이터 저장
├── DT_core/           # DeepLabV3 모델 관련 코드
├── FF_core/           # FlowFormer++ 관련 코드
├── FlowFormerPlusPlus/# Optical Flow 추정 코드
└── scripts/           # 실행 스크립트
```

## 핵심 알고리즘
1. **번호판 검출**
   - DeepLabV3를 사용하여 프레임별 번호판 영역 검출
   - 검출된 번호판 영역 크기 4배 업샘플링

2. **움직임 추정**
   - FlowFormer++를 통한 Optical Flow 추정
   - 시공간적 일관성 검사를 통한 움직임 보정

3. **화질 개선**
   - 움직임 정보를 활용한 프레임 정렬
   - CLAHE를 통한 대비 향상
   - 최종 이미지 집계 및 복원

## 결과 예시
[이미지 삽입 위치]

## 개발 환경
- OS: Linux
- Python 3.8+
- CUDA 지원 GPU

## 설치 및 실행
1. 환경 설정
```bash
conda env create -f environment.yaml
conda activate SR
```

2. 모델 다운로드
- [model_v2.pth](https://drive.google.com/file/d/15pkZ2haNdr6uDVBE2iy8mcXLALLD70qf/view?usp=drive_link)를 DT_core 폴더에 저장

3. 실행
```bash
# 번호판 영역 추출
sh Crop.sh

# 화질 개선
sh SR.sh
```

## 참고 문헌
[1] Bappert, D. (2020). Pytorch-licenseplate-segmentation: Pretrained Pytorch License Plate segmentation model (DeepLabV3 with ResNet-101 backbone). GitHub.

[2] Shi, X. (2023). Flowformerplusplus: Flowformer++: Masked cost volume autoencoding for Pretraining Optical Flow Estimation. GitHub.

---
*이 프로젝트는 포트폴리오 용도로만 공개되었습니다.*
