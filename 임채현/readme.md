# 멀티모달

- 여러 개의 데이터 형식을 가지고 수행하는 AI
    - 주로 텍스트, 이미지, 음성, 비디오 등
- text 데이터와 함께 쓰는 경우가 가장 활발하게 상용화 되는

<aside>
📌

### 멀티모달 VS 멀티모델 ?

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/97b9ba3d-ae7c-415a-a91d-37a659d6459e/b20dcb3b-c9d3-4384-8309-33dd11dd7313/image.png)

- input data 종류가 2가지 이상인 경우, 모델은 1개 이상

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/97b9ba3d-ae7c-415a-a91d-37a659d6459e/14220e95-0a21-4230-8e29-19a27ae42eaa/image.png)

- input data의 종류가 1가지면서 여러 개의 모델을 거치는 방식
</aside>

## ChatGPT 멀티모달

https://cookbook.openai.com/examples/multimodal/vision_fine_tuning_on_gpt4o_for_visual_question_answering

### 이미지 - 텍스트

1. **CLIP (Contrastive Language-Image Pre-Training)**
- 대규모 웹 언어-이미지 병렬 데이터셋에서 언어와 이미지 간의 상호 작용을 학습하는 방식으로 구성
- 텍스트 입력 만으로도 주어진 정보에 해당하는 이미지 정보를 얻어내어 활용 가능
- 반대로 이미지 입력에서 원하는 텍스트 정보를 추출 가능

2. **Zero-Shot Learning**
- 학습 중에 학습한 언어적 표현과 이미지 특징을 활용하여, 학습 과정에서 보지 않은 클래스에 대한 이미지 분류를 수행
- 모델이 이미지에 대한 언어적 표현과 이미지 자체의 특징을 고려하여 새로운 클래스에 대한 예측을 수행

3. **VQA (Vision Question Answering)**
- 입력 모달리티인 이미지와 관련된 질문에 대한 답을 자연어로 출력해 주는 작업
- 비전 데이터에 존재하는 객체나 배경에 대한 질문을 할 수도 있고, 인물의 상황과 행동에 관한 질문에 답을 얻을 수도 있음
