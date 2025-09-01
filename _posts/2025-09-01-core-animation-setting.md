---
layout: post
title: Core Animation - Setting
tags: [swift, core-animation]
date: 2025-09-01 10:01:00 +0900
---

- **Core Animation 지원 활성화 방법**
    - iOS → 항상 활성화되어 있고, 모든 뷰가 기본적으로 레이어를 갖고 있음
    - macOS → 직접 활성화 필요
        1. **QuartzCore** 프레임워크 링크
        2. NSView 객체 레이어 지원 켜기
            1. 가능하다면 윈도우의 **content view**에서 활성화하는 게 권장, setWantsLayer 호출
- 상황에 맞는 레이어 클래스 사용
    - 보통 CALayer로 충분 허나 특정 목적에 맞게 최적화된 클래스 존재
        1. 큰 이미지 
            1. CATiledLayer : 큰 이미지를 조각 단위로 잘라 효율적으로 표시
        2. Metal, OpenGL ES로 직접 그리기 
            1. CAMetalLayer, CAEAGLLayer : GPU 렌더링을 위한 전용 레이어
        3. 특수 효과
            1. CAEmitterLayer: 입자 효과(파티클)
            2. CAReplicatorLayer: 뷰/레이어 복제 효과
- makeBackingLayer를 사용하여 원하는 레이어 클래스로 바꿈
    - 혹은 Layer-hosting으로 새로운 레이어를 자신이 직접 붙여 모든 것을 관리

### 용도에 따른 레이어 클래스들

| 클래스 | 용도 |
| :-- | :-- |
| **CAEmitterLayer** | Core Animation 기반 파티클(입자) 시스템 구현에 사용됩니다. 파티클 생성과 출발점을 제어합니다. |
| **CAGradientLayer** | 레이어의 영역(둥근 모서리 포함)을 채우는 색상 그라데이션을 그릴 때 사용됩니다. |
| **CAMetalLayer** | Metal을 사용하여 레이어 콘텐츠를 렌더링할 텍스처를 설정하고 제공할 때 사용됩니다. |
| **CAEAGLLayer** / **CAOpenGLLayer** | OpenGL ES(iOS) 또는 OpenGL(macOS)로 콘텐츠를 렌더링하기 위한 백업 저장소와 컨텍스트를 설정할 때 사용됩니다. |
| **CAReplicatorLayer** | 하나 이상의 서브레이어를 자동으로 복제할 때 사용됩니다. 지정한 속성을 이용해 복제본의 모양이나 속성을 변경할 수 있습니다. |
| **CAScrollLayer** | 여러 서브레이어로 구성된 큰 스크롤 가능한 영역을 관리할 때 사용됩니다. |
| **CAShapeLayer** | 베지어 곡선을 이용해 경로(Path) 기반 도형을 그릴 때 사용됩니다. 항상 선명하게 스케일되므로 유리합니다. 다만, 이 과정은 메인 스레드에서 경로를 그린 뒤 캐싱하기 때문에 성능 주의가 필요합니다. |
| **CATextLayer** | 일반 문자열이나 속성 문자열(NSAttributedString)을 렌더링할 때 사용됩니다. |
| **CATiledLayer** | 큰 이미지를 잘게 나누어(tile) 부분적으로 렌더링하고 확대/축소를 지원할 때 사용됩니다. |
| **CATransformLayer** | 다른 레이어 클래스들이 단순히 평면적으로(flat) 계층화되는 것과 달리, 실제 3D 레이어 계층 구조를 렌더링할 때 사용됩니다. |
| **QCCompositionLayer** | Quartz Composer composition을 렌더링할 때 사용됩니다. (macOS 전용) |

- macOS 레이어 이미지 관리 3가지 방법
    - 이미지 객체를 contents 속성에 직접 할당
        - 콘텐츠가 거의 변하지 않는 경우에 적합
    - 델리게이트(delegate) 객체를 레이어에 지정하고, 델리게이트가 레이어의 콘텐츠를 그림
        - 콘텐츠가 주기적으로 바뀌거나, 뷰 같은 외부 객체에서 제공되는 경우 적합
    - 레이어 서브클래스를 정의하고, 그 안에서 그리기 메서드를 오버라이드하여 직접 콘텐츠 제공
        - (커스텀 레이어를 만들 필요가 있거나, 레이어의 근본적인 그리기 동작을 바꾸고 싶을 때 적합)
- 뷰가 계속 바뀌는 상황에서는 델리게이트를 사용해야 하는데, 2가지 방법 존재
    1. **displayLayer:**
        - 내가 직접 비트맵(이미지)을 만들고, layer.contents에 넣어줌
        - → 즉, 레이어에 최종 그림을 “완성본”으로 넣어주는 방식
    2. **drawLayer:inContext:**
        - Core Animation이 먼저 비트맵과 그래픽 컨텍스트를 준비해 줌
        - 내 메서드는 그 컨텍스트 안에 그냥 “그림을 그리기만” 하면 됨
        - → 즉, 그림을 “직접 그려 넣는 방식”
    
    → 만약 둘다 구현했다면 1번만 호출됨
    
- 이미지 배치 방법
    1. 위치 기반 
        - 나머지 부분은 다 짤림
            - 정사각형 이미지를 TopLeft에 배치한다면, 키패드를 예시로 123, 456,78 부분은 다 짤림
        ![image.png](/assets/images/coreanimation/core-2.png)
        
    2. 스케일링 기반
        ![image.png](/assets/images/coreanimation/core-3.png)
        
- 이미지에 대해서 픽셀이 아닌 비트맵을 저장하기 때문에 흐릴 수 있음
    - `layer.contentsScale  = UIScreen.main.scale` 을 하면 스케일 정보를 저장하여 진하게 보여줌
- macOS에서는 코어 이미지에 Filter를 입힐 수 있음