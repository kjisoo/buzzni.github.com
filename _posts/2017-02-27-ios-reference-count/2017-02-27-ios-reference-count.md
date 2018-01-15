---
layout: post
title: iOS 개발하며 처음 부딪히는 문제 Reference Count에 대하여.
tags: [ios, swift, referencecount, objective-c, error]
author: 기원준
image: assets/img/character/roy.png 
type: ENGINEERING
---

# iOS 개발하며 처음 부딪히는 문제 Reference Count에 대하여.

# Objective-C 부터 Swift까지 Reference Count에 대하여

iOS를 처음 공부를 하면서 접하게 되는 참조 카운트(reference count)에 대하여 짧은 글을 써보려고 합니다. c, c++, java, c#... 등의 다른 언어를 배우고 objective-c , swift를 접하는 분들이 많을것으로 생각되고 저 역시 c부터 공부를 하였고 이후 아이폰이 국내에 들어오면서 iOS개발자로 전향을 하게 되었습니다. 제가 이해했던 바를 정리함으로써 이 글을 보시는 분의 이해에 도움이 되었으면 합니다. 

* * *

### Reference Count

오브젝티브씨의 특징 중 하나는 기본적으로 포인터로 구조가 이뤄져 있습니다. 

**`C++`** **` class CFoo {}`** **` CFoo foo = CFoo()`**

위의 형태를 objective - c에서는 사용을 할수가 없습니다. 

**`objective- c`** **` @interface Foo{} @end`** **` Foo* foo = [[Foo alloc] init];`**

**`swift`** **` class Foo{}`** **` let foo = Foo()`**

