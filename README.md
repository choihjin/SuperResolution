# Dash cam License Plate Restoration

## 목차
1. [소프트웨어 개요](#소프트웨어-개요)
2. [환경 설정](#환경-설정) 
3. [영상 화질 개선](#영상-화질-개선)
4. [Reference](#Reference)
---

## 소프트웨어 개요

이 문서는 여러 이미지나 영상을 이미지로 바꾸어 인간이 인식 가능하는 수준으로 자동차 번호판 화질을 복원하는 프로그램에 대한 소프트웨어 설정과 실행하는 방법에 대한 지침을 제공합니다.

번호판을 detect하기 위해 DeepLabV3를 기반으로 한 license plate segmentation 모델을 사용하였으며, Optical Flow 추정을 위해서 Flowformer++ 모델을 사용하였습니다. 특정 부분 내용은 보고서 참고 바랍니다.

본 프로그램은 마우스와 모니터가 필요하며 현재 개발이 리눅스 서버에서 진행되어 리눅스 운영체제에서만 프로그램 실행이 가능합니다.

추후 윈도우나 맥으로 구동하는 일반 PC에서도 실행 가능하도록 여러 가상환경 설정을 추가할 예정입니다.

프로그램 수행 절차는 다음 순서로 이루어집니다.

### 1. Crop.sh 실행(모니터/마우스 사용 단계, 운영체제는 상관 無)

input 프레임 단위로 연속되는 이미지들 -> input 이미지들 중에서 화질 개선을 목표로 하는 자동차 번호판을 마우스로 클릭하여 현재 이미지 기준 앞 뒤 각각 15개를 추출하여 총 31개의 특정 자동차 이미지 추출 -> 추출한 자동차 이미지들은 "Data/Car_crop" 폴더에 저장하고 원본 input 이미지들은 "Data/Full_frame" 폴더에 저장

### 2. SR.sh 실행(리눅스 OS 수행 단계)

각 31개 자동차 이미지마다 번호판 인식 후 번호판 이미지만 추출 -> 31개의 번호판 이미지들을 사용하여 Optical Flow 추정 -> 추정 값들에 Optical Flow Refinement 기법 적용 -> De-warping 진행 -> 추가 화질 개선을 위한 postprocess과 aggregation 적용 -> output 화질 개선된 번호판 이미지

+ 추가) 리눅스 PC에서 마우스나 모니터 사용이 불가능하다면 Crop.sh는 마우스와 모니터 사용이 가능한 PC에서 따로 수행하고 Data 폴더를 리눅스 PC로 옮겨서 SR.sh 실행

---

## 환경 설정

프로그램을 실행하기 전에 아래와 같이 환경을 설정해야 합니다.

1. Conda를 사용하여 가상 환경 생성 및 환경 설정
   ```
   conda env create -f environment.yaml
   ```

2. 생성된 가상 환경 활성화
   ```
   conda activate SR
   ```
   
3. 가상 환경 비활성화
   ```
   conda deactivate
   ```

4. model_v2.pth 다운로드
   |    Model     | 
   |--------------|
   |[model_v2.pth](https://drive.google.com/file/d/15pkZ2haNdr6uDVBE2iy8mcXLALLD70qf/view?usp=drive_link) |

+ 다운로드 후 model_v2.pth 파일을 DT_core 폴더로 업로드

---
   
## 영상 화질 개선

예시를 통해 학습된 모델을 이용하여 영상의 화질을 개선하는 단계를 설명합니다.

### Step1. Input images를 ./Data 폴더에 위치
```
.
├── Data
│   ├── Input_1
│   │   ├── image1.png   <- HERE
      …
│   │   └── image31.png  <- HERE
│   ├── Input_2 
│   │   ├── image1.png   <- HERE
      …
│   │   └── image31.png  <- HERE
   …
├── DT_core
│   ├── model_v2.pth
   …
├── FF_core
├── FlowFormerPlusPlus
│   ├── visualize_flow.py
 …
├── Crop.sh
├── SR.sh
├── aggregation.py
├── click.py
├── plate_crop_v3.py
├── sort.py
├── spatial_consist_final.py
├── temporal_consist_final.py
├── upsample.py
├── viz_flow_final.py
└── environment.yaml
 

```

### Input 형식 및 제약
Acceptable image suffixes : `'bmp', 'jpg', 'jpeg', 'png', 'tif', 'tiff', 'dng', 'webp'`</br>
Acceptable video suffixes : `'mov', 'avi', 'mp4', 'mpg', 'mpeg', 'm4v', 'wmv', 'mkv'`
<br></br>

### Step2. Crop.sh 실행
bash 파일 Crop.sh를 실행하여 원하는 차량을 선택한다.
```
sh Crop.sh 
```
bash 코드를 실행하게 되면 순차적으로 코드들을 실행하게 되는데

   - sort.py 파일을 실행하여 Data/Input_# 폴더에 들어있는 이미지 파일들의 이름을 01~31로 변경해준다.
   - click.py 파일을 실행하여 블랙박스 영상 중 원하는 차량을 선택한다. 이를 위해 아래와 같은 이미지가 출력된다.
<img src="https://github.com/HGU-DLLAB/NC-SuperResolution/assets/102033523/c38de1ad-4bad-42a3-98d0-07036353a58e">

이러한 화면이 나오면 여기서 원하는 차량의 번호판 중앙 부분(위 사진에서의 빨간 점)을 한번 클릭후 esc를 눌러 화면을 종료해준다.

**(esc를 누른 후 31 frame을 crop하는데 2초정도 걸리니 esc는 한번만 눌러주세요. 여러번 누르면 뒤에 있는 영상들이 넘어갑니다.)**

<img src="https://github.com/HGU-DLLAB/NC-SuperResolution/assets/102033523/37888fa1-8287-4eeb-8183-c0402f2e45b9">

화면을 종료해주면 위의 이미지처럼 해당 차량을 crop한 이미지가 앞, 뒤 15프레임 간격의 이미지들이 저장되어 총 31장의 이미지를 저장한다.

### Step3. SR.sh 실행
bash 파일 SR.sh를 실행하여 블랙박스 영상에서의 원하는 차량의 번호판 화질을 개선할 수 있다.
```
sh SR.sh 
```

위 작업을 마치면 이후에는 bash코드가 이어서 순차적으로 코드들을 실행하게 되는데
   - plate_crop_v3.py 파일을 실행하여 차량의 번호판을 detect 한다. 
<img src="https://github.com/HGU-DLLAB/NC-SuperResolution/assets/102033523/7354e3ec-341c-442f-8706-115e75164265">

   - upsample.py 파일을 실행하여 위의 detect한 번호판의 크기를 4배정도 upsampling 한다.
   
   - visualize_flow.py 파일을 실행하여 flowformer++를 통해 영상 속 번호판의 움직임을 파악하고 이를 상대 좌표와 최대 radius로 normalize 한다.
 <img src="https://github.com/HGU-DLLAB/NC-SuperResolution/assets/102033523/871bcb33-5478-46b6-91e1-7b47c5ab23bd">
  
  
   - viz_flow_final.py 파일을 실행하여 위의 값들을 절대 좌표로 변환하고 fixed ratio로 visualization 한다.
<img src="https://github.com/HGU-DLLAB/NC-SuperResolution/assets/102033523/16368005-d146-4986-9621-c8eafea0fe00">
  
 
   - temporal_consist_final.py 파일과 spatial_consistency_final.py 파일을 실행하여 움직임이 큰 frame에 대하여 보정하고 기각한다.
<img src="https://github.com/HGU-DLLAB/NC-SuperResolution/assets/102033523/0f982be2-0842-4a12-9642-e6357355a688">

위 viz_flow_final.py를 통해 얻은 것보다 일정한 값들을 가지고 있음을 확인할 수 있다.
  
  
   - aggregation.py 파일을 실행하여 최종적으로 화질이 개선된 이미지들을 얻게 된다.
<img src="https://github.com/HGU-DLLAB/NC-SuperResolution/assets/102033523/2353dec6-a766-4cbb-b602-e8f7d960adcf">


### Step4. ./Data/Output/ 하위 폴더에서 결과 확인
```
.
├── All
│   ├── Input_1_foreground.png
│   ├── …
├── All_CLAHE
│   ├── Input_1_foreground_CLAHE.png
│   ├── …
├── EA
│   ├── Input_1
│   │   ├── Input_1_foreground.png
│   │   ├── Input_1_foreground_CLAHE.png
│   ├── Input_2
      …
└── 

```
- All 폴더는 보고서 목차 2.1.7에서 설명한 aggregation이 적용된 이미지

- All_CLAHE 폴더는 All 폴더 이미지에 추가로 CLAHE 모델을 사용하여 영상의 대비 효과를 더 크게 하여 화질을 개선한 이미지

- EA 폴더는 위 All과 All_CLAHE 이미지들을 비교하기 쉽게 동일한 번호판끼리 묶어놓은 폴더.


---
 
## Reference
[1] Bappert, D. (2020, May 3). Pytorch-licenseplate-segmentation: Pretrained Pytorch License Plate segmentation model (DeepLabV3 with ResNet-101 backbone). GitHub. https://github.com/dbpprt/pytorch-licenseplate-segmentation 


[2] Shi, X. (2023, July 1). Flowformerplusplus: Flowformer++: Masked cost volume autoencoding for Pretraining Optical Flow Estimation. GitHub. https://github.com/XiaoyuShi97/FlowFormerPlusPlus 
