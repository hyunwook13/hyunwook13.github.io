---
layout: post
title: Core Animation - Animating
tags: [swift, core-animation]
date: 2025-09-01 10:02:00 +0900
---

- 암시적, 명시적 애니메이션이 존재
    - layer.opacity = 0.0 → 자연스러운 페이드 아웃
    - CABasicAnimation를 직접 만들어서 설정 → 더 많은 제어 가능
        - 팁) 명시적 애니메이션에서는 fromValue를 직접 지정하는 게 좋다 → 안하면 애니메이션이 안나올 수 있음
- CAKeyframeAnimation는 하나의 애니메이션 사이에 하나 더 도착하는 장소를 추가하는 것
    - 만약 맨 처음부터 끝까지 가는 애니메이션 중간에 CAKeyframeAnimation를 추가하면 아래처럼 됨
        ![image.png](/assets/images/coreanimation/core-4.png)
        
    - 배열을 통해 도착지를 넣을 수 있고, 원하는 값에 따라 타입을 바꿔야 함
        - 숫자 (opacity, cornerRadius) → NSNumber
        - 사각형 (frame, bounds) → NSValue(CGRect)
        - 좌표 (position) → NSValue(CGPoint) 배열 **또는** CGPath
        - 색상 (borderColor) → CGColor
        - 이미지 (contents) → CGImage 배열
- **Keyframe Animation Timing**

| **모드 (calculationMode)** | **동작 방식** | **특징 / 예시** |
| :-- | :-- | :-- |
| **kCAAnimationLinear** | 키프레임 값 사이를 직선적으로 보간 | 기본 직선 보간, 구간별 시간·이징 직접 지정 가능 |
| **kCAAnimationCubic** | 키프레임 값 사이를 곡선(스플라인)으로 보간 | 더 부드러운 곡선 애니메이션 |
| **kCAAnimationPaced** | Core Animation이 속도를 자동 계산해 일정하게 이동 | 전체 경로를 따라 **등속 이동**, 타이밍 직접 지정 불가 |
| **kCAAnimationCubicPaced** | 곡선을 따라 등속 이동 | 곡선 경로 + 속도 일정 |
| **kCAAnimationDiscrete** | 키프레임 값 사이에 보간 없음 → 점프 이동 | 값이 뚝뚝 바뀌는 애니메이션 (예: 스프라이트 프레임 전환) |


- 추가한 애니메이션은 삭제 가능
    - removeAnimation() or removeAllAnimations()
    - 허나 애니메이션 도중 삭제한다면 도중에 텔레포트를 진행함
    - presentation() 를 통해서 현재 레이어 위치를 알아내고, 포지션은 변경
        
        ```swift
        // 1) 특정 애니메이션 제거
        myLayer.removeAnimation(forKey: "move")
        
        // 2) 모든 애니메이션 제거
        myLayer.removeAllAnimations()
        
        // 3) 멈춘 시점의 상태 유지 (presentation 값 반영)
        if let pres = myLayer.presentation() {
            myLayer.position = pres.position   // 현재 움직이던 좌표를 최종값으로 고정
        }
        myLayer.removeAnimation(forKey: "move")
        ```
        
- 애니메이션을 그룹화해서 동시 시작 및 종료 가능
    
    ```swift
    // 그룹 애니메이션 만들기
    let moveAnim = CABasicAnimation(keyPath: "position")
    moveAnim.fromValue = NSValue(cgPoint: CGPoint(x: 80, y: 120))
    moveAnim.toValue   = NSValue(cgPoint: CGPoint(x: 300, y: 400))
    
    let fadeAnim = CABasicAnimation(keyPath: "opacity")
    fadeAnim.fromValue = 1.0
    fadeAnim.toValue   = 0.0
    
    let group = CAAnimationGroup()
    group.animations = [moveAnim, fadeAnim]
    group.duration = 3.0
    
    // 실행
    myLayer.add(group, forKey: "moveAndFade")
    ```
    
- 애니메이션이 끝났는지 판단하는 방법 2가지
    1. CATransaction의 completionBlock
    2. CAAnimationDelegate 사용