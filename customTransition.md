# ИСТОРИЯ ВЬЮ-КОНТРОЛЛЕРА, КОТОРЫЙ ХОТЕЛ ПОКАЗЫВАТЬСЯ КРАСИВО

Жил был скромный вью-контроллер *VCYellow*. И не было у него ни картинки, ни текста, ни даже малюсенькой бизнес логики. Жил он обычной вью-контроллерской жизнью.
<cut />
Его товарищ вью-контроллер *VCMain* иногда презентовал его миру:

```swift
class VCMain: UIViewController {
...
@IBAction func onBtnTapMeTapped(_ sender: Any) {
    let vcYellow = self.storyboard!.instantiateViewController(withIdentifier: "VCYellow") as! VCYellow
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

![](https://habrastorage.org/webt/wk/tl/b4/wktlb4m-hcmjrf8_qpvycz9p0eg.gif)

Но была у нашего героя мечта научиться показываться и скрываться по-красоте. Да так, чтобы можно было эту красоту менять потом по праздникам или просто в честь хорошего настроения.

![](https://habrastorage.org/webt/ev/c7/oz/evc7ozq2dgznnwbm0h9mry7brko.gif)

Шли года... и так и осталась бы мечта мечтой, если бы не узнал *VCYellow* о магии под названием:
```
UIViewControllerTransitioningDelegate
```
А сила этой магии в том, что даёт она возможность подсунуть соответствующий аниматор для показа и для скрытия вью-контроллера. Как раз то, о чём мечтал наш контроллер.
Прочитал он в [древних свитках](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioningdelegate) как использовать заклятие и начал готовиться.
Записал себе шпаргалку с самим заклинанием, чтобы не забыть:

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
В ней он тщательно расписал, что для показа нужно использовать аниматор *AnimatorPresent*, а при закрытии *AnimatorDismiss*.
Ну и в качестве помощи обоим аниматорам было решено передать фрейм главной кнопки из *VCMain*

А потом и сам морально настроился. Потому как без правильного настроя, как известно, никакая магия не работает:
```swift
override func viewDidLoad() {
    super.viewDidLoad()
    self.modalPresentationStyle = .custom
    self.transitioningDelegate = self
}
```
Попросил он своего друга *VCMain* презентануть себя, чтобы проверить как магия сработает и… сработала она никак…
Оказалось, что AnimatorPresent и AnimatorDismiss сами собой не появляются.

Останавливаться было уже поздно и наш герой решил создать необходимые аниматоры. Поковырялся в нужном разделе [древних свитков](https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning) и узнал, что для создания аниматора достаточно двух вещей.

Во-первых надо задать время, отведённое для анимации:

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

    UIView.animate(withDuration: 0.3, animations: {
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

1) вытащить презентуемый вью-контроллер(в нашем случае VCYellow) и сфоткать его. Фотка нужна для упрощения анимации.

2) Получить вьюшку, на которой будет происходить анимационное колдунство.
Назовем её контекст.

3) Нацепить вьюху конечного контроллера на контекст и скрыть её. Показать
её было решено после того как закончится анимация.

4) Подготовить фотку для анимации. Уменьшить до начальных размеров и кинуть на контекст.

5) Расщеперить фотку на весь экран, тем самым анимировав процесс презентации.

6) После окончания анимации показать настоящую вьюху конечного контроллера,
избавиться от фотки и сообщить, что действо окончено.

В результате вышел вот такой аниматор для показа:

```swift
import UIKit

class AnimatorPresent: NSObject, UIViewControllerAnimatedTransitioning {
    let startFrame: CGRect
    
    init(startFrame: CGRect) {
        self.startFrame = startFrame
    }

    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return 0.3
    }

    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        guard let vcTo = transitionContext.viewController(forKey: .to),
        let snapshot = vcTo.view.snapshotView(afterScreenUpdates: true) else {
            return
        }

        let vContainer = transitionContext.containerView

        vcTo.view.isHidden = true
        vContainer.addSubview(vcTo.view)

        snapshot.frame = self.startFrame
        vContainer.addSubview(snapshot)

        UIView.animate(withDuration: 0.3, animations: {
            snapshot.frame = (transitionContext.finalFrame(for: vcTo))
        }, completion: { success in
            vcTo.view.isHidden = false
            snapshot.removeFromSuperview()
            transitionContext.completeTransition(true)
        })
    }
}
```

А после этого несложно было написать аниматор для скрывания, который делает примерно то же самое, но наоборот:

```swift
import UIKit

class AnimatorDismiss: NSObject, UIViewControllerAnimatedTransitioning {

    let endFrame: CGRect

    init(endFrame: CGRect) {
        self.endFrame = endFrame
    }

    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return 0.3
    }

    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        guard let vcTo = transitionContext.viewController(forKey: .to),
        let vcFrom = transitionContext.viewController(forKey: .from),
        let snapshot = vcFrom.view.snapshotView(afterScreenUpdates: true) else {
            return
        }

        let vContainer = transitionContext.containerView
        vContainer.addSubview(vcTo.view)
        vContainer.addSubview(snapshot)

        vcFrom.view.isHidden = true

        UIView.animate(withDuration: 0.3, animations: {
            snapshot.frame = self.endFrame
        }, completion: { success in
            transitionContext.completeTransition(true)
        })
    }
}
```

Закончив все доделки, *VCYellow* опять попросил своего друга *VCMain* презентовать себя и о чудо!

![](https://habrastorage.org/webt/ev/c7/oz/evc7ozq2dgznnwbm0h9mry7brko.gif)

Магия сработала! Мечта *VCYellow* сбылась! Теперь он может показываться и скрываться как ему захочется и ничто не будет ограничивать его фантазию!

Проект-пример можно скачать [тут](https://github.com/funkydevil/customTransition)

Статья, которую я использовал для вдохновения находится [тут](https://www.raywenderlich.com/322-custom-uiviewcontroller-transitions-getting-started)
