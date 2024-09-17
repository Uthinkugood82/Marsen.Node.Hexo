---
title: " [翻譯] AHA Programming 💡"
date: 2020/09/30 14:44:53
---

The dangers of DRY, the web of WET, the awesomeness of AHA.

## 前情提要

為什麼我要翻譯 ? 練習英文，同時學習程式開發的原則 。
程式開發原則其實超級多，稍稍整理一下。

### 物件導向程式設計的 SOLID 原則

這五個知名的原則，可以一言以蔽之 --- 高內聚、低耦合。  
而我自已更喜歡 [Teddy 老師的五則皆變:「面對原始碼改變的五種不同策略」](http://teddy-chen-tw.blogspot.com/2014/04/solid.html)

#### SRP(Single Responsibility Principle) 單一職責原則

A class should have only one reason to change  
以一個類別來說，應該只有一個引起它變化的原因。  

舉個實務上在改動代碼的例子:  
線上回報了一個購物車的問題，  
你去追查發現購物車保含禮品與折價券的情境下，才會發生問題，  
同時主管要求你，這段代碼發生異常時加上 Log 並發 Email 通知。  

你會怎麼改呢 ? 加個 if 判斷購物車/禮品/折價券之間的對應關係 ?  
使用 Try Catch 補捉異常，記錄 Log 並發送 Email ?  
那麼這些邏輯應該被加到哪些類別呢 ?  
請思考一下  
在這個原則底下，應該考慮 if 的邏輯屬於購物車/禮品/折價券中的哪一類 ?  
而三個商務邏輯如何互動，當前的設計也是要注意的方向。  
Log / Email 也是同樣的，是否也應該隸屬獨立類別呢。  
請思考一下，請思考一下。  

這遵循這個原則的程式設計下，可以有效減少單一類別被改變的機會。  
以上面情境為例，Log / Email / if 的邏輯都是不同的職責，  
如果設計得當，一個職責需要異動時只會修改到一個類別。

#### OCP(Open Closed Principle) 開閉原則

software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification
軟體實體(類別,模組,函式等等) 應該是可以擴展的，但不能被修改

同上的情境，遵循 OCP 在面對新增需求的時候，可以單純透過新增類別或方法來達成需求。

#### LSP(Liskov Substitution Principle) 里氏替換原則

Subtypes must be substitutable for their base types.  
基底類別應該要能夠被他的衍生類別給替代而不影響原本的功能

這裡提到的原則，其實是對繼承的限制，在強型別語言都已經有這方面的實作了。
以上述的例子來說，Log/Email 的實作至少應該有一層的抽象隔離，  
並以繼承抽象的實作來，  
也就是說實際上如何 Log 可以是 files、DB、Cloud 或任何其它的東西。  

這裡也跟下面的 ISP 相呼應。

#### ISP(Interface Segregation Principles) 介面隔離原則

No client should be forced to depend on methods it does not use.
不應該讓使用者去依賴他根本不會使用到的東西

介面是方法的抽象，所以我認為當一個介面有超過一個方法時，  
就應該思考一下有沒有違背這個原則。  
實務上會有許多的同類型的介面應該被歸屬到同一類之中，  
簡單可行的方法是當你實作一個介面時，會不會要跟著實作許多用不到的方法呢?
如果有的話，試著將它們拆分成不同的介面吧。

#### DIP(Dependency Inversion Principle) 依賴反轉原則

High-level modules should not depend on low-level modules. Both should depend on abstractions (e.g. interfaces).
高層模組不應該依賴於低層模組。兩者皆應該依賴抽象。

Abstractions should not depend on details. Details (concrete implementations) should depend on abstractions.
抽象不應該依賴細節。細節應該依賴抽象
DRY(an acronym for "")

![DRY](https://i.imgur.com/pI1bZ7h.png)

直接透過 wiki 的圖片來說明上面的例子。
想像 Object A 是購物車 Object B 是 Logger,  
Figure 1 顯示的就是直接相依的情境，而如果我們可以透過一個 ILogger 的介面就可以反轉相依，  
使得購物車與 Logger 都相依與介面，  
現行主流的程式也都有提供對應的 DI Framework 或 Library，  
只要好好使用，這個原則並不難實踐了。

### 小記

說得有點太多了，以上的部份未來另外抽成相關的文章吧。

## 主文

{% youtube wuVy7rwkCfc %}

### DRY

DRY(Don't Repeat Yourself 的縮寫，不要重複你自已)，一個古老的軟體原則，  
在維基百科的定義如下:

> Every piece of knowledge must have a single,  
> unambiguous, authoritative representation within a system
> 每一項知識都必須在系統中具有單一，明確，權威的表示形式

儘管實務上比定義更為寬鬆，我仍然認為這個一個好習慣。  
使用 Copy/Paste 最大的問題是，當某處發現了一個錯誤，  
我會意識到其它複製貼上的地方也有相同的錯誤，並且需要被修正。

我曾經接手了一份代碼，有著大量重複代碼，為了修正一個 Bug 我需要修改 8 個地方!  
抽像化這部份的代碼成為一個隨處可呼叫 function 將有很大的幫助。

### WET

另一個概念是`WET(Write Everything Twice)`

> You can ask yourself "Haven't I written this before?"
> two times, but never three
>
> 你可以問自己 "我以前沒有寫過這個嗎？" 兩次，但從來沒有三次

在上面的例子中，有時候過度抽像比重複更有害。  
那是一份 AngularJS 代碼與一些 AngularJS controller，  
這份代碼傳遞 `this` 給一個猴子補丁(Monkey Patch)方法(註:一種在runtime時動態修改代碼的技巧)，  
這個方法會用某種方式提供 controller 額外的能力。  
但是這超級讓人困惑，而讓人害怕修改。

這個代碼在三個以上的地方重複使用，但是抽象性很差，我寧可代碼重複。

### AHA

> Avoid Hasty Abstractions
> 避免草率抽象
> prefer duplication over the wrong abstraction
> 寧可重複也不要錯誤抽像
> Optimize for change first
> 為改變而優化

關鍵是我們不會知道代碼的未來。  
我們無法確認，我們能確認的只有改變可能會發生，  
如果沒有發生，為什麼要修改代碼 ?

注意以下事實:

- 不要介意重複，直到你對重複代碼的用例有信心為止
- 代碼有差異的部份，是否可以抽出參數提供給你的方法
- 逐步執行的過程，可以抽象共用的部份將會躍來而上
- 不用擔心何時該對重複代碼進行抽象，直到你接觸到他

### 結論

不用擔心何時該對重複代碼進行抽象，  
這不是一個硬性規定，也是為什麼我更喜歡 AHA 勝於 DRY 或 WET.  

## 相關文章

- [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)
- [Code Duplicate](https://en.wikipedia.org/wiki/Duplicate_code)
- [WET](https://dev.to/wuz/stop-trying-to-be-so-dry-instead-write-everything-twice-wet-5g33)
- [五則皆變](http://teddy-chen-tw.blogspot.com/2014/04/solid.html)
- [[閱讀筆記] 物件導向的心得與隨筆](https://blog.marsen.me/2020/07/27/2020/oop_experience/)
- [Kent Beck 的四個簡單程式設計原則](https://ihower.tw/blog/archives/7181)
- [九大設計原則](https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/410455/)
- [Stop trying to be so DRY, instead Write Everything Twice (WET)](https://dev.to/wuz/stop-trying-to-be-so-dry-instead-write-everything-twice-wet-5g33)

(fin)
