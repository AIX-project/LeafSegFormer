# LeafSegFormer
#### 3D 포인트 클라우드 이미지를 활용한 트랜스포머 기반의 개별 잎 인스턴스 세그멘테이션 모델인 **LeafSegFormer**를 제안.


## Background 
- 고품질 작물 수요 증가 → 집약적인 노동력(labor-intensive)을 요구하는 원예분야에서 자동화 기술 필요성 대두
- 식물 표현형(Plant Phenotype) 연구:
  - 작물의 생육 상태를 모니터링하고 정확히 측정
  - 관찰 가능한 특성(식물의 모양, 색상, 크기, 구조적 특징 등)을 분석 및 측정
  - 개별 잎의 개수 및 크기 추정 작업 → 작물의 수확량, 건강상태, 영양소 또는 농약의 필요성을 판단하는데 중요한 정보 제공


## PLANesT-3D Dataset 
##### Mertoğlu, Kerem, et al., 2024
- Annotated dataset for semantic/instance segmentation of 3D plant point clouds
- 34개의 point cloud model로 구성: model별 평균 2백만 개 points + 53개의 leaflets 포함
- 3가지 식물 종을 포함: Capsicum annuum(10개체), Rosa kordana(10개체), Ribes rubrum(14개체)
- XYZ coordinate & RGB 색상 데이터 + Confidence map + Semantic Segmentation Label + Instance Segmentation Label
- Data Augmentation: rotation, translation, scaling, jittering

<img width="1896" alt="image" src="https://github.com/user-attachments/assets/ec5165d6-e573-4247-946e-87a55d2f5373" />

- ##### Figure 1. Instance segmentation of individual leaves of Capsicum annuum visualized in a 3D point cloud.  

<img width="417" alt="image" src="https://github.com/user-attachments/assets/3e98f2f3-292d-48b9-94ef-75a37f2140d9" />

- ##### Figure 2. PLANesT-3D dataset Instance Label rendered with point color 


## Approach for 3D PointCloud Segmentation  
- Transformer-Based Approach
  - Mask Attention 메커니즘을 기반으로 고정된 개수의 Object Query에서 Instance 예측을 학습
  - 최근 연구(et al.)에서는 이러한 Transformer 기반 모델의 강력한 아키텍처적 이점을 바탕으로 높은 성능을 보여주고 있어, 새로운 SOTA를 달성

## Architecture
<img width="1896" alt="image" src="https://github.com/user-attachments/assets/107bb941-d2d8-4b3d-9ab3-946386f13929" />

- ##### Figure 2. The overall architecture of LeafSegFormer.

#### 1. PointNet++
##### Qi, Charles Ruizhongtai, et al.,2017
- 순서가 없는(Undordered) Pointcloud 데이터를 Voxelization 없이 그대로 입력받아 딥러닝에 활용하는 네트워크
- Nested Partitioning 기법을 사용해 Point-Wise Feature Extraction을 수행. 구체적으로는 Local Feature를 추출



#### 2. Transformer
##### Sun, Jiahao, et al.,2023

- Attention Mechanism으로 잎 간의 Global Context를 직접적으로 학습하여 잎의 복잡한 경계와 잎의 겹침 문제를 해결
- Encoder :  Global Context를 학습하는 단계. PointNet++이 추출한 전체 특징을 Transformer의 Self-Attention 메커니즘을 통해 전역적(Global) 관계를 학습
- Decoder : Object Query를 기반으로 인스턴스 마스크를 추론하는 단계
    - Object Query : Transformer의 디코더 입력으로 사용되는 고정된 개수의 학습 가능한 Vector
    - Cross-Attention : Object Query와 Global Features 간의 관계를 학습하여, 각 query는 잠재적으로 하나의 객체를 나타내도록 업데이트
    - Self-Attention : Cross-Attention에서 업데이트 된 Query 간의 상호작용 학습
    - Feedforward Network : Query 벡터를 비선형 변환하여 객체의 class, score, mask 정보를 생성
#### 3. Hungarian Matcher
- 모델의 예측결과 Query와 Ground Truth간의 1:1 최적 매칭을 수행
- Cost Matrix : Class 예측인 Corss Entropy loss와 Mask 예측의 BCE 기반 비용을 합산 → 이 비용을 최소화 하는 최적의 매칭
- Unordered Dataset인 PointCloud에 대해 순서에 관계 없이 동일한 결과를 보장하는 방법

## Result Accuracy 
- Baseline (PointNet++ Only) : 57.4%
- LeafSegFormer (Our Purposed Model) : 76.9% 

## Contribution
- 식물 개별 잎 Instance Segmentation Task의 정량적 지표를 제공
- PointNet++와 Transformer 구조를 결합한 End-to-End Instance Segmentation 모델을 설계하여 Baseline 대비, 약 20% 향상된 결과를 도출
- 복잡한 중간 단계 없이 직접적으로 Instance Mask를 예측할 수 있어, 모델의 학습과 추론 과정을 단순화하고 성능을 향상
