
### 3D 가면을 구현하기 위해 필요한 기술



https://github.com/google-ai-edge/mediapipe/blob/master/docs/solutions/face_mesh.md

### 얼굴 추적
- OpenCV: 얼굴 감지 및 위치 추적
- MediaPipe: 얼굴의 468개 랜드마크를 추적하여 정확한 얼굴 위치, 각도, 크기를 감지
- ARKit/ARCore: 얼굴의 위치와 회전을 감지하는 AR 플랫폼

### 3D 이미지화 및 가면 렌더링
- ThreeJS : 3D 렌더링 제공
(Meta 등의 제공은 대부분 Unity 플랫폼 기준으로 제공되는 라이브러리)

### 결론론

1. 이미지화 및 가면 렌더링링 - 평면 이미지 가면을 3D화 시켜 주어야 한다. -> ThreeJS
2. 얼굴 추적 - 가면을 얼굴의 위치에 맞게 배치해주어야 한다. -> mediapipe facemesh
