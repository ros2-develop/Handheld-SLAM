# 🚀 라즈베리파이 5 + D500 라이다 SLAM 시스템

헤드리스 Docker 환경에서 실행되는 완전한 SLAM 시스템입니다. 이 문서에는
프로젝트의 모든 파일이 포함되어 있습니다. 각 파일의 내용은 제목을 클릭하여 펼치거나
접을 수 있습니다.

## ⚡ 빠른 시작

### 1. 시스템 준비

### # 프로젝트 클론

git clone <repository-url>
cd rpi_slam_system

### # 라즈베리파이 최적화 적용

sudo ./scripts/setup_rpi.sh

### 2. 빌드 & 배포

### # 도커 이미지 빌드 (15-30분 소요)

./scripts/build.sh

### # SLAM 시스템 배포

./scripts/deploy.sh

### 3. 모니터링

### # 실시간 모니터링

./scripts/monitor.sh continuous

### # 상태 확인

./scripts/monitor.sh

## 🔧 지원하는 SLAM 알고리즘

```
● 1 단계 : Hector SLAM - 빠르고 안정적 (권장 시작점)
● 2 단계 : GMapping - 더 정확한 맵핑
● 3 단계 : Cartographer - 최고 품질 (높은 CPU 사용량)
```
## 🖥 PC 에서 시각화


### 개발 PC에서 다음 명령어로 시각화:

### # ROS2 도메인 설정

export ROS_DOMAIN_ID=

### # 토픽 확인

ros2 topic list

# RViz2 실행
rviz2 -d pc_visualization/rviz_configs/hector_slam.rviz

## 📂 전체 프로젝트 파일

<br>

<details>
<summary><code>docker/Dockerfile</code></summary>
# ========================================
# Stage 1: Base ROS2 Environment
# ========================================
FROM ros:humble-ros-base-jammy AS ros2-base

### # 필수 시스템 패키지 설치

