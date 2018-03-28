---
layout: post
title:  How to draw a gradient border button? 
subtitle: Gradient is becoming popular. It's easy to make a full gradient button. But how to make a gradient border button? 
date:   2017-4-13
background: '/img/posts/gradient_button.jpg'
---

## My Problem 

Last week, my partner showed me his design for our application. Everything is great, easily implemented with some custom controls. But wait, something is not in my knowledge. 

![](https://gist.github.com/nguyentruongky/89020a5f495420b260952f2ff19c4147/raw/df4fc4447088c33ed934e8f4fb56cdc6da1b6a55/res_Gradient_border_button.png)

A button with gradient border. Never try it before. Up to now, I just created gradient background views 2 times in previous projects. Googled and found some good results. 

## Other solutions 
```
let button = UIButton(frame: CGRect(x: 100, y: 100, width: 200, height: 100))
let gradient = CAGradientLayer()
gradient.frame =  CGRect(origin: .zero, size: button.frame.size)
gradient.colors = [UIColor.blue.cgColor, UIColor.green.cgColor]
let shape = CAShapeLayer()
shape.lineWidth = 2
shape.path = UIBezierPath(rect: button.bounds).cgPath
shape.strokeColor = UIColor.black.cgColor
shape.fillColor = UIColor.clear.cgColor
gradient.mask = shape
button.layer.addSublayer(gradient)
```
Awesome, it's perfect. 

![](https://gist.github.com/nguyentruongky/89020a5f495420b260952f2ff19c4147/raw/df4fc4447088c33ed934e8f4fb56cdc6da1b6a55/res_Gradient_border_rectangle.png)

> Rectangles with rounded corners are everywhere! (Steve Jobs)

Tried to make my button rounded corner. And... 

![](https://gist.github.com/nguyentruongky/89020a5f495420b260952f2ff19c4147/raw/df4fc4447088c33ed934e8f4fb56cdc6da1b6a55/res_Wrong_round_corner_gradient_rectangle.png)

Oops. It didn't work as what I need. Did some more search and found another great one from Ian Hirschfeld here [https://medium.com/swift-programming/how-to-create-an-angle-gradient-border-in-swift-f4856dde4c90](https://medium.com/swift-programming/how-to-create-an-angle-gradient-border-in-swift-f4856dde4c90)

## My solution 

It works like a charm, but it's not my favorite solution. I need a Swifty project, not a combination with Objective-C. So don't I try to make something simpler and easier for me and others. Finally, I did it. 

### The idea 

Simple idea, create a button with gradient background, fill it with a solid color view, and the padding to the button bounds is the border width. 

> Talk is cheap. Show me the code. (Linus Torvalds)

```
func setupView() {    
    let gradientLayer = CAGradientLayer()
    gradientLayer.frame = bounds
    gradientLayer.colors = colors.map({ return $0.cgColor })
    gradientLayer.startPoint = startPoint
    gradientLayer.endPoint = endPoint
    layer.insertSublayer(gradientLayer, at: 0) // * important
    
    let backgroundView = UIView()
    backgroundView.translatesAutoresizingMaskIntoConstraints = false
    insertSubview(backgroundView, at: 1) // * (1)
    backgroundView.backgroundColor = backgroundColor // (2)
    backgroundView.fill(toView: self, space: UIEdgeInsets(space: borderWidth)) // (3)
    createRoundCorner(cornerRadius)
}
```

(1) - The most important thing here is the order of the gradient layer and background view. The title label will be overlapped and when we use `addSublayer` or `addSubview`. 

(2) - backgroundColor: the view property from UIKit. 

(3) - borderWidth is new property, we will use it every time re-setupView. 

Now is time to create the rounded corner. 

```
func createRoundCorner(_ radius: CGFloat) {
    cornerRadius = radius
    super.createRoundCorner(radius)
    backgroundView.createRoundCorner(radius - borderWidth)
}
```

Every time we set the radius or backgroundColor, `setupView` called. So that, the `gradientLayer` and `backgroundView` are added many times. Keep an instance and remove it every time view setup. 

But, no gradient border appears. The border width still zero. We need to give a non-zero value to `borderWidth` and `setupView` again. 

```
func createBorder(_ width: CGFloat) {
    borderWidth = width
    setupView()
}
```
See the result 

![](https://gist.github.com/nguyentruongky/89020a5f495420b260952f2ff19c4147/raw/df4fc4447088c33ed934e8f4fb56cdc6da1b6a55/res_Final_button.png)

### Completed code 

Have a look at my completed code for this library at [https://github.com/nguyentruongky/knGradientBorderButton](https://github.com/nguyentruongky/knGradientBorderButton)

I use my auto layout library in this project. You can find it here: [https://github.com/nguyentruongky/knConstraints](https://github.com/nguyentruongky/knConstraints)

## Suggestions or feedback?

Feel free to comment, suggest, create pull request or open issue. 