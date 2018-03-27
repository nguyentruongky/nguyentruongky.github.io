---
layout: post
title:  Dynamically load view controller into another
subtitle: Embed a view controller to container view
date:   2016-1-29
background: '/img/posts/embed.jpg'
---
I saw a question on [stackoverflow](http://stackoverflow.com/questions/35011539/how-to-make-view-scroll-like-instagrams-profile-vc-with-segmentedcontroller/35079835#35079835). He ran into a problem when use ContainerView to embed other view controller. I'll share what I did.

-   Create new project.
-   Design what you want. I add a label, a segmented control and a container view to embed child controller.

![design](https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fcontainer_view.png?alt=media&token=1a69995c-9f5f-4226-a67c-b7bbd3bbce83)

-   Select all controls and choose menu Editor\\Embed In\\View. Name it: Content View.
-   Select Content View and embed into ScrollView.
-   Set constraint for ScrollView: leading, trailing, top, bottom to super view equal 0.
-   Set constraint for Content View: leading, trailing, top, bottom to ScrollView equal to 0, vertical and horizontal alignment.
-   Connect outlets and action to your controller.
-   Add your new controllers. In my demo, I added 2 TableViewController, 1 for displaying images (ImageController), 1 for loading text (TextController).
-   Back to your main view controller, add some code.
    
```
var dynamicHeight: CGFloat = 0    
@IBOutlet weak var scrollView: UIScrollView!  
@IBOutlet weak var pageContainer: UIView!
@IBAction func selectPage(sender: AnyObject) {
	// remove child controllers before add others
	for controller in childViewControllers {  
		controller.removeFromParentViewController()
	}  
	for view in pageContainer.subviews {  
		view.removeFromSuperview()
	}
	let segment = sender as! UISegmentedControl
	if segment.selectedSegmentIndex == 0 {  
		createImageController()
	} else { 
		createTextController()
	}  
	updateScrollContentSize()
}
    
func updateScrollContentSize() {
	// 150 is your other view height. 
	scrollView.contentSize = CGSize(width: view.frame.size.width, height: dynamicHeight + 150)
}
    
func createTextController() {    
	let textController = UIStoryboard(name: "Main", bundle: nil).instantiateViewControllerWithIdentifier("TextController") as! TextController
	dynamicHeight = textController.getTableContentHeight()
	textController.tableView.scrollEnabled = false  
	self.addChildViewController(textController)
	pageContainer.addSubview(textController.view)
	textController.view.frame = CGRectMake(0, 0, pageContainer.frame.size.width, dynamicHeight)
}

func createImageController() {    
	let imageController = UIStoryboard(name: "Main", bundle: nil).instantiateViewControllerWithIdentifier("ImageShowController") as! ImageShowController
	dynamicHeight = imageController.getTableContentHeight()
	imageController.tableView.scrollEnabled = false  
	self.addChildViewController(imageController)
	pageContainer.addSubview(imageController.view)
	imageController.view.frame = CGRectMake(0, 0, pageContainer.frame.size.width, dynamicHeight * 2)
}
    
override func viewDidAppear(animated: Bool) {    
	super.viewDidAppear(true)
	createImageController()
	updateScrollContentSize()
}
```
    
-   Xcode tells you about the error. Fix it now. Open your controller you want to embed and add some code.
    
```
func getTableContentHeight() -> CGFloat {    
	// calculate the total height of your controller. 
	return CGFloat(vietnamTravelLocations.count) * 44
}
```
    
It's done, run your project and see the result.
    
# Conclusion

Container view is very convenient, But don't exploit it, especially when your embed controller change the height many times. Your app will be lag, unstable and bring a bad experience to your user.

You can download my demo at [my github](https://github.com/nguyentruongky/DemoContainerViewDynamicLoading). Hope this can help.