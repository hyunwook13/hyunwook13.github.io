---
layout: post
title: Core Animation - Changing a Layer’s Default Behavior
tags: [swift, core-animation]
date: 2025-09-02 10:00:00 +0900
---

- CAAction을 상속하여 기본 액션 대신 내가 원하는 액션을 사용할 수 있음
    
    ```swift
    import UIKit
    import QuartzCore
    
    // 커스텀 액션: position 속성이 바뀌면 파란색으로 변하는 애니메이션 실행
    class CustomAction: NSObject, CAAction {
        func run(forKey event: String,
                 object anObject: Any,
                 arguments dict: [AnyHashable : Any]?) {
            
            if let layer = anObject as? CALayer {
                print("🔥 Action triggered for key: \(event)")
                
                // position 변경일 때
                if event == "position" {
                    let anim = CABasicAnimation(keyPath: "backgroundColor")
                    anim.fromValue = layer.backgroundColor
                    anim.toValue = UIColor.blue.cgColor
                    anim.duration = 1.0
                    
                    layer.add(anim, forKey: "bgColorChange")
                    layer.backgroundColor = UIColor.blue.cgColor
                }
            }
        }
    }
    
    // 레이어 생성
    let myLayer = CALayer()
    myLayer.frame = CGRect(x: 50, y: 50, width: 100, height: 100)
    myLayer.backgroundColor = UIColor.red.cgColor
    
    // position 속성이 바뀔 때 실행할 액션 등록
    myLayer.actions = ["position": CustomAction()]
    
    // 이 순간 → position 바뀌면서 색상도 파란색으로 바뀌는 애니메이션 실행됨
    myLayer.position = CGPoint(x: 200, y: 200)
    ```
    
- 액션을 찾는 과정
    1. **Delegate**
        - 레이어에 delegate가 있고
        - actionForLayer(_:forKey:) 구현되어 있으면 → 반환된 액션 실행
        - nil → 다음 단계
        - NSNull → 검색 종료, 액션 없음
        
        ```swift
        class MyLayerDelegate: NSObject, CALayerDelegate {
            func action(for layer: CALayer, forKey event: String) -> CAAction? {
                if event == "contents" {
                    let transition = CATransition()
                    transition.duration = 1.0
                    transition.timingFunction = CAMediaTimingFunction(name: .easeIn)
                    transition.type = .push
                    transition.subtype = .fromRight
                    return transition
                }
                return nil
            }
        }
        
        let myLayer = CALayer()
        let delegate = MyLayerDelegate()
        myLayer.delegate = delegate
        
        // 👉 이때 contents 바꾸면 push transition 애니메이션 실행
        myLayer.contents = UIImage(named: "newImage")?.cgImage
        ```
        
    2. **Layer의 actions 딕셔너리**
        
        ```
        myLayer.actions = ["contents": myCustomAction]
        ```
        
    3. **Style 딕셔너리 안의 actions**
        - style["actions"] 안에서 검색
    4. **defaultActionForKey(_:)**
        - 서브클래싱해서 오버라이드 가능
    5. **Core Animation 기본 암묵적 애니메이션** 실행
- **내가 Action을 넣을 수 있는 방법**
    - **특정 상황에서만 필요** → Delegate 방식 사용
    - **Delegate 안 쓰는 레이어** → actions 딕셔너리 사용
    - **내가 만든 커스텀 속성용** → style 딕셔너리에 추가
    - **레이어 기본 동작 자체 바꾸고 싶음** → 서브클래싱 후 defaultActionForKey(_:) 오버라이드
- 애니메이션을 비활성화 할 수 있음
    - `CATransaction.setValue(true, forKey: kCATransactionDisableActions)`