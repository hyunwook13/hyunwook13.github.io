---
layout: post
title: Core Animation - Improving Animation Performance
tags: [swift, core-animation]
date: 2025-09-02 10:00:00 +0900
---

- 코어 애니메이션을 사용하면 프레임 속도는 일반적으로 개선되지만 만능은 아님
    - 레이어 정책이나 업데이트 방식에 따라 다르기에, Insturment를 통해서 확인을 해야한다
- NSView의 기본 redraw 정책은 **원래의 그리기 방식**을 그대로 유지하는데, 불필요하게 전체를 다시 그리는 경우가 많음
    
    → 성능이 떨어질 수 있다, 필요할 때만 다시 그리는 정책을 도입 **`NSViewLayerContentsRedrawOnSetNeedsDisplay` 권장**
    
- 레이어를 다시 그리는 것이 아닌 업데이트를 통해 CPU 사용량을 줄일 수 있다

### 팁

1. **가능한 경우 Opaque Layers(불투명 레이어) 사용**
    1. opaque = YES로 설정 
2. CAShapeLayer에서는 단순한 Path 사용
    1. 주어진 path를 통해 비트맵을 랜더링하는데, 복잡하거나 크기가 자주 변하면 CPU 성능 떨어짐
    
    → 복잡한 path를 여러 개의 단순한 shape 레이어로 쪼개서 GPU 합성(compositing)에 맡기면 훨씬 빠를 수 있습니다.
    
3. 동일한 레이어는 contents를 직접 설정
    1. 테이블셀에 하트 버튼이 존재한다면 이미지를 바로 콘텐츠에 넣음
    2. 만약 따로 만들게 되면 비트맵을 각각 만들어서 사용함
4. **레이어 크기는 정수 값으로 설정**
    1. 100.5 같은 반올림 크기는 내부적으로 추가 연산이 필요 → 성능 저하 가능.
5. **필요할 때만 비동기 렌더링(drawsAsynchronously) 사용**
    1. `layer.drawsAsynchronously = true` 를 통해서 랜더링을 비동기로 돌려서 메인쓰레드의 점유율을 낮춤
6. **그림자(Shadow)는  shadowPath 지정**
    1. 그림자 경계를 자동 계산하는 건 비용이 큼