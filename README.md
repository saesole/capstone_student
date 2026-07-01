# ND1 캡스톤 디자인 - ROS2 기반 물체 이동 시뮬레이션

## 1. 프로젝트 개요

본 프로젝트는 ROS2 기반 로봇 시스템에서 자연어 명령을 입력받아 물체 이동 미션을 수행하는 캡스톤 과제이다. 사용자는 "A에서 B로 이동"과 같은 자연어 명령을 입력하고, 시스템은 이를 구조화된 미션으로 변환하여 이동, 파지, 운반, 배치, 도킹 흐름을 순차적으로 수행한다.

## 2. 주요 기능

- 자연어 명령 입력 처리
- A/B/C 구역 좌표 기반 미션 생성
- ROS2 pub/sub 기반 노드 간 통신
- Coordinator FSM을 통한 단계별 상태 전이
- 시뮬레이션 모드에서 이동, 파지, 배치, 도킹 흐름 검증
- /robot_status 토픽을 통한 진행 상태 모니터링

## 3. 구현 내용

### Node A - 자연어 명령 파서

node_a_llm.py에서 키워드 기반 폴백 파서를 구현하였다. Groq API가 없는 환경에서도 한국어 명령을 해석할 수 있도록 하였으며, 구역 정보와 이동 키워드를 기준으로 pick_and_place, navigate, stop 명령을 생성한다.

입력 예시:
- A에서 B로 이동

출력 미션:
- pick_and_place pick=(1.5,0.5) place=(2.5,-1.0)

### Coordinator FSM

coordinator_fsm.py에서 미션 종류에 따른 상태 전이 로직을 구현하였다. pick_and_place 미션 입력 시 다음 순서로 실행된다.

- UNDOCKING
- NAVIGATING
- GRASPING
- TRANSPORTING
- PLACING
- DOCKING
- DONE

## 4. 실행 환경

- Ubuntu 22.04 WSL
- Docker
- Docker Compose
- ROS2 Humble
- Python 3
- noVNC Web Desktop

## 5. 실행 방법

### 1. Docker 실행

- cd ~/capstone_student
- sudo service docker start
- docker-compose up -d

### 2. 컨테이너 접속

- docker exec -it nd1_capstone_dev bash

### 3. ROS2 워크스페이스 빌드

- cd /home/ubuntu/ros2_ws
- source /opt/ros/humble/setup.bash
- colcon build
- source install/setup.bash

### 4. 시뮬레이션 실행

- ros2 launch nd1_capstone bringup.launch.py sim_mode:=true

### 5. 다른 터미널에서 명령 입력

- docker exec -it nd1_capstone_dev bash
- cd /home/ubuntu/ros2_ws
- source /opt/ros/humble/setup.bash
- source install/setup.bash
- ros2 topic pub --once /llm_command std_msgs/String "data: 'A에서 B로 이동'"

### 6. 상태 확인

- ros2 topic echo /robot_status

## 6. 검증 결과

시뮬레이션 모드에서 다음 흐름이 정상 출력되는 것을 확인하였다.

- [FSM:PLANNING] 미션 접수: pick_and_place
- [FSM:UNDOCKING] undock 요청
- [FSM:NAVIGATING] 이동 요청 -> (1.50, 0.50)
- [FSM:GRASPING] grasp 요청 -> (1.50, 0.50)
- [FSM:TRANSPORTING] 이동 요청 -> (2.50, -1.00)
- [FSM:PLACING] place 요청 -> (2.50, -1.00)
- [FSM:DOCKING] dock 요청
- [FSM:DONE] Mission done

## 7. GitHub 저장소

https://github.com/saesole/capstone_student