# Surgery-assisted Cooperative Robot Project Using Doosan Manipulator M0609

> **Note:** Among the project folders, the `최종` folder contains the final source code.

---

## 1. Project Overview

![스크린샷 2025-06-04 13-36-51](https://github.com/user-attachments/assets/6532c598-7d46-4b40-acac-6a35c17f8e70)

의료 파업 등 의료 공백 문제를 보완하기 위한 수술 보조 협동 로봇 프로젝트입니다. 기존 의료진의 부담을 줄이는 것을 목표로 합니다.

---

### 주요 기능

#### 1) `robot_control` node

**a. 마취 (Anesthesia)**
수술 시작 전 환자의 입에 수면 마취 마스크를 가져다 댑니다.

**b. 의료 기기 객체 인식 (Medical Device Object Recognition)**
STT를 활용해 데이터 라벨 클래스 내 의료 기기 키워드를 인식하고, Realsense 카메라가 해당 키워드의 객체를 인식하면 집어 올립니다.

**c. 손 객체 인식 (Hand Object Recognition)**
그리퍼에 장착된 Realsense 카메라가 의사의 손 객체를 인식하면, 해당 손의 깊이(depth) 위치로 의료 기기를 전달합니다.

**d. 기기 반납**
의사가 사용한 의료 기기를 Realsense 카메라가 인식하고, 데이터 라벨 클래스의 의료 기기로 확인되면 최초 배치 위치로 되돌려 놓습니다.

**e. 봉합 (Seal)**
의료 기기(Hemostat) 인식 후 수술 부위를 인식하여 절개 부위를 봉합합니다.

---

#### 2) `detect_wound` node

**a. 수술 절개 부위 확대 (Expand the Surgical Incision)**
매니퓰레이터가 수술 위치로 이동한 후, 키워드(예: `camera`)를 통해 수술 부위 객체를 인식합니다.

인식이 완료되면 해당 부위에 근접하여 모니터(rqt)로 절개 부위를 확인할 수 있습니다.
이후 **적응형 제어(Adaptive Control)** 를 통해 외력이 감지되면 수술 완료로 판단하고 초기 좌표로 복귀합니다.

**b. 흡입을 이용한 혈액 흡입 (Blood Suction)**
STT(Whisper API)를 사용하여 키워드(예: `suction`, `Yankauer`)를 인식하고, 인식된 수술 부위 주변의 혈액을 흡입합니다.

---

## 2. 사용 장비 및 개발 환경

| 항목 | 내용 |
|------|------|
| 로봇 | Doosan Manipulator M0609 |
| OS / 프레임워크 | Ubuntu 22.04 / ROS2 Humble |
| 그리퍼 | OnRobot2 Gripper |
| 음성 인식 | STT (Whisper API) |
| 카메라 | Realsense Camera |
| 객체 인식 모델 | YOLOv11n |
| 수술 도구 | Scalpel, Mayo_metz, Forcep, Hemostat |

<img width="463" alt="image" src="https://github.com/user-attachments/assets/46f17870-5819-4ee0-8290-3efa13d07690" />

---

## 3. 프로젝트 진행 내용

### 수술 도구 데이터셋 (Surgical Tools Dataset)

<img width="490" alt="image" src="https://github.com/user-attachments/assets/25375e33-d360-478c-bd98-fa6b3529f29c" />

데이터셋 출처: [Roboflow - SGTD](https://universe.roboflow.com/northeastern-university-ftufl/sgtd)

**클래스 레이블**

| 클래스 | 설명 |
|--------|------|
| Mayo_metz | 피부 및 조직 절개용 가위 |
| Forceps | 의료용 핀셋 |
| Scalpel | 수술용 메스 |
| Hemostat | 동맥 클램프 / 지혈 장치 |

**학습 결과**

![image](https://github.com/user-attachments/assets/b49ba1c8-c90a-4586-92c9-239b3509453f)

YOLOv11n 모델을 사용하였으며, `epoch=100`, `img_size=512`, `batch=20(auto)` 설정으로 높은 성능의 객체 인식을 확인하였습니다.

![image](https://github.com/user-attachments/assets/45044496-10bc-495b-9516-b2bae43d4192)

학습이 진행될수록 전체 loss 값이 감소하였으며, Precision 및 Recall 값이 90%를 초과함을 확인하였습니다.

---

### 손 데이터셋 (Hands Dataset)

![image](https://github.com/user-attachments/assets/b0b3b55f-2412-42be-a2d7-b4491cf8a5b2)

데이터셋 출처: [Roboflow - New Hand](https://universe.roboflow.com/hyfyolo/new-hand)

- **사용 클래스:** Hands
- **활용 목적:** 그리퍼가 의료 기기를 집어 들고 이동한 후, 의사의 손을 인식할 때 사용

---

### 수술 상처 데이터셋 (Surgical Wounds Dataset)

![image](https://github.com/user-attachments/assets/59f4c4b4-3c2b-419e-9fbf-9a1801e467da)

데이터셋 출처: [Roboflow - Surgical Wounds](https://universe.roboflow.com/myworkspace-zgags/my-first-project-d3ifu/browse?queryText=&pageSize=50&startingIndex=0&browseQuery=true)

**클래스 레이블**

| 클래스 | 설명 |
|--------|------|
| Stitched | 봉합된 상처 |
| Wound | 절개 부위 |

- **활용 목적:** 절개 부위 인식 및 봉합 기능

---

### 동작 흐름도 (Flowchart)

<img width="494" alt="image" src="https://github.com/user-attachments/assets/d55f09fa-5043-4a30-83af-a6839c123c15" />
<img width="420" alt="image" src="https://github.com/user-attachments/assets/8815abcc-86f5-4730-a8a3-8c9e0ff3b8df" />

**`robot_control` 알고리즘 흐름**

```
"Hello, Rokey" 웨이크워드 감지
→ 환자 수면 마취 진행
→ STT로 의료 기기 키워드 인식
→ 기기 집기
→ 손 객체 인식 후 기기 전달
→ 의료진이 내려놓은 기기 유무 확인
→ 초기 좌표로 복귀
```

**`detect_wound` 노드 흐름**

```
"Hello, Rokey" 웨이크워드 감지
→ STT로 수술 절개 키워드 인식
→ 수술 부위 절개 인식 후 확대
→ 컴플라이언스 제어를 통해 초기 좌표로 복귀
```

---

## 4. 프로젝트 중 경험한 오류

- **a.** 빛 반사로 인해 객체 인식률이 저하됨. 환경에 민감하게 반응.
- **b.** Whisper API 성능 향상을 위해 다양한 프롬프트를 설정하였으나, 일부 경우에서 음성 인식 불가.
- **c.** 데이터 라벨은 총 15개 클래스이나, 상황상 4개 클래스만 사용. (`Mayo_metz`, `Forceps`, `Hemostat`, `Scalpel`)

---

## 5. 데모 영상

[▶ 데모 영상 보기 (Google Drive)](https://drive.google.com/file/d/1ya3sM34Hc5CPN4aMMyc6QHAeQYTwBzs0/view?usp=sharing)
