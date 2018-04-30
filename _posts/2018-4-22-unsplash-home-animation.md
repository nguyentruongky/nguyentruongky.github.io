---
layout: post
title: Unsplash iOs - Homescreen - Animation
subtitle: Animate the Table header view - scrolling and zooming
date:   2018-4-22
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Funsplash.png?alt=media&token=b4e42f63-f873-4161-ab0d-1c865e971c9a'
---

# Introduction
Animation is a must-have thing in iOs app. Animation makes app look cool and live. Everyone likes something animate than a sudden change. 

Unsplash does a great job here. A cool animation when scrolling makes app more interesting. 

Today I will make an animation like Unsplash, not 100% same, but not bad, I think :D. I don't record Unsplash animation, you should download the app and enjoy it. Really cool. Cool app worths to be downloaded and rated. You can download [here](https://itunes.apple.com/us/app/unsplash/id1290631746?mt=8).


## Whole series 

I am making an app like Unsplash iOs to practise skills. This is the part 3 in the series.

- Part 1: <a href="/2018/04/20/unsplash-homescreen.html" target="_blank">Note</a> <a href="https://drive.google.com/open?id=1eJWCPYe0mrLj30yUhHixL3kKe1vXNipX" target="_blank">Source</a>

- Part 2: <a href="/2018/04/21/unsplash-header-collection.html" target="_blank">Note</a> <a href="https://drive.google.com/open?id=17nrIA8hjXWOiofVlAmR-G1aL-xx65vvY" target="_blank">Source</a>

- **Part 3**: <a href="/2018/04/22/unsplash-home-animation.html" target="_blank">Note</a> <a href="https://drive.google.com/open?id=1BWS51BVmJ0HwpDxTv3AXhGoINOPEvyOY" target="_blank">Source</a>

- Part 4 <a href="/2018/04/24/drap-drop-tableview-animation.html" target="_blank">Note</a> <a href="https://drive.google.com/open?id=1sPk9e72ToR0VFbaK1IFFPpHHrQb4zOpX" target="_blank">Source</a>

## Let's do it

- In `HomeController`, add a new HeaderView

```

    let animatedHeader = HeaderView()

```

- Replace this code to `view.addSubviews(views: tableView)`

```

    view.addSubviews(views: animatedHeader, tableView)

    // (2)
    animatedHeader.translatesAutoresizingMaskIntoConstraints = false
    animatedHeader.horizontal(toView: view)
    // (3)
    animatedHeaderHeightConstraint = animatedHeader.height(headerHeight)
    animatedHeader.top(toView: view)

```

That means the `animatedHeader` is behind the tableView. 

(2) is very important. Do you know why? We need to setup auto layout for `animatedHeader` so that, we need to set `translatesAutoresizingMaskIntoConstraints` to false. 

(3): We need to keep the animatedHeader height constraint to animate it later. 

- `animatedHeader` and `headerView` need same data. Set the same data for `animatedHeader` in `fetchData`

```

    animatedHeader.data = headerView.data

```

- The keypoint is here. Add method `scrollViewDidScroll`

```

    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        // (1)
        let yOffset = scrollView.contentOffset.y
        // (2)
        let stopPoint: CGFloat = 96
        let newHeight = headerHeight - yOffset

        if newHeight > stopPoint {
            // (3)
            animatedHeaderHeightConstraint?.constant = newHeight
            // view.layoutIfNeeded()

            // (4)
            headerView.isHidden = true
        }
        else {
            headerView.isHidden = false
        }
    }

```

Move very slowly to understand clearly. 

(1): Animation will animate by the changing of `scrollView.contentOffset.y`. Most of animations in `UITableView` need this property. 

(2): Why `stopPoint` is 96? 96 is total of some paddings. First, the padding from the `searchTextField` to the `titleLabel` in `HeaderView`, it's 16px. Second, the padding from `searchTextField` to the bottom of the `HeaderView`, it should be equal to first padding, 16px. Third, the status bar height, 20px. Final, the `searchTextField` height, 44px. It's 16 + 16 + 20 + 44.
 
Stop point means when the `tableView` scroll (yOffset) to greater than `stopPoint`, animation will stop. 

Forgot to tell you. We need to move `let headerHeight: CGFloat = 350` from `setupView` to class. 

(3): Update the `animatedHeaderHeightConstraint`. iOs 9 needs call `view.layoutIfNeeded()` to update the constraints. 

(4): We have to hide the real `headerView` because the `animatedHeader` is behind the tableView, so we can't see anything if the real `headerView` is shown. 

