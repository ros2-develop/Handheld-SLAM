🚀 라즈베리파이 5 + D500 라이다 SLAM 시스템헤드리스 Docker 환경에서 실행되는 완전한 SLAM 시스템입니다.📊 시스템 인포그래픽이 프로젝트의 아키텍처, 워크플로우, 주요 성능 지표를 시각적으로 보여주는 인터랙티브 인포그래픽을 만들었습니다. 아래 링크를 통해 확인해 보세요.➡️ 인포그래픽 보기 (./rpi-slam-infographic.html)(참고: 위 링크의 HTML 파일을 웹 브라우저에서 열어주세요.)⚡ 빠른 시작1. 시스템 준비# 프로젝트 클론
git clone <repository-url>
cd rpi_slam_system

# 라즈베리파이 최적화 적용
sudo ./scripts/setup_rpi.sh
2. 빌드 & 배포# 도커 이미지 빌드 (15-30분 소요)
./scripts/build.sh

# SLAM 시스템 배포
./scripts/deploy.sh
3. 모니터링# 실시간 모니터링
./scripts/monitor.sh continuous

# 상태 확인
./scripts/monitor.sh
🔧 지원하는 SLAM 알고리즘1단계: Hector SLAM - 빠르고 안정적 (권장 시작점)2단계: GMapping - 더 정확한 맵핑3단계: Cartographer - 최고 품질 (높은 CPU 사용량)🖥️ PC에서 시각화개발 PC에서 다음 명령어로 시각화:# ROS2 도메인 설정
export ROS_DOMAIN_ID=42

# 토픽 확인
ros2 topic list

# RViz2 실행
rviz2 -d pc_visualization/rviz_configs/hector_slam.rviz
📊 성능 최적화ARM64 네이티브 컴파일멀티 스테이지 Docker 빌드DDS 네트워크 최적화라즈베리파이 5 전용 튜닝🆘 문제 해결라이다 연결 문제# USB 장치 확인
lsusb

# 시리얼 포트 확인  
ls -la /dev/ttyUSB* /dev/ttyACM*

# 권한 설정
sudo chmod 666 /dev/ttyUSB0
성능 문제# CPU 성능 모드
echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

# 메모리 상태 확인
free -h

# 쓰로틀링 확인
vcgencmd measure_temp
vcgencmd get_throttled
📝 라이센스MIT License - 자유롭게 사용하세요!
