# NeRF-Review


## NeRF의 후속 연구

1. Training, rendering 속도 개선 : Instant NeRF (SIGGRAPH 2022)
```
문제점 : NeRF는 V100 GPU 에서 학습에 1일 이상 소요되고, Rendering 1장에 30초정도 소요됩니다. 실시간 Application 서비스를 할 수 없습니다.
```
- Feature Encoding 방법 : HashMap, Linear Interpolation 으로 좌표 Encoding 방법 제안
  - (1) 주어진 x좌표에 대해, Grid Level별로 주변 4개의 좌표를 선택한다.
  - (2) 선택된 좌표들로 HashKey값을 만들어, HashTable에서 Value들을 읽는다.
  - (3) x좌표와 주변 4개 좌표의 거리를 기반으로 4개의 Value들을 Weighted Sum하여, 1개의 Feature Vector로 만든다.
  - (4) 각 Grid Level별 Feature Vector들과 Auxiliary값(ex, view direction)들을 Concat하여 최종 Feature Vector로 만든다.
  - (5) MLP를 통과한다.


2. **입력 이미지수를 축소** : PixelNeRF (CVPR 2021)
```
문제점 : NeRF는 50장 이상의 입력 이미지에 대해 optimization하여 Model을 생성합니다. 실용적인 서비스를 만들기에는 너무 많은 입력 이미지가 요구됩니다. 적은 수의 입력값으로 Model을 생성하는 모델을 생각 해 볼 수 있습니다.
```
- CNN Encoder를 사용함 : input image → feature grid 로 만듦
  → 적은 수의 Input Image로 다양한 View Direction을 Rendering 가능.
  
  
3. BARF : Bundle-Adjusting Neural Radiance Fields (ICCV2021)
BARF는 Original NeRF의 카메라 위치를 모를 경우 정확도가 매우 떨어진다는 점을 개선하여 카메라 pose(위치)가 없이도 3D Representation이 가능한 모델이며, 수학적으로 Camera Pose와 3D Representation을 최적화 하는 방법에 대해서 제안합니다. 기존 NeRF 모델에서 모델링 방법과 Positional Encoding 방법 등을 변경하고 모델의 구조를 수정하여 사용함으로써 Camera Pose 없이도 3D Representation을 가능하게 합니다.
  
4. Point-NeRF: Point-based Neural Radiance Fields
Point-NeRF는 Original NeRF의 Training 속도가 매우 느리다는 점과 퀄리티를 개선한 모델로, 연속 Volume Radiance 공간을 모델링 하기 위해서 3D neural point cloud를 사용합니다.
기존 NeRF가 ray를 기반으로 Sampling하여 모델을 학습하는 것과 다르게, Point-NeRF는 3D 공간 상에서 voxel point마다 feature와 confidence를 먼저 구성하고 그 구성을 기반으로 Sampling point와 neural point 간의 거리 정보를 통해 가중치를 구성하여 neural point confidence 값을 사용하여 volume density와 radiance를 계산합니다.

6. NeRF in the wild
기존의 NeRF는 같은 환경에서 다양한 각도로 찍은 사진을 학습에 사용해야 한다는 단점이 있다.
Static Scene만 렌더링 할 수 있는 문제에서 나아가 lightning 을 변화 시킬 수 없고 Scene 수정을 할 수 없다.
NeRF in the Wild 기술은 구글에 있는 전세계 사람들이 다른 환경에서 찍은 것 적용이 가능하다.
NeRF와 비교해서 추출 할 대상과 노이즈 구분을 좀 더 정교하게 하며,
이 과정에서 사진에서 고정되어 있는 건물과 움직이는 피사체(사람이나 자동차)를 구분한다.
따라서 여러 장의 사진을 학습에 사용하여 자유롭게 각도를 이동하고 조명 효과, 하늘 변경을 마음대로 변경할 수 있다.

7. **Generalization** : PixelNeRF (CVPR 2021)
```
문제점 : NeRF는 한개 Scene에 대해 1개의 모델을 학습해야 합니다. 네트워크를 재사용 할 수 없습니다.
```

- Spatial Image Feature를 사용 → Scene들 간의 knowledge를 재사용 가능.
- View Direction과 (CNN으로) Encoding된 feature가 주어졌을 때, NeRF Network로 Color와 density를 출력
    - 샘플링된 points를 view space의 feature grid에 projection.
  - points가 sparse 하기 때문에 모든 correspond point를 구할 수 없으므로 없으므로 bilinear interpolation을 하여 feature vector(W(pi(x))를 추출 → prior knowledge를 share
  - 공간 좌표계는 Canonical Space(Object가 중심이 되는 자표 시스템, NeRF에서 사용)을 사용하지 않고, 입력 View에 대한 Camera Space(촬영 카메라가 중심이 되는 좌표 시스템)을 사용 → 절대적인 좌표계가 존재하지 않기 때문에, generalization 하는데 유리
