---
layout: post
title:  "PageViewController"
subtitle: "How to easily reuse UIPageViewController?"
date:   2016-1-12 
background: '/img/posts/page_view_controller.png'
---
I always get tired of rewriting (or copying) code from my previous projects every single time I use UIPageViewController. Finally, I found a way to reuse it anywhere by copying a file then write some small codes.  
Try it now.  
Follow [this article](http://swiftiostutorials.com/ios-tutorial-using-uipageviewcontroller-create-content-slider-objective-cswift/) to understand UIPageViewController.

OK, let's do some smart thing. Use your current project.

Create new file name `MyPageViewController.swift`. `MyPageViewController` will conform to `UIPageViewControllerDataSource`

Create an integer variable name `contentCount`

```
var contentCount = 0
```

Write a function to setup this controller

```
func setupController(numberOfContent: Int) {
    contentCount = numberOfContent

    // 1
    let pageController = UIPageViewController(transitionStyle: UIPageViewControllerTransitionStyle.Scroll, navigationOrientation: UIPageViewControllerNavigationOrientation.Horizontal, options: nil)
    pageController.dataSource = self
    
    // 2
    if contentCount > 0 {
        let firstController = getItemController(0)!
        let startingViewControllers: NSArray = [firstController]
        pageController.setViewControllers(startingViewControllers as? [UIViewController], direction: UIPageViewControllerNavigationDirection.Forward, animated: false, completion: nil)
    }

    // 3
    addChildViewController(pageController)
    self.view.addSubview(pageController.view)
}

```

(1): initialize a UIPageViewController. This is what show your content.

(2): setup the first page for controller

(3): add UIPageController to `MyPageViewController` subview.

Go ahead, implement `UIPageViewControllerDataSource`

```
func pageViewController(pageViewController: UIPageViewController, viewControllerBeforeViewController viewController: UIViewController) -> UIViewController? {
    let itemController = viewController as! PageItemController
    if itemController.itemIndex > 0 {
        return getItemController(itemController.itemIndex - 1)
    }
    return nil
}

func pageViewController(pageViewController: UIPageViewController, viewControllerAfterViewController viewController: UIViewController) -> UIViewController? {
    let itemController = viewController as! PageItemController
    if itemController.itemIndex + 1 < contentCount {
        return getItemController(itemController.itemIndex+1)
    }
    return nil
}

private func getItemController(itemIndex: Int) -> PageItemController? {
    if itemIndex < contentCount {
        return delegate?.getPageItemAtIndex(itemIndex) as? PageItemController
    }
    return nil
}

```

These method are defined in UIPageViewControllerDataSource and you have to implement it. You understand them after the tutorial on swiftiostutorials.com

You can see `PageItemController`. This is your Page Item Controller. If you don't want to change the base class, you can name your page item controller `PageItemController`. If not, you have to change `PageItemController` to another name.

XCode will ask you "What the hell is delegate?". So, what the hell is delegate?

Find out what it is [here](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/DelegatesandDataSources/DelegatesandDataSources.html). Now imagine that you are a shopkeeper but you can not ship products to your customer. Delivery man will help you. Delegate is a virtual delivery man. Delegate is a virtual delivery man. In your current controller (MyPageViewController), you can't do `getPageItemAtIndex`, so you tell the delegate do that for you.

Define delegate now.

```
protocol PageViewDelegate {
    func getPageItemAtIndex(index: Int) -> UIViewController
}
```

Write this code out of the class scope. I usually write it under import functions.

And use delegate in MyPageViewController
```
var delegate: PageViewDelegate?
```

It's almost done here. You have to implement PageViewDelegate in your main controller. However, I want to show you the complete code for this base class. Go ahead.

Add new variable to determine should we show indicator dots.

```
var didShowPageControl = false
```

Add these code at the end of `setupController` method.

```
if didShowPageControl == true {
    setupPageControl()
}
```

Add new methods
```
private func setupPageControl() {
    let appearance = UIPageControl.appearance()
    appearance.pageIndicatorTintColor = UIColor.grayColor()
    appearance.currentPageIndicatorTintColor = UIColor.whiteColor()
    appearance.backgroundColor = UIColor.darkGrayColor()
}

func presentationCountForPageViewController(pageViewController: UIPageViewController) -> Int {
    if didShowPageControl == false { return 0 }
    return contentCount
}

func presentationIndexForPageViewController(pageViewController: UIPageViewController) -> Int {
    return 0 }

```

We've done. You can copy it to anywhere you want and use it easily.

Design your beautiful page item to show on UIPageViewController. Here we will use the PageItemController you created in the swiftiostutorials's tutorial.

And in your main view controller, remove all codes in class ViewController then write some new codes. Your class looks like this:

```
class ViewController: UIViewController, PageViewDelegate {
    // Initialize it right away here
    private let contentImages = ["nature_pic_1.png",
                                 "nature_pic_2.png",
                                 "nature_pic_3.png",
                                 "nature_pic_4.png"];
    override func viewDidLoad() {
        super.viewDidLoad()
        let pageController = MyPageViewController()
        pageController.delegate = self
        pageController.setupController(contentImages.count)

        addChildViewController(pageController)
        self.view.addSubview(pageController.view)
    }

    func getPageItemAtIndex(index: Int) -> UIViewController {
        let pageItemController = UIStoryboard(name: "Main", bundle: nil).instantiateViewControllerWithIdentifier("ItemController") as! PageItemController
        pageItemController.itemIndex = index
        pageItemController.imageName = contentImages[index]
        return pageItemController
    }
}
```

You can see your `PageViewDelegate` you defined here.

What you have to do is, define an array of images, initilize a MyPageController instance and implement delegate.

Take a look at `getPageItemAtIndex`. What you want to show in PageItemController, you will code here. For example: you can add new label to show the item's description, define it and pass data here.

Run app and enjoy your code.

# Conclusion

Now we don't have to write the repeat code anymore. Copy this file and use it anywhere.

You can download the complete project [at my github](https://github.com/nguyentruongky/KPageViewController).