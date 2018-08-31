Жил был скромный вью контроллер *VCYellow*. И не было у него ни картинки, ни текста, ни даже малюсенькой бизнес логики. Жил он обычной вью-контроллероской жизнью. 
<cut />
Его товарищ вью-контроллер *VCMain* иногда презентовал его миру:

```swift
class VCMain: UIViewController {
...
	@IBAction func onBtnTapMeTapped(_ sender: Any) {
        let sb = UIStoryboard(name: "Main", bundle: nil)
        let vcYellow = sb.instantiateViewController(withIdentifier: "VCYellow") as! VCYellow
        self.present(vcYellow, animated: true, completion: nil)
    }
```
А *VCYellow* в свою очередь скрывался при помощи единственной кнопки "X", которой он, кстати говоря, очень гордился:

```swift
class VCYellow: UIViewController {
...
	@IBAction func onBtnCloseTapped(_ sender: Any) {
        self.dismiss(animated: true, completion: nil)
    }
```

И выглядело это не то чтобы плохо, но скучно и обыденно:

![](https://habrastorage.org/webt/fb/lf/op/fblfopvr4h0lp2dyltbjqhled64.gif)


Но была у нашего героя мечта научиться показываться и скрываться диковенным образом. Да таким, чтобы можно было эту красоту менять потом по праздникам или просто в честь хорошего настроения.

![](https://habrastorage.org/webt/ow/f1/jd/owf1jdk2uqqufbr_fzuntpwlovk.gif)


Шли года... и так и осталась бы мечта мечтой, если бы не узнал *VCYellow* о магии под названием: 
```
UIViewControllerTransitioningDelegate
```
А сила этой магии в том, что даёт она возможность подсунуть соответствующий аниматор для показа и для скрытия вью-контроллера. Как раз то, о чём мечтал наш контроллер.
Почитал он в [древних свитках](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioningdelegate) как использовать заклятие и начал готовиться.
Записал себе шпаргалку с самим заклинаниеми, чтобы не забыть:

```swift
extension VCYellow: UIViewControllerTransitioningDelegate {
    func animationController(forPresented presented: UIViewController, presenting: UIViewController, source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return AnimatorPresent(startFrame: self.startFrame)
    }
    
    func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return AnimatorDismiss(endFrame: self.startFrame)
    }
}
```
В ней он тщательно расписал, что при открытии нужно использовать аниматор *AnimatorPresent*, а при закрытии *AnimatorDismiss*.
Ну и в качестве помощи обоим аниматором было решено передать фрейм главной кнопки из *VCMain*



А потом и сам морально настроился. Потому как без правильного настроя, как известно, никакая магия не работает:
```swift
override func viewDidLoad() {
        super.viewDidLoad()
        
        self.modalPresentationStyle = .custom
        self.transitioningDelegate = self
    }
```
Попросил он своего друга VCMain презентануть себя ещё разок, что бы проверить как магия сработает и… сработала она никак…
Оказалось что AnimatorPresent и AnimatorDismiss сами собой не появляются, а следовательно магический сеанс красивого появляения не состоялся.

Останавливаться было уже поздно и наш герой решил создать недостающие аниматоры. Поковырялся в нужном разделе [древних свитков] (https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning) 
и узнал что надо во-первых задать время, отведённое на анимацию:

```swift
func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return 0.3
    }
```


а во-вторых обозначить саму анимацию:

```swift
func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        //1
		guard let vcTo = transitionContext.viewController(forKey: .to),
            let snapshot = vcTo.view.snapshotView(afterScreenUpdates: true) else {
            return
        }
        
		//2
        let vContainer = transitionContext.containerView
        
		//3
        vcTo.view.isHidden = true
        vContainer.addSubview(vcTo.view)
        
        //4
		snapshot.frame = self.startFrame
        vContainer.addSubview(snapshot)
        
        UIView.animate(withDuration: 0.3,
                       animations: {
			//5			   
            snapshot.frame = (transitionContext.finalFrame(for: vcTo))
        }, completion: { success in
			//6
			vcTo.view.isHidden = false
            snapshot.removeFromSuperview()
            transitionContext.completeTransition(true)
        })
    }
```

Тут в //1 



