---
layout: post
title: Customize your UITabBarController
subtitle: How to make your UITabBarController be more attractive? 
date:   2018-4-15
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2FtabController%2FTabBar_2x.png?alt=media&token=62fb2b53-552d-47be-ac56-7a0680ac1e1b'
---
From 2017, design trending has changed to tab bar, instead of slide menu. `UITabBarController` has become one of the most popular controller. It's very simple to use. But sometime, it's too simple to customize and make attractive. Designers are artists, and they usually want tab bar like this 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2FtabController%2Fogenii.png?alt=media&token=033df77c-7adb-40a7-aca9-ad921efb4629" width="100%"/>

or like this 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2FtabController%2Fdoctor.png?alt=media&token=e5a38fff-eddf-4863-b2bb-b4e15fdef4d2" width="100%"/>


I had to implement a tab bar like above in 2015. My implementation at that time was bad. I used a `UIViewController` as a main controller, and added a custom view like a tab bar. And everytime a tab selected, embed the other `UIViewController` into the main controller. That's not good option.

Today I make a `UITabBarController` with a custom view like a tab bar. With this solution, we can use the max strength of `UITabBarController` with the same behavior and code. Much easier and better than my implementation before. Hope anyone can easily customize to their design if they want after this note. 

## knTabBarItem
First thing need to be customized is the `UITabBarItem`. `UITabBarItem` is not flexible enough to be customized to anything I want. I need to use `UIButton`. 

```
class knTabBarItem: UIButton {
    // (1)
    var itemHeight: CGFloat = 0
    // (2)
    var lock = false
    // (3)
    var color: UIColor = UIColor.lightGray {
        didSet {
            guard lock == false else { return }
            iconImageView.change(color: color)
            textLabel.textColor = color
        }}
    
    // (4)
    private let iconImageView = knUIMaker.makeImageView(contentMode: .scaleAspectFit)
    private let textLabel = knUIMaker.makeLabel(font: UIFont.systemFont(ofSize: 11),
                                        color: .black, alignment: .center)
    
    convenience init(icon: UIImage, title: String,
                     font: UIFont = UIFont.systemFont(ofSize: 11)) {
        self.init()
        translatesAutoresizingMaskIntoConstraints = false
        iconImageView.image = icon
        textLabel.text = title
        textLabel.font = UIFont(name: font.fontName, size: 11)
        setupView()
    }
    
    // (5)
    private func setupView() {
        addSubviews(views: iconImageView, textLabel)
        iconImageView.top(toView: self, space: 4)
        iconImageView.centerX(toView: self)
        iconImageView.square()
        
        let iconBottomConstant: CGFloat = textLabel.text == "" ? -2 : -20
        iconImageView.bottom(toView: self, space: iconBottomConstant)
        
        textLabel.bottom(toView: self, space: -2)
        textLabel.centerX(toView: self)
    }
}
```

(1): Some tab items can be bigger than others. I can easily set the height for them to make difference with others. 

(2) (3): Same to (1), some tab items are very unacceptional with different color and don't change color when selected.  

(4): knUIMaker is my collection to make controls. Just easier to make UIButton, UIImageView, UILabel by code. 

