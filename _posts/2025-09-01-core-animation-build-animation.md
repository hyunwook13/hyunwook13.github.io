---
layout: post
title: Core Animation - Build Animation
tags: [swift, core-animation]
date: 2025-09-01 10:03:00 +0900
---

- **레이어 계층 관리 메서드**

| 동작 | 메서드 | 설명 |
| :-- | :-- | :-- |
| 레이어 추가 | `addSublayer:` | 새로운 서브레이어를 현재 레이어에 추가. 배열의 끝에 들어가므로 같은 `zPosition`일 때는 제일 위에 표시됨. |
| 레이어 삽입 |  `insertSublayer:atIndex:` |서브레이어 배열의 특정 인덱스 또는 다른 레이어 기준 위/아래에 삽입. 실제 표시 순서는 `zPosition` 값이 우선, 그다음 배열 순서가 반영됨. |
| ^^ | ^^ `insertSublayer:above:` | ^^ |
| ^^ | ^^ `insertSublayer:below:` | ^^ |
| 레이어 제거 | `removeFromSuperlayer` | 현재 레이어에서 부모(`superlayer`)와의 관계를 끊고 제거. |
| 레이어 교체 | `replaceSublayer:with:` | 기존 서브레이어를 다른 레이어로 교체. 교체할 레이어가 이미 다른 계층에 있으면 자동 제거 후 새 계층에 추가됨. |

- 서브레이어에는 **크기(bounds)** 와 **위치(position)**를 지정해야 화면에 출력됨
    
    ```swift
    myLayer.bounds = CGRectMake(0, 0, 100, 100); // 크기 100x100
    myLayer.position = CGPointMake(200, 200);   // 부모 좌표에서 (200,200)에 배치
    ```
    
- 부모의 속성은 자식 레이어에도 전달됨
- 부모 레이어에서 자식 레이어의 위치나 크기를 자동으로 조정 가능
    - 위치 특정 위치( midX, midY 등 )로 정렬
    - 크기, 상대 배치 ( A를 기준으로 10정도 떨어져라 )
        - iOS → 오토레이아웃 이용
        - macOS → Core Animation 자체의 제약(Constraints) 시스템 제공
        
        ```swift
        layerA.addConstraint(CAConstraint(attribute: .midX,
                                          relativeTo: "superlayer",
                                          attribute: .midX))
        layerA.addConstraint(CAConstraint(attribute: .midY,
                                          relativeTo: "superlayer",
                                          attribute: .midY))
        ```
        
- **Coordinate Conversion (좌표 변환)**
    - 때때로 **한 레이어 좌표계의 점/사각형을 다른 레이어 좌표계로 바꿔야** 할 때 있음.
    - CALayer 제공 메서드:
        - convertPoint(_:from:), convertPoint(_:to:)
        - convertRect(_:from:), convertRect(_:to:)
    - 시간(time) 변환도 가능:
        - convertTime(_:from:), convertTime(_:to:)
        - 애니메이션 속도를 다르게 준 레이어끼리 타이밍 맞출 때 사용.