RUN apt-get update && apt-get install -y \
python3-pip \
python3-colcon-common-extensions \
python3-rosdep \
git \
wget \
curl \
htop \
tmux \
nano \
usbutils \
&& rm -rf /var/lib/apt/lists/*

### # ARM64 최적화 컴파일 플래그

ENV MAKEFLAGS="-j4"
ENV COLCON_PARALLEL_EXECUTOR_ARGUMENTS="--parallel-workers 4"


### # ========================================

# Stage 2: SLAM Libraries Builder
# ========================================
FROM ros2-base AS slam-builder

WORKDIR /tmp/slam_build

# Hector SLAM 설치 (1단계)
RUN apt-get update && apt-get install -y \
ros-humble-hector-slam \
ros-humble-hector-mapping \
ros-humble-hector-trajectory-server \
&& rm -rf /var/lib/apt/lists/*

# GMapping을 위한 ROS1 Bridge 설치 (2단계)
RUN apt-get update && apt-get install -y \
ros-humble-ros1-bridge \
ros-humble-gmapping \
&& rm -rf /var/lib/apt/lists/*

# Cartographer 설치 (3단계)
RUN apt-get update && apt-get install -y \
ros-humble-cartographer \
ros-humble-cartographer-ros \
&& rm -rf /var/lib/apt/lists/*

### # ========================================

# Stage 3: D500 LiDAR Integration
# ========================================
FROM slam-builder AS lidar-integration

WORKDIR /tmp/lidar_build

### # D500 공식 드라이버 빌드

RUN mkdir -p ros2_ws/src && cd ros2_ws/src && \
git clone https://github.com/ldrobotSensorTeam/ldlidar_stl_ros2.git && \
cd .. && \

. /opt/ros/humble/setup.sh && \


colcon build --packages-select ldlidar_stl_ros2 && \

. install/setup.sh

### # 대안 드라이버도 빌드 (더 안정적)

RUN cd ros2_ws/src && \
git clone https://github.com/Myzhar/ldrobot-lidar-ros2.git && \
cd .. && \

. /opt/ros/humble/setup.sh && \
colcon build --packages-select ldlidar_node && \
. install/setup.sh

### # ========================================

# Stage 4: Runtime Environment (최종 경량화)
# ========================================
FROM ros2-base AS slam-runtime

### # 빌드된 패키지들만 복사

COPY --from=lidar-integration /tmp/lidar_build/ros2_ws/install /opt/ldlidar_ws/
COPY --from=slam-builder /opt/ros/humble /opt/ros/humble

### # 설정 파일들 복사

COPY config/ /opt/slam_config/
COPY launch/ /opt/slam_launch/
COPY scripts/monitor.sh /opt/scripts/
COPY docker/entrypoint.sh /entrypoint.sh

### # 권한 설정

RUN chmod +x /entrypoint.sh /opt/scripts/monitor.sh

### # 작업 디렉토리 설정

WORKDIR /opt/slam_ws

### # 환경 변수 설정

### ENV ROS_DOMAIN_ID=

ENV RMW_IMPLEMENTATION=rmw_cyclonedx_cpp
ENV CYCLONEDX_URI=file:///opt/slam_config/dds/cyclonedx_config.xml

### # USB 장치 접근을 위한 사용자 그룹

RUN usermod -a -G dialout root


ENTRYPOINT ["/entrypoint.sh"]
CMD ["bash"]

</details>

<details>
<summary><code>docker/docker-compose.yml</code></summary>
version: '3.8'

services:
slam-robot:
build:
context: ..
dockerfile: docker/Dockerfile
target: slam-runtime
container_name: rpi_slam_system

### # 리소스 제한 (라즈베리파이 5 최적화)

deploy:
resources:
limits:
memory: 6G
cpus: '3.5'
reservations:
memory: 2G
cpus: '1.0'

### # 네트워크 설정

network_mode: host

### # 볼륨 마운트

volumes:

- ../config:/opt/slam_config:ro
- ../launch:/opt/slam_launch:ro
- slam_maps:/opt/slam_ws/maps
- slam_logs:/opt/slam_ws/logs

### # 장치 접근


devices:

- /dev/ttyUSB0:/dev/ttyUSB0 # D500 라이다
- /dev/ttyACM0:/dev/ttyACM0 # 대안 포트

privileged: true # USB 접근 권한

### # 환경 변수

environment:

- ROS_DOMAIN_ID=
- RMW_IMPLEMENTATION=rmw_cyclonedx_cpp
- SLAM_MODE=hector # hector/gmapping/cartographer
- LIDAR_PORT=/dev/ttyUSB
- MAP_RESOLUTION=0.
- DEBUG_MODE=false

### # 헬스체크

healthcheck:
test: ["CMD", "ros2", "topic", "list"]
interval: 30s
timeout: 10s
retries: 3
start_period: 60s

### # 재시작 정책

restart: unless-stopped

### # 로그 설정

logging:
driver: "json-file"
options:
max-size: "100m"
max-file: "3"

volumes:
slam_maps:
driver: local
slam_logs:
driver: local


</details>

<details>
<summary><code>docker/entrypoint.sh</code></summary>
#!/bin/bash

### # ===========================================

### # 라즈베리파이 SLAM 시스템 엔트리포인트

### # ===========================================

set -e

echo "🚀 Starting RaspberryPi SLAM System..."

### # ROS2 환경 설정

source /opt/ros/humble/setup.bash
source /opt/ldlidar_ws/setup.bash

### # DDS 설정

export CYCLONEDX_URI=file:///opt/slam_config/dds/cyclonedx_config.xml

### # 라이다 포트 권한 설정

if [ -e "$LIDAR_PORT" ]; then
chmod 666 $LIDAR_PORT
echo "✅ LiDAR port $LIDAR_PORT configured"
else
echo "⚠ LiDAR port $LIDAR_PORT not found, searching alternatives..."
for port in /dev/ttyUSB* /dev/ttyACM*; do
if [ -e "$port" ]; then
export LIDAR_PORT=$port
chmod 666 $port
echo "✅ Found alternative port: $port"
break
fi
done
fi

### # 모니터링 스크립트 백그라운드 실행


if [ "$DEBUG_MODE" = "true" ]; then
/opt/scripts/monitor.sh &
MONITOR_PID=$!
echo "🔍 Monitoring started (PID: $MONITOR_PID)"
fi

### # SLAM 모드에 따른 실행

case "$SLAM_MODE" in
"hector")
echo "🗺 Starting Hector SLAM (Stage 1)"
ros2 launch /opt/slam_launch/stage1_hector.launch.py
;;
"gmapping")
echo "🗺 Starting GMapping SLAM (Stage 2)"
ros2 launch /opt/slam_launch/stage2_gmapping.launch.py
;;
"cartographer")
echo "🗺 Starting Cartographer SLAM (Stage 3)"
ros2 launch /opt/slam_launch/stage3_cartographer.launch.py
;;
*)
echo "ℹ No SLAM mode specified, starting interactive shell"
exec "$@"
;;
esac

</details>

<details>
<summary><code>config/dds/cyclonedx_config.xml</code></summary>
<?xml version="1.0" encoding="UTF-8" ?>
<!--
D500 라이다 + 라즈베리파이 5 최적화 DDS 설정
네트워크 성능 및 실시간성 최적화
-->
<CycloneDX>
<Domain id="42">
<Discovery>
<ParticipantIndex>auto</ParticipantIndex>


<Peers>
<!-- 개발 PC IP 주소 (실제 환경에 맞게 수정) -->
<Peer address="192.168.1.50"/>
<Peer address="192.168.1.51"/>
</Peers>

<MulticastRecvNetworkInterfaceAddress>wlan0</MulticastRecvNetworkInterfaceAddr
ess>
</Discovery>

### <!-- 네트워크 성능 최적화 -->

<Internal>
<MaxMessageSize>65536</MaxMessageSize>
<FragmentSize>1400</FragmentSize> <!-- WiFi MTU 고려 -->
<DeliveryQueueMaxSamples>100</DeliveryQueueMaxSamples>
</Internal>

### <!-- 실시간성 최적화 -->

<Scheduling>
<WorkerThreads>2</WorkerThreads>
<PriorityClass>RealTime</PriorityClass>
</Scheduling>
</Domain>
</CycloneDX>

</details>

<details>
<summary><code>config/lidar/d500_config.yaml</code></summary>
# ==========================================
# LDROBOT D500 (STL-19P) 설정 파일
# 검색 결과 기반 정확한 스펙 반영
# ==========================================

ldlidar_node:
ros__parameters:
# 기본 설정
product_name: "LDLiDAR_STL19P" # D500 = STL-19P
topic_name: "scan"


frame_id: "base_laser"

### # 포트 설정

port_name: "/dev/ttyUSB0"
port_baudrate: 115200 # D500 표준 보드레이트

### # D500 특화 설정

laser_scan_dir: false # 시계방향 -> ROS 표준으로 변환
enable_angle_crop_func: false
angle_crop_min: 0.
angle_crop_max: 360.

# 스캔 품질 설정 (D500 5kHz 측정 주파수 활용)
range_min: 0.12 # D500 최소 범위
range_max: 12.0 # D500 최대 범위
scan_frequency: 10.0 # D500 표준 스캔 주파수

# 노이즈 필터링 (±30mm 정확도 활용)
outlier_filter: true
outlier_threshold: 0.05 # 5cm 임계값
median_filter: true
median_kernel_size: 3

# QoS 설정
qos_overrides:
scan:
reliability: best_effort # 실시간성 우선
durability: volatile
depth: 10

</details>

<details>
<summary><code>config/slam/hector_slam.yaml</code></summary>
# ==========================================
# Hector SLAM 설정 (1단계)
# D500 라이다 특성에 최적화
# ==========================================


hector_mapping:
ros__parameters:
# 맵 설정 (D500 12m 범위 고려)
map_resolution: 0.025 # 2.5cm (D500 ±30mm 정확도 활용)
map_size: 2048 # 51.2m x 51.2m
map_start_x: 0.
map_start_y: 0.
map_multi_res_levels: 3 # 3 단계 해상도

### # 좌표계 설정

map_frame: "map"
base_frame: "base_link"
odom_frame: "odom"

### # 스캔 매칭 설정 (D500 고밀도 데이터 활용)

use_tf_scan_transformation: true
use_tf_pose_start_estimate: false
pub_map_odom_transform: true
pub_odometry: true
pub_map_scanmatch_transform: true

### # 매칭 알고리즘 파라미터

laser_min_dist: 0.12 # D500 최소 범위
laser_max_dist: 12.0 # D500 최대 범위
laser_z_min_value: -1.
laser_z_max_value: 1.

# 스캔 매칭 정밀도 (D500 5kHz 주파수 활용)
map_update_distance_thresh: 0.1 # 10cm마다 업데이트
map_update_angle_thresh: 0.05 # 2.8도마다 업데이트

### # 최적화 설정

update_factor_free: 0.
update_factor_occupied: 0.
map_pub_period: 2.0 # 2 초마다 맵 발행

# QoS 설정
qos_overrides:
map:


reliability: reliable
durability: transient_local
depth: 1
scan:
reliability: best_effort
durability: volatile
depth: 10

</details>

<details>
<summary><code>launch/stage1_hector.launch.py</code></summary>
#!/usr/bin/env python
"""
1 단계: Hector SLAM 런처
D500 라이다 + 라즈베리파이 5 최적화
"""

import os
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, OpaqueFunction
from launch.substitutions import LaunchConfiguration, PathJoinSubstitution
from launch_ros.actions import Node
from launch_ros.substitutions import FindPackageShare

def generate_launch_description():

### # 런처 인수

lidar_port_arg = DeclareLaunchArgument(
'lidar_port',
default_value='/dev/ttyUSB0',
description='D500 LiDAR USB port'
)

map_resolution_arg = DeclareLaunchArgument(
'map_resolution',
default_value='0.025',
description='Map resolution in meters/pixel'
)


debug_mode_arg = DeclareLaunchArgument(
'debug',
default_value='false',
description='Enable debug output'
)

def launch_setup(context, *args, **kwargs):
# 설정 파일 경로
d500_config = '/opt/slam_config/lidar/d500_config.yaml'
hector_config = '/opt/slam_config/slam/hector_slam.yaml'

nodes = []

### # D500 라이다 드라이버 노드

lidar_node = Node(
package='ldlidar_stl_ros2',
executable='ldlidar_stl_ros2_node',
name='d500_lidar',
output='screen',
parameters=[d500_config, {
'port_name': LaunchConfiguration('lidar_port'),
'laser_scan_dir': False, # 시계방향 -> 반시계방향 변환
}],
remappings=[
('scan', '/scan'),
]
)
nodes.append(lidar_node)

# Hector Mapping 노드
hector_mapping_node = Node(
package='hector_mapping',
executable='hector_mapping',
name='hector_mapping',
output='screen',
parameters=[hector_config, {
'map_resolution': LaunchConfiguration('map_resolution'),
}],


remappings=[
('scan', '/scan'),
('map', '/map'),
('poseupdate', '/poseupdate'),
]
)
nodes.append(hector_mapping_node)

# TF 변환 설정 (base_link -> base_laser)
tf_node = Node(
package='tf2_ros',
executable='static_transform_publisher',
name='base_to_laser_tf',
arguments=['0', '0', '0.1', '0', '0', '0', 'base_link', 'base_laser'],
output='screen'
)
nodes.append(tf_node)

### # 맵 서버 (맵 저장/로드용)

map_server_node = Node(
package='nav2_map_server',
executable='map_saver_server',
name='map_saver',
output='screen',
parameters=[{
'save_map_timeout': 5.0,
'free_thresh_default': 0.25,
'occupied_thresh_default': 0.65,
}]
)
nodes.append(map_server_node)

### # 디버그 모드시 추가 노드들

if LaunchConfiguration('debug').perform(context).lower() == 'true':
# 스캔 품질 모니터링
scan_monitor = Node(
package='diagnostic_updater',
executable='diagnostic_aggregator',
name='lidar_diagnostics',


output='screen'
)
nodes.append(scan_monitor)

return nodes

return LaunchDescription([
lidar_port_arg,
map_resolution_arg,
debug_mode_arg,
OpaqueFunction(function=launch_setup)
])

</details>

<details>
<summary><code>scripts/build.sh</code></summary>
#!/bin/bash

### # ===========================================

### # 라즈베리파이 SLAM 시스템 빌드 스크립트

### # ARM64 최적화 빌드

### # ===========================================

set -e

### # 색상 정의

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo -e "${GREEN}🔨 Building RaspberryPi SLAM Docker System${NC}"

### # 프로젝트 루트 확인

if [! -f "docker/Dockerfile" ]; then
echo -e "${RED}❌ Please run from project root directory${NC}"
exit 1
fi


### # 시스템 정보 확인

echo -e "\n${YELLOW}📋 System Information:${NC}"
echo "Architecture: $(uname -m)"
echo "Kernel: $(uname -r)"
echo "Available Memory: $(free -h | grep '^Mem:' | awk '{print $2}')"
echo "CPU Cores: $(nproc)"

# Docker 버전 확인
if! command -v docker &> /dev/null; then
echo -e "${RED}❌ Docker not installed${NC}"
exit 1
fi

echo "Docker Version: $(docker --version)"

### # 빌드 옵션 설정

### BUILD_ARGS=""

if [ "$(uname -m)" = "aarch64" ]; then
echo -e "${GREEN}✅ ARM64 architecture detected${NC}"
BUILD_ARGS="--platform linux/arm64"
fi

### # 이전 이미지 정리 (선택적)

read -p "🗑 Clean previous images? (y/N): " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
echo -e "${YELLOW}🧹 Cleaning previous images...${NC}"
docker image prune -f
docker system prune -f
fi

### # 빌드 시작

echo -e "\n${GREEN}🚀 Starting build process...${NC}"
echo "This may take 15-30 minutes on RaspberryPi 5..."

### # 멀티 스테이지 빌드 진행률 표시

docker build $BUILD_ARGS \
--tag rpi-slam:latest \


--file docker/Dockerfile \
--progress=plain \

. || {
echo -e "${RED}❌ Build failed${NC}"
exit 1
}

### # 빌드 완료 확인

echo -e "\n${GREEN}✅ Build completed successfully!${NC}"

### # 이미지 정보 표시

echo -e "\n${YELLOW}📊 Image Information:${NC}"
docker images rpi-slam:latest --format "table
{{.Repository}}\t{{.Tag}}\t{{.Size}}\t{{.CreatedAt}}"

### # 컨테이너 실행 가이드

echo -e "\n${GREEN}🎯 Next Steps:${NC}"
echo "1. Run: ./scripts/deploy.sh"
echo "2. Or manual: docker-compose up -d"
echo "3. Monitor: ./scripts/monitor.sh"

echo -e "\n${GREEN}🎉 Ready to deploy!${NC}"

</details>

<details>
<summary><code>scripts/deploy.sh</code></summary>
#!/bin/bash

### # ===========================================

### # 라즈베리파이 SLAM 시스템 배포 스크립트

### # ===========================================

set -e

### # 색상 정의

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'


BLUE='\033[0;34m'
NC='\033[0m'

echo -e "${GREEN}🚀 Deploying RaspberryPi SLAM System${NC}"

### # 프로젝트 루트 확인

if [! -f "docker-compose.yml" ]; then
cd docker
fi

### # SLAM 모드 선택

echo -e "\n${BLUE}📋 Select SLAM Algorithm:${NC}"
echo "1) Hector SLAM (Recommended for first test)"
echo "2) GMapping SLAM (Better accuracy, needs more CPU)"
echo "3) Cartographer SLAM (Best quality, highest CPU usage)"
echo "4) Custom configuration"

read -p "Choose option (1-4): " slam_choice

case $slam_choice in
1)
SLAM_MODE="hector"
echo -e "${GREEN}✅ Selected: Hector SLAM${NC}"
;;
2)
SLAM_MODE="gmapping"
echo -e "${GREEN}✅ Selected: GMapping SLAM${NC}"
;;
3)
SLAM_MODE="cartographer"
echo -e "${GREEN}✅ Selected: Cartographer SLAM${NC}"
;;
4)
read -p "Enter custom SLAM mode: " SLAM_MODE
echo -e "${GREEN}✅ Selected: Custom ($SLAM_MODE)${NC}"
;;
*)
SLAM_MODE="hector"
echo -e "${YELLOW}⚠ Invalid choice, defaulting to Hector SLAM${NC}"


### ;;

esac

### # 라이다 포트 감지

echo -e "\n${BLUE}🔍 Detecting LiDAR port...${NC}"
LIDAR_PORT=""
for port in /dev/ttyUSB0 /dev/ttyUSB1 /dev/ttyACM0 /dev/ttyACM1; do
if [ -e "$port" ]; then
LIDAR_PORT=$port
echo -e "${GREEN}✅ Found LiDAR at: $port${NC}"
break
fi
done

if [ -z "$LIDAR_PORT" ]; then
echo -e "${YELLOW}⚠ No LiDAR port auto-detected${NC}"
read -p "Enter LiDAR port manually: " LIDAR_PORT
fi

### # 디버그 모드 옵션

read -p "🔍 Enable debug mode? (y/N): " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
DEBUG_MODE="true"
echo -e "${YELLOW}🐛 Debug mode enabled${NC}"
else
DEBUG_MODE="false"
fi

### # 환경 변수 설정

export SLAM_MODE
export LIDAR_PORT
export DEBUG_MODE

### # 기존 컨테이너 정리

if [ "$(docker ps -q -f name=rpi_slam_system)" ]; then
echo -e "\n${YELLOW}🛑 Stopping existing container...${NC}"
docker-compose down
fi


### # 시스템 최적화 적용

echo -e "\n${BLUE}⚙ Applying system optimizations...${NC}"
sudo sysctl -w net.core.rmem_max=134217728 2>/dev/null || true
sudo sysctl -w net.core.wmem_max=134217728 2>/dev/null || true

### # 컨테이너 시작

echo -e "\n${GREEN}🚀 Starting SLAM container...${NC}"
docker-compose up -d

### # 시작 확인

echo -e "\n${BLUE}⏳ Waiting for system startup...${NC}"
sleep 10

### # 헬스체크

if docker ps | grep -q "rpi_slam_system.*healthy\|rpi_slam_system.*Up"; then
echo -e "${GREEN}✅ SLAM system started successfully!${NC}"

### # 시스템 정보 표시

echo -e "\n${BLUE}📊 System Status:${NC}"
docker-compose ps

echo -e "\n${BLUE}📡 ROS2 Topics:${NC}"
docker exec rpi_slam_system ros2 topic list 2>/dev/null || echo "Topics will be
available shortly..."

echo -e "\n${GREEN}🎯 System is running with:${NC}"
echo " • SLAM Mode: $SLAM_MODE"
echo " • LiDAR Port: $LIDAR_PORT"
echo " • Debug Mode: $DEBUG_MODE"
echo " • ROS Domain: 42"

echo -e "\n${BLUE}📱 Monitoring Commands:${NC}"
echo " • View logs: docker-compose logs -f"
echo " • Monitor: ./scripts/monitor.sh"
echo " • Shell access: docker exec -it rpi_slam_system bash"

echo -e "\n${GREEN}🖥 PC Visualization Setup:${NC}"
echo " On your development PC, run:"


echo " export ROS_DOMAIN_ID=42"
echo " ros2 topic list # Should see /scan, /map topics"
echo " rviz2 -d pc_visualization/rviz_configs/${SLAM_MODE}_slam.rviz"

else
echo -e "${RED}❌ Failed to start SLAM system${NC}"
echo -e "\n${YELLOW}🔍 Checking logs...${NC}"
docker-compose logs --tail=20
exit 1
fi

echo -e "\n${GREEN}🎉 Deployment completed successfully!${NC}"

</details>

<details>
<summary><code>scripts/monitor.sh</code></summary>
#!/bin/bash

### # ===========================================

### # 라즈베리파이 SLAM 시스템 모니터링 스크립트

### # ===========================================

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

### # 모니터링 함수들

show_system_info() {
echo -e "\n${BLUE}📊 System Information${NC}"
echo "============================================"
echo "Hostname: $(hostname)"
echo "Uptime: $(uptime -p)"
echo "Load Average: $(cat /proc/loadavg | cut -d' ' -f1-3)"
echo "Memory Usage: $(free -h | grep '^Mem:' | awk '{print $3 "/" $2}')"
echo "CPU Temperature: $(vcgencmd measure_temp 2>/dev/null || echo 'N/A')"
echo "Disk Usage: $(df -h / | tail -1 | awk '{print $3 "/" $2 " (" $5 ")"}')"


### }

show_docker_status() {
echo -e "\n${BLUE}🐳 Docker Status${NC}"
echo "============================================"
if docker ps | grep -q "rpi_slam_system"; then
echo -e "${GREEN}✅ SLAM Container: Running${NC}"
echo "Container Stats:"
docker stats --no-stream --format "table
{{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}" rpi_slam_system
else
echo -e "${RED}❌ SLAM Container: Not Running${NC}"
fi
}

show_ros_status() {
echo -e "\n${BLUE}🤖 ROS2 Status${NC}"
echo "============================================"

if docker exec rpi_slam_system ros2 node list >/dev/null 2>&1; then
echo -e "${GREEN}✅ ROS2 System: Active${NC}"

echo -e "\n${YELLOW}Active Nodes:${NC}"
docker exec rpi_slam_system ros2 node list 2>/dev/null || echo "No nodes
detected"

echo -e "\n${YELLOW}Active Topics:${NC}"
docker exec rpi_slam_system ros2 topic list 2>/dev/null || echo "No topics
detected"

echo -e "\n${YELLOW}Topic Frequencies:${NC}"
timeout 5 docker exec rpi_slam_system ros2 topic hz /scan 2>/dev/null || echo
"/scan: No data"

else
echo -e "${RED}❌ ROS2 System: Not Active${NC}"
fi
}


show_lidar_status() {
echo -e "\n${BLUE}📡 LiDAR Status${NC}"
echo "============================================"

### # USB 장치 확인

if lsusb | grep -i "cp210\|ch340\|ftdi" >/dev/null; then
echo -e "${GREEN}✅ USB-Serial Adapter: Detected${NC}"
else
echo -e "${YELLOW}⚠ USB-Serial Adapter: Not clearly detected${NC}"
fi

### # 시리얼 포트 확인

for port in /dev/ttyUSB* /dev/ttyACM*; do
if [ -e "$port" ]; then
echo -e "${GREEN}✅ Serial Port: $port${NC}"
ls -la "$port"
fi
done

### # 스캔 데이터 확인

echo -e "\n${YELLOW}Scan Data Sample:${NC}"
timeout 3 docker exec rpi_slam_system ros2 topic echo /scan --once 2>/dev/null |
head -10 || echo "No scan data available"
}

show_network_status() {
echo -e "\n${BLUE}🌐 Network Status${NC}"
echo "============================================"

### # IP 주소

echo "WiFi IP: $(ip route get 8.8.8.8 | grep -oP 'src \K\S+' 2>/dev/null || echo 'Not
connected')"

### # ROS2 네트워크

echo "ROS Domain ID: ${ROS_DOMAIN_ID:-'Not set'}"
echo "DDS Implementation: ${RMW_IMPLEMENTATION:-'Default'}"

### # 멀티캐스트 테스트

if docker exec rpi_slam_system ros2 multicast receive >/dev/null 2>&1 &


then
sleep 2
pkill -f "multicast receive"
echo -e "${GREEN}✅ Multicast: Working${NC}"
else
echo -e "${YELLOW}⚠ Multicast: May have issues${NC}"
fi
}

### # 연속 모니터링 모드

continuous_monitor() {
echo -e "${GREEN}🔄 Starting continuous monitoring (Ctrl+C to stop)${NC}"

while true; do
clear
echo -e "${GREEN}🚀 RaspberryPi SLAM System Monitor${NC}"
echo "Last Update: $(date)"

show_system_info
show_docker_status
show_ros_status
show_lidar_status
show_network_status

echo -e "\n${BLUE}Press Ctrl+C to stop monitoring${NC}"
sleep 5
done
}

### # 메인 스크립트

case "${1:-status}" in
"continuous"|"cont"|"c")
continuous_monitor
;;
"system"|"sys"|"s")
show_system_info
;;
"docker"|"d")
show_docker_status


### ;;

"ros"|"r")
show_ros_status
;;
"lidar"|"l")
show_lidar_status
;;
"network"|"net"|"n")
show_network_status
;;
"help"|"h")
echo "Usage: $0 [option]"
echo "Options:"
echo " status (default) - Show all status information"
echo " continuous - Continuous monitoring mode"
echo " system - System information only"
echo " docker - Docker status only"
echo " ros - ROS2 status only"
echo " lidar - LiDAR status only"
echo " network - Network status only"
;;
*)
show_system_info
show_docker_status
show_ros_status
show_lidar_status
show_network_status
;;
esac

</details>

<details>
<summary><code>pc_visualization/rviz_configs/hector_slam.rviz</code></summary>
# RViz2 설정 - Hector SLAM 시각화
# 개발 PC에서 사용

Panels:

- Class: rviz_common/Displays


Name: Displays

- Class: rviz_common/Selection
Name: Selection
- Class: rviz_common/Tool Properties
Name: Tool Properties
- Class: rviz_common/Views
Name: Views
- Class: rviz_common/Time
Name: Time

Visualization Manager:
Class: ""
Displays:

- Alpha: 0.5
Cell Size: 1
Class: rviz_default_plugins/Grid
Color: 160; 160; 164
Enabled: true
Line Style:
Line Width: 0.029999999329447746
Value: Lines
Name: Grid
Normal Cell Count: 0
Offset:
X: 0
Y: 0
Z: 0
Plane: XY
Plane Cell Count: 100
Reference Frame: <Fixed Frame>
Value: true
- Class: rviz_default_plugins/LaserScan
Enabled: true
Name: LiDAR Scan
Topic:
Depth: 5
Durability Policy: Best Effort
Filter size: 10


History Policy: Keep Last
Reliability Policy: Best Effort
Value: /scan
Color: 255; 0; 0
Size (Pixels): 3
Style: Points
Value: true

- Alpha: 0.7
Class: rviz_default_plugins/Map
Color Scheme: map
Draw Behind: false
Enabled: true
Name: Occupancy Map
Topic:
Depth: 1
Durability Policy: Transient Local
Filter size: 10
History Policy: Keep Last
Reliability Policy: Reliable
Value: /map
Update Topic:
Depth: 5
Durability Policy: Volatile
History Policy: Keep Last
Reliability Policy: Reliable
Value: /map_updates
Use Timestamp: false
Value: true
- Class: rviz_default_plugins/TF
Enabled: true
Filter (blacklist): ""
Filter (whitelist): ""
Frame Timeout: 15
Frames:
All Enabled: true
Marker Scale: 1
Name: TF


Show Arrows: true
Show Axes: true
Show Names: true
Tree:
map:
odom:
base_link:
base_laser:
{}
Update Interval: 0
Value: true

- Angle Tolerance: 0.1
Class: rviz_default_plugins/Odometry
Covariance:
Orientation:
Alpha: 0.5
Color: 255; 255; 127
Color Style: Unique
Frame: Local
Offset: 1
Scale: 1
Value: true
Position:
Alpha: 0.3
Color: 204; 51; 204
Scale: 1
Value: true
Value: false
Enabled: true
Keep: 100
Name: Robot Odometry
Position Tolerance: 0.1
Shape:
Alpha: 1
Axes Length: 0.1
Axes Radius: 0.01
Color: 255; 25; 0
Head Length: 0.1


Head Radius: 0.03
Shaft Length: 0.1
Shaft Radius: 0.01
Value: Arrow
Topic:
Depth: 5
Durability Policy: Volatile
Filter size: 10
History Policy: Keep Last
Reliability Policy: Reliable
Value: /poseupdate
Value: true

Enabled: true
Global Options:
Background Color: 48; 48; 48
Fixed Frame: map
Frame Rate: 30
Name: root
Tools:

- Class: rviz_default_plugins/Interact
Hide Inactive Objects: true
- Class: rviz_default_plugins/MoveCamera
- Class: rviz_default_plugins/Select
- Class: rviz_default_plugins/FocusCamera
- Class: rviz_default_plugins/Measure
- Class: rviz_default_plugins/SetInitialPose
Theta std deviation: 0.2617993877991494
Topic:
Depth: 5
Durability Policy: Volatile
History Policy: Keep Last
Reliability Policy: Reliable
Value: /initialpose
X std deviation: 0.5
Y std deviation: 0.5
- Class: rviz_default_plugins/SetGoal
Topic:
Depth: 5


Durability Policy: Volatile
History Policy: Keep Last
Reliability Policy: Reliable
Value: /goal_pose
Value: true
Views:
Current:
Class: rviz_default_plugins/Orbit
Distance: 20
Enable Stereo Rendering:
Stereo Eye Separation: 0.06
Stereo Focal Distance: 1
Swap Stereo Eyes: false
Value: false
Focal Point:
X: 0
Y: 0
Z: 0
Focal Shape Fixed Size: true
Focal Shape Size: 0.05
Invert Z Axis: false
Name: Current View
Near Clip Distance: 0.01
Pitch: 1.5708
Target Frame: <Fixed Frame>
Value: Orbit (rviz_default_plugins)
Yaw: 0
Saved: ~
Window Geometry:
Displays:
collapsed: false
Height: 1056
Hide Left Dock: false
Hide Right Dock: false
QMainWindow State:
000000ff00000000fd0000000400000000000001560000041afc0200000008f
b0000001200530065006c0065006300740069006f006e00000001e10000009b
0000005c00fffffffb0000001e0054006f006f006c002000500072006f00700065
0072007400690065007302000001ed000001df00000185000000a3fb00000012


0056006900650077007300200054006f006f02000001df000002110000018500
000122fb000000200054006f006f006c002000500072006f0070006500720074
006900650073003203000002880000011d000002210000017afb000000100044
006900730070006c006100790073010000003d0000041a000000c900fffffffb00
00002000730065006c0065006300740069006f006e002000620075006600660
06500720200000138000000aa0000023a00000294fb00000014005700690064
006500470065007400000001e10000009b0000000000000000fb0000000c0
04b00690065006e00650063007400000001e10000009b0000000000000000
000000010000010f0000041afc0200000003fb0000001e0054006f006f006c00
2000500072006f0070006500720074006900650073010000004100000078000
0000000000000fb0000000a00560069006500770073010000003d0000041a0
00000a400fffffffb0000001200530065006c0065006300740069006f006e01000
0025a000000b20000000000000000000000020000049000000a9fc0100000
001fb0000000a00560069006500770073030000004e00000080000002e1000
0019700000003000007800000003efc0100000002fb0000000800540069006d
00650100000000000007800000000000000000fb0000000800540069006d0
06501000000000000045000000000000000000000023e0000041a0000000
4000000040000000800000008fc0000000100000002000000010000000a0
054006f006f006c00730100000000ffffffff0000000000000000
Selection:
collapsed: false
Time:
collapsed: false
Tool Properties:
collapsed: false
Views:
collapsed: false
Width: 1920
X: 0
Y: 27

</details>

<br>

## 📊 성능 최적화

### ● ARM64 네이티브 컴파일

```
● 멀티 스테이지 Docker 빌드
```

### ● DDS 네트워크 최적화

### ● 라즈베리파이 5 전용 튜닝

## 🆘 문제 해결

### 라이다 연결 문제

### # USB 장치 확인

lsusb

### # 시리얼 포트 확인

ls -la /dev/ttyUSB* /dev/ttyACM*

### # 권한 설정

sudo chmod 666 /dev/ttyUSB0

### 성능 문제

### # CPU 성능 모드

echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

### # 메모리 상태 확인

free -h

### # 쓰로틀링 확인

vcgencmd measure_temp
vcgencmd get_throttled

## 📝 라이센스

MIT License - 자유롭게 사용하세요!


