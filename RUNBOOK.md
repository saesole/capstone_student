# ND1 캡스톤 실행 절차서

## 1. 목적

이 문서는 ND1 캡스톤 프로젝트를 재현하기 위한 실행 절차를 정리한 문서이다. Docker 기반 ROS2 환경을 실행하고, 시뮬레이션 모드에서 자연어 명령 기반 물체 이동 미션이 정상 동작하는지 확인하는 방법을 포함한다.

## 2. 사전 준비

- Windows WSL Ubuntu 22.04
- Docker 설치
- Docker Compose 설치
- GitHub 저장소 클론 완료
- 프로젝트 경로: ~/capstone_student

## 3. Docker 컨테이너 실행

터미널에서 다음 순서로 실행한다.

1. 프로젝트 폴더로 이동

cd ~/capstone_student

2. Docker 서비스 시작

sudo service docker start

3. 컨테이너 실행

docker-compose up -d

4. 컨테이너 상태 확인

docker ps

정상 실행 시 nd1_capstone_dev 컨테이너가 Up 상태로 표시된다.

## 4. noVNC 접속

브라우저에서 아래 주소로 접속한다.

http://localhost:8080

noVNC 화면이 나타나면 연결 버튼을 눌러 컨테이너의 GUI 환경을 확인할 수 있다.

## 5. ROS2 컨테이너 접속

터미널에서 다음 명령을 실행한다.

docker exec -it nd1_capstone_dev bash

## 6. 워크스페이스 빌드

컨테이너 안에서 다음 순서로 실행한다.

cd /home/ubuntu/ros2_ws
source /opt/ros/humble/setup.bash
colcon build
source install/setup.bash

정상 빌드 시 다음과 유사한 메시지가 출력된다.

Summary: 1 package finished

## 7. 시뮬레이션 실행

첫 번째 컨테이너 터미널에서 다음 명령을 실행한다.

ros2 launch nd1_capstone bringup.launch.py sim_mode:=true

정상 실행 시 node_a_llm, node_b_nav, node_c_grasp, coordinator_fsm 노드가 시작된다.

## 8. 명령 입력

두 번째 터미널을 열고 WSL에서 컨테이너에 접속한다.

docker exec -it nd1_capstone_dev bash

컨테이너 안에서 다음 명령을 실행한다.

cd /home/ubuntu/ros2_ws
source /opt/ros/humble/setup.bash
source install/setup.bash
ros2 topic pub --once /llm_command std_msgs/String "data: 'A에서 B로 이동'"

## 9. 상태 확인

세 번째 터미널 또는 같은 컨테이너 터미널에서 다음 명령을 실행한다.

ros2 topic echo /robot_status

정상 동작 시 다음과 같은 상태 흐름이 출력된다.

- 미션 접수: pick_and_place
- undock 요청
- 이동 요청 -> A 좌표
- grasp 요청
- 이동 요청 -> B 좌표
- place 요청
- dock 요청
- Mission done

## 10. 정상 동작 기준

다음 조건을 만족하면 시뮬레이션 검증 성공으로 판단한다.

- 자연어 명령이 pick_and_place 미션으로 변환된다.
- A 좌표 pick=(1.5,0.5)가 사용된다.
- B 좌표 place=(2.5,-1.0)가 사용된다.
- FSM 상태가 PLANNING부터 DONE까지 순차적으로 진행된다.
- /robot_status에서 Mission done 메시지가 확인된다.

## 11. 문제 해결

### Docker 명령이 안 될 때

sudo service docker start

명령으로 Docker 서비스를 다시 시작한다.

### ros2 명령을 찾을 수 없을 때

source /opt/ros/humble/setup.bash

명령을 다시 실행한다.

### 패키지를 찾을 수 없을 때

cd /home/ubuntu/ros2_ws
colcon build
source install/setup.bash

순서로 다시 빌드하고 환경을 적용한다.

### 명령 입력 시 Waiting for at least 1 matching subscription(s)가 반복될 때

bringup.launch.py가 실행 중인지 확인한다. Node A가 실행 중이어야 /llm_command 구독자가 존재한다.