(5): `addSubviews`, `top`, `centerX`, `bottom` are from my [knConstraints](https://github.com/nguyentruongky/knConstraints) to make auto layout. I'm a fan of auto layout programmatically, so make controls and set layouts by code is what to do hundreds of times everyday. 

## knTabBar
Next thing I have to focus on is `UITabBar`. 
```
class knTabBar: UITabBar {
    // (1)
    var kn_items = [knTabBarItem]()
    convenience init(items: [knTabBarItem]) {
        self.init()
        kn_items = items
        translatesAutoresizingMaskIntoConstraints = false
        setupView()
    }
    
    override var tintColor: UIColor! {
        didSet {
            for item in kn_items {
                item.color = tintColor
            }}}
    
    func setupView() {
        backgroundColor = .white
        if kn_items.count == 0 { return }
        
        // (2)
        let line = knUIMaker.makeLine(color: .gray, height: 0.5)
        addSubviews(views: line)
        line.horizontal(toView: self)
        line.top(toView: self)
        
        // (3)
        var horizontalConstraints = "H:|"
        let itemWidth: CGFloat = screenWidth / CGFloat(kn_items.count)
        for i in 0 ..< kn_items.count {
            let item = kn_items[i]
            addSubviews(views: item)
            if item.itemHeight == 0 {
                item.vertical(toView: self)
            }
            else {
                item.bottom(toView: self)
                item.height(item.itemHeight)
            }
            item.width(itemWidth)
            horizontalConstraints += String(format: "[v%d]", i)
            if item.lock == false {
                item.color = tintColor
            }
        }
        
        horizontalConstraints += "|"
        addConstraints(withFormat: horizontalConstraints, arrayOf: kn_items)
    }
}
```
(1): I can't override `items` in `UITabBar`, so I name it a little bit similar to easier to remember

(2): Add a line to separate the tab bar to the controller. Some designs need indicator at the selected item, I will add indicator here. 

(3): Flexible to add items by programmatically. Thanks Apple for Auto Layout Programmatically. 

## knTabController
The easiest thing is here. Just inherit from `UITabBarController`, add some code, and it works. 

```
class knTabController: UITabBarController {
    var kn_tabBar: knTabBar!
    var selectedColor = UIColor.darkGray
    var normalColor = UIColor.lightGray {
        didSet {
            kn_tabBar.tintColor = normalColor
        }}
    
    private var kn_tabBarHeight: CGFloat = 49
    override func viewDidLoad() {
        super.viewDidLoad()
        tabBar.isHidden = true
        setupView()
    }

    func setupView() {}
    
    private func setTabBar(items: [knTabBarItem], height: CGFloat = 49) {
        guard items.count > 0 else { return }
        
        kn_tabBar = knTabBar(items: items)
        guard let bar = kn_tabBar else { return }
        kn_tabBar.tintColor = normalColor
        bar.kn_items.first?.color = selectedColor
        
        view.addSubviews(views: bar)
        bar.horizontal(toView: view)
        bar.bottom(toView: view)
        kn_tabBarHeight = height
        bar.height(kn_tabBarHeight)
        for i in 0 ..< items.count {
            items[i].tag = i
            items[i].addTarget(self, action: #selector(switchTab))
        }
    }
    
    @objc func switchTab(button: UIButton) {
        selectedIndex = button.tag
    }
}
```

That's ready for new tab bar. Just use it in a controller. 

## How to use? 
- Inherit class `knTabController` to your controller. 
- Override `setupView` method. 
```
class DoctorController: knTabController {
    override func setupView() {
        let home = knTabBarItem(icon: #imageLiteral(resourceName: "home"), title: "Home")
        let appointment = knTabBarItem(icon: #imageLiteral(resourceName: "appointment"), title: "Appointment")
        let add = knTabBarItem(icon: #imageLiteral(resourceName: "add"), title: "")
        add.lock = true
        add.itemHeight = 66
        let doctors = knTabBarItem(icon: #imageLiteral(resourceName: "doctors"), title: "Doctors")
        let porfolio = knTabBarItem(icon: #imageLiteral(resourceName: "user"), title: "Porfolio")

        let red = UIViewController()
        red.view.backgroundColor = .red
        let green = UIViewController()
        green.view.backgroundColor = .green
        let blue = UIViewController()
        blue.view.backgroundColor = .white
        let yellow = UIViewController()
        yellow.view.backgroundColor = .yellow
        let gray = UIViewController()
        gray.view.backgroundColor = .gray

        setTabBar(items: [home, appointment, add, doctors, porfolio])
        viewControllers = [red, green, blue, yellow, gray]
        normalColor = .red
    }
}
```
- Run and see. The main button (Hexagon Add) is locked, don't change the color when selected. 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2FtabController%2Fdoctor.gif?alt=media&token=27df3f84-5d39-499a-9205-43eaeea87515" width="40%"/>

But it's better if we have animation, looks much better. 

## Add animation 
- In `knTabController`, change method `switchTab` content to 

```
    let newIndex = button.tag
    changeTab(from: selectedIndex, to: newIndex)
```
- Add method `changeTab`

```
private func changeTab(from fromIndex: Int, to toIndex: Int) {
    kn_tabBar.kn_items[fromIndex].color = normalColor
    kn_tabBar.kn_items[toIndex].color = selectedColor
    animateSliding(from: fromIndex, to: toIndex)
}
```

- And result: 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2FtabController%2Fdoctor_slide.gif?alt=media&token=2b1a56d8-aacf-4d51-924e-a44d6827a99d" width="40%"/>

## Conclusion 
Hope this can help anyone want to customize a tab bar can do it effortless. Code is [here](https://github.com/nguyentruongky/knTabController).

I will add some more animations and some properties to make it more convenience in very near future. Suggestions and feedbacks are welcome. 

### Notes
I am a fan of Auto Layout Programmatically, so usually use my library, **knConstraints** for setting constraints. **knConstraints** is a very simple way to setup Auto Layout with very easy to read syntax. You can try it yourself [here](https://github.com/nguyentruongky/knConstraints).