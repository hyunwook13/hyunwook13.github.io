---
layout: post
title: Core Animation - Advanced Animation Tricks
tags: [swift, core-animation]
date: 2025-09-02 10:00:00 +0900
---

- 기존엔 Property Animation로써 단순 이동, 투명해지는 효과였다면 Transition Animation이 존재함
    - 뷰를 밀거나 위에서 들어오거나 서서히 바뀌는 효과등이 존재
    
    ```swift
    let transition = CATransition()
    transition.startProgress = 0
    transition.endProgress = 1.0
    transition.type = .push
    transition.subtype = .fromRight
    transition.duration = 1.0
    
    myView1.layer.add(transition, forKey: "transition")
    myView2.layer.add(transition, forKey: "transition")
    
    myView1.isHidden = true
    myView2.isHidden = false
    ```
    
- 애니메이션의 타이밍은 중요하고, 그러므로 더 세세하게 컨트롤 가능
    - **시작 시점 제어** → beginTime
        
        → “애니메이션 A 끝나고 B 시작” 같은 체이닝 가능
        
    - **종료 후 상태 유지** → fillMode
        
        → “애니메이션 끝나도 원래 값으로 튀지 말고, 마지막 상태 그대로 보여줘”
        
    - **왕복 & 반복** → autoreverses, repeatCount
        
        → 펄스 효과, 흔들림 효과 만들 때 유용
        
    - **시간 변형** → speed, timeOffset
        
        → 전체 애니메이션 빠르게/느리게, 특정 구간만 늦게 시작
        
- 애니메이션을 멈췄다가 다시 진행 가능
    
    ```swift
    func pauseLayer(_ layer: CALayer) {
        let pausedTime = layer.convertTime(CACurrentMediaTime(), from: nil)
        layer.speed = 0.0
        layer.timeOffset = pausedTime
    }
    
    func resumeLayer(_ layer: CALayer) {
        let pausedTime = layer.timeOffset
        layer.speed = 1.0
        layer.timeOffset = 0.0
        layer.beginTime = 0.0
        let timeSincePause = layer.convertTime(CACurrentMediaTime(), from: nil) - pausedTime
        layer.beginTime = timeSincePause
    }
    ```
    
- 트랜잭션
    - 묵시적 애니메이션은 트랜잭션을 자동으로 만들어서 실행해줌
    - 명시적은 트랜잭션을 만들어서 사용하고, 그 내부에 여러가지 요소를 넣어 한꺼번에 처리를 할 수 있음