Run the project. And you see white space at `headerView` portion. The `tableView.backgroundColor` is white, it has to be clear. Set `tableView.backgroundColor = .clear` in `setupView`.

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Fanimation_1.gif?alt=media&token=4ff5e4ba-72c1-4783-a294-5a79ba04c651" width="40%"/>

- The animation doesn't stop at stopPoint. Actually, it stops, but we can't see it. The `tableView` overlaps the `animatedHeader`. So we need to bring it to front. 

```

    func bringFakeViewToFontIf(isTop: Bool) {
        view.bringSubview(toFront: isTop ? tableView : animatedHeader)
    }

```

In `scrollViewDidScroll`, add code after line `let newHeight = headerHeight - yOffset`

```
   
    bringFakeViewToFontIf(isTop: yOffset == 0)

```

Run again and see

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Fanimation_2.gif?alt=media&token=ee7b5983-675e-49f0-a33e-ec9f857e2d78" width="40%"/>

Yeah, it stops. But you can see the problem, `titleLabel` and `authorLabel` are in wrong position. In Unsplash, they're fading out. Do the same thing. 

- Fade out `titleLabel` and `authorLabel`

```
    
    var opacity = (newHeight - stopPoint) / headerHeight
    opacity = opacity > 0 ? opacity : 0

```

Add these lines after `headerView.isHidden = true`

```

    animatedHeader.authorLabel.alpha = opacity
    animatedHeader.titleLabel.alpha = opacity

```

Run and see, it works. 

- Next, we need to change the background from an image to white. 

Open `HeadView.swift`, define new view 

```

    let whiteView = knUIMaker.makeView(background: .white)

```

Change `addSubviews(views: imageView, titleLabel, searchTextField, authorLabel)` to 

```

    addSubviews(views: imageView, whiteView, titleLabel, searchTextField, authorLabel)    
    whiteView.fill(toView: self)

```

Add new method 

```

    func animateWhiteView(by opacity: CGFloat) {
        let whiteViewOpacity = opacity > 0 ? opacity : 0
        whiteView.alpha = 1 - whiteViewOpacity
    }

```

Back to `HomeController`, add these before close bracket of `scrollViewDidScroll`

```

    let whiteViewOpacity = (newHeight * 2.2) / headerHeight
    animatedHeader.animateWhiteView(by: whiteViewOpacity)

```

Why `whiteViewOpacity` is `(newHeight * 2.2) / headerHeight`? The whiteView need to be white when scroll up upto the `categoryView` is overlapped, so it needs to be calculated with a higher value. `2.2` is just a guess and try value. I had to try several times before and found 2.2 is a good value for this purpose. 

We need to change the status bar style to default. 

```

    if newHeight > stopPoint {
        ...
        statusBarStyle = .lightContent
    }
    else {
        ...
        statusBarStyle = .default
    }

```

Run and see. 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Fanimation_3.gif?alt=media&token=80e8ae5b-1c62-4bd2-9cff-cb5b698b5f3d" width="40%"/>

- Almost there. We need to change the searchTextField backgroundColor, placeholder text color, the search icon color.

```

    func changeSearchTextFieldStyle(_ style: Style) {
        if style == .light {
            blurEffectView.removeFromSuperview()
            searchTextField.backgroundColor = UIColor.color(value: 230)
            let color = UIColor.color(value: 150)
            searchTextField.changePlaceholderTextColor(color)
            (searchTextField.leftView as? UIImageView)?.change(color: color)
        }
        else {
            addBlur(to: searchTextField, size: CGSize(width: screenWidth - 16 * 2, height: 44))
            let color = UIColor.white
            searchTextField.changePlaceholderTextColor(color)
            (searchTextField.leftView as? UIImageView)?.change(color: color)
        }
    }

```

```

    enum Style {
        case dark, light
    }

```

Move `blurEffectView` from `addBlur` method and init in the class `HeaderView`. 

```

    private let blurEffectView = UIVisualEffectView(effect: UIBlurEffect(style: UIBlurEffectStyle.dark))

```

Back to `HomeController`, in `scrollViewDidScroll` method 

```

    if newHeight > stopPoint {
        ...
        animatedHeader.changeSearchTextFieldStyle(.dark)
    }
    else {
        ...
        animatedHeader.changeSearchTextFieldStyle(.light)
    }

```
- Finally, animation is here. 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Fanimation_final.gif?alt=media&token=b71dd320-749d-41fd-ad8e-66444a46747a" width="40%"/>

<hr/>
That's some simple thing about animation in Unsplash. Simple and good. Love Unsplash team. 