의 형태로 메모리를 할당하는 형태로만 가능하고 그러다보니 메모리내에서 Heap 영역에 객체가 생성이 되고 해제가 됩니다. 자, Heap의 특징인 **메모리를 할당했으면 해제도 직접 해야한다.** 라는 문제가 발생되죠. c언어에서는 free, delete를 개발자가 직접하고 자바는 가바지가 돌면서 하겠죠. 그 중에서 NextStep 진영에서 채택한 방법이 **Reference Count기법** 입니다. 그렇다고 이게 다른 언어에서 없는 개념은 아닙니다. [c++에서 스마트 포인터](http://jason-heo.github.io/cpp/2014/03/16/smart-pointer3.html) [CodeProject에서 참고한 reference-counting](https://www.codeproject.com/Articles/64111/Building-a-Quick-and-Handy-Reference-Counting-Clas) (_위의 링크처럼 다른 언어에서도 사용되는 기법입니다만 다른 언어에서는 옵셔널한 성격이 강한 반면 objective-c / swift에서는 필수적인 요소입니다._) 의도는 간단합니다. 

  * 내가 참조 하고 있으니 참조카운트를 1증가하고
  * 내가 더이상 안쓰면 참조카운트를 1감소시킨다.
  * 카운트가 잘 지켜져서 0이되면 아무도 날 참조하지 않는 상황이니 난 없어져도 되겠다 싶어서 해제됩니다.
![예시1](https://www.codeproject.com/KB/cpp/rcptr/rcptr01.png) 위의 그림이 이를 나타냅니다. 

> 위의 Reference Count 를 잘 지키기 위해 **Ownership 정책** 이라고 나옵니다. 오너쉽을 한줄로 이야기하자면 **내가 1 증가시켰으면 책임지고 내가 1 감소시키자** 입니다. 책임져!!! 내가 쓴다고 증가를 시켜놓고 필요하지 않는데 감소를 하지 않는다면 다른데서 참조하고 있는 것들이 아무리 잘해도 1이 남아서 메모리릭이 발생하게 되겠죠. 이미지파일등의 용량이 큰 파일들을 참조하고있던 것이 해제가 안되고 쌓인다면 메모리 워닝이 발생될것이고 점차 쌓이다가 앱이 뻗어버리는 사태가 발생되게 됩니다.

  자 위의 내용이 가장 기본적인 reference count의 기본 내용 및 이를 잘 지키고자하는 ownership 의 내용이였습니다. 그러나 위의 내용을 잘 지킨다 해도 순환참조의 문제로 객체가 해제가 안되는 상황이 발생되게 됩니다. 

## 아래에서 주로 발생되는 케이스에 대해서 이야기를 하려 합니다.

### 순환참조??? reference cycles... 뭘까.?

레퍼런스 카운팅에 대해서 이야기를 하다보면 꼭 나오는 주제... 순환참조가 뭘까요....? 아래에 이미지들은 위에 첨부된 codeproject에 들어 있는 내용을 발췌하였습니다. ![순환참조](https://www.codeproject.com/KB/cpp/rcptr/rcptr02.png) 메모리 관리 기법으로 reference count를 하기로 했는데... 위의 그림처럼 내가 널 잡고 너도 날 잡고... 누구하나가 먼저 손을 놔야 나도 놓을 텐데 둘다 고집이 쎼서 손을 놓을 생각을 하지 않고 있습니다. 이 내용이 순환참조입니다. 그럼 해결법은 무엇일까요.?? 아. 그럼 한쪽이 살포시~~ 잡게 하자. 한쪽이 살포시~~ 잡게해서 살포시~~ 잡은 놈이 고집 안피우고 먼저 놓으면 다른 놈도 놓게 될것이고... 그럼되지 않을까? 라는 생각에서 출발합니다. 그림으로는 아래와 같습니다. ![weak참조](https://www.codeproject.com/KB/cpp/rcptr/rcptr03.png) 이를 지원하기 위해 몇몇 키워드가 있고 

**`\- objective -c`**

**`@property (nonatomic, strong) NSString* titleString; // 참조할때 카운트 + 1 (strong)`** **` @property (nonatomic, assign) NSString* titleString; // 참조할때 카운트를 늘리지 않는다. 참조하던 객체가 해제되면 댕글리포인터로 남게된다. (assign)`** **` @property (nonatomic, weak) NSString* titleString; // 참조할떄 카운트를 늘리지 않는다. 참조하던 객체가 해제되면 nil로 변경된다. (weak)`** **` \- swift`**

**`var titleString : String //기본적으로 strong 속성을 갖습니다.`** **` weak var titleString : String? //weak는 optional 속성으로 nil이 될수 있으므로 !or ?로만 가능하고 String로는 될수 없습니다.`**

**`그외에 unowned 속성도 있습니다.`**

  위의 코드에 보이는 키워드(strong, assign, weak, unowned) 등이 있습니다. 참고로 weak는 처음엔 없었으나 objective-c 2.0, modern objective c를 거치면서 필요에 의해서 추가된 키워드입니다. 코드에서 명시한것 처럼 **참조하고 있는 객체가 해제되면 nil이 된다**라는 특징이자 강점을 가지고 있습니다. 이래의 방법은 objective-c에서 블럭 코딩 / swift에서 closure를 사용할때 순환참조를 피하기 위해 제가 사용하는 방법입니다.  

**`\- 통신을 하고 나서 결과로 어떤 처리를 할때`**

**`//swift `** **`//{ 이 안에서 capture가 발생되서 순환참조가 일어 날수 있습니다. 해서 [weak self] 로 self가 weak속성을 가지게 합니다. }`** **`self.completeHandler = { [weak self] in `**

**` //여기서 self는 weak.`** **` guard let `self` = self else {`** **`    return`** **` }`** **` //계속 self?.updateUI로 접근하는 것을 막고자 위에서 guard를 사용하여 self를 언래핑하여 strong한 self로 가져옵니다.`**

**`//guard를 지난 여기서의 self는 strong`** **`    self.updateUI() // self?.updateUI()로 사용되지 않는다.`** **`}`**

**`//objective - c `** **`typedef void (^COMPLETE_HANDLER)();`** **`...`** **`@property (nonatomic, strong) COMPLETE_HANDLER handler;`** **`...`** **`\- (void)zoo {`**