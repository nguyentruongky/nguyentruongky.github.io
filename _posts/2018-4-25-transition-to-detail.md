---
layout: post
title: Unsplash iOs - Homescreen - Transition to view photo fullscreen
subtitle: Animation to open detail screen and go back. 
date:   2018-4-25
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Funsplash.png?alt=media&token=b4e42f63-f873-4161-ab0d-1c865e971c9a'
---
## Whole series 

I am making an app like Unsplash iOs to practise skills. This is the part 3 in the series.

- Part 1: <a href="/2018/04/20/unsplash-homescreen.html" target="_blank">Note</a> <a href="https://drive.google.com/open?id=1eJWCPYe0mrLj30yUhHixL3kKe1vXNipX" target="_blank">Source</a>

- Part 2: <a href="/2018/04/21/unsplash-header-collection.html" target="_blank">Note</a> <a href="https://drive.google.com/open?id=17nrIA8hjXWOiofVlAmR-G1aL-xx65vvY" target="_blank">Source</a>

- Part 3: <a href="/2018/04/22/unsplash-home-animation.html" target="_blank">Note</a> <a href="https://drive.google.com/open?id=1BWS51BVmJ0HwpDxTv3AXhGoINOPEvyOY" target="_blank">Source</a>

- Part 4: <a href="/2018/04/24/drap-drop-tableview-animation.html" target="_blank">Note</a> <a href="https://drive.google.com/open?id=1sPk9e72ToR0VFbaK1IFFPpHHrQb4zOpX" target="_blank">Source</a>

- **Part 5**: <a href="/2018/04/25/transition-to-detail.html" target="_blank">Note</a> <a href="https://drive.google.com/open?id=1pTvT1MDiIhdQbvu2PshwEgyrNPnmc0fL" target="_blank">Source</a>


## Introduction
In any photo apps, when select a photo, user can view fullscreen photo, double tap to zoom, and pinch zoom. It should have animation to make the transition look smooth and make sense. 

In my note, I only tell you how to make transition to view detail and go back to the list. Double tap to zoom you can read in the project. Pinch zoom you can easily google. 

## Implementation 

### Add method in HomeController

- Add `tableView(didSelectRowAt)` to fire transition and show detail controller

```

    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        // (1)
        if indexPath.row == 0 { return }
        let cell = tableView.cellForRow(at: indexPath)

        // (2)
        guard let imageView = cell?.viewWithTag(1001) as? UIImageView else { return }
        let newImageView = animator.cloneImageView(from: imageView)

        // (3)
        let originalFrame = animator.changeFrameToView(from: imageView)
        newImageView.frame = originalFrame

        // (4)
        view.addSubviews(views: newImageView)

        UIView.animate(withDuration: 0.35, animations: {
            // (5)
            newImageView.backgroundColor = .black
            newImageView.contentMode = .scaleAspectFit
            newImageView.frame = UIScreen.main.bounds
        }, completion: { _ in
            let controller = DetailController()
            
            // (6)
            controller.originalFrame = originalFrame
            controller.index = indexPath.row - 1

            // (7)
            controller.modalPresentationStyle = .overCurrentContext
            self.present(controller, animated: false)
            newImageView.removeFromSuperview()
        })
    }

```

(1): The first cell is PhotoGroupCell, we don't show DetailController while first cell tapped. 

(2): Get the `photoImageView` inside the selected cell, same to animation when long press to download. 

(3): We need to save `originalFrame` to go back at the same position. It's cool when you select a photo, it moves to a position, and you swipe down, the photo goes back to the previous position. 

(4): Add `newImageView` to view and we will animate this imageView. Same idea to download animation. 

(5): We use the strength of imageView to display an image fit the current size at the center but keep the aspect. The other space will be the backgroundColor.

(6): We paste the `originalFrame`, current index to `DetailController`. Current index will be used to display correct photo in `DetailController`. 

(7): We present `DetailController` with a transparent background. When close `DetailController`, we will have a cool animation here. 

### Add DetailController

- Add new file `DetailController.swift`

- Add properties to class `DetailController`, inherit from `knCollectionController`

```

    var datasource = [Photo]()
    var currentPhotoIndex = 0
    var originalFrame = CGRect.zero

```

- Wee need to show correct photo in the collection view, add these code 

```

    override func viewWillLayoutSubviews() {
        super.viewWillLayoutSubviews()
        collectionView.contentOffset = CGPoint(x: currentPhotoIndex * Int(screenWidth), y: 0)
    }

```

This controller is not too difficult, so that, I don't write too detail, just something important. 

- Add code to `setupView`

```

    override func setupView() {
        if let layout = collectionView.collectionViewLayout as? UICollectionViewFlowLayout {
            layout.scrollDirection = .horizontal
        }
        
        collectionView.backgroundColor = .black
        view.addSubviews(views: collectionView)
        collectionView.fill(toView: view)
        
        collectionView.isPagingEnabled = true
        collectionView.showsHorizontalScrollIndicator = false
        statusBarStyle = .lightContent

        view.backgroundColor = .clear
        view.isOpaque = false
        
        // add close button here
    }

```

I think `setupView` is clean enough to understand. 

- Add action to close detail controller 

```

    @objc func closeView() {
        guard let cell = collectionView.visibleCells.first as? DetailCell else { return }
        let imageView = cell.imageView
        let newImageView = cloneImageView(from: imageView)
        view.addSubviews(views: newImageView)
        newImageView.frame = view.frame
        collectionView.isHidden = true
        
        UIView.animate(withDuration: 0.4, animations: { [unowned self] in
            newImageView.frame = self.originalFrame
        }, completion: { [weak self] _ in
            self?.dismiss(animated: false) })
    }

```

Same idea before, we get the imageView inside the current cell, clone it and animate it. 

- It will be cooler when you swipe down and close the view. `UIPanGestureRecognizer` can help to do this. 

We need `DetailCell` before, but it's very simple so I don't write it down here. Just contain an imageView and some buttons. I didn't implement too detail about buttons, you can do it easily. 

Open `DetailCell`, add a property to keep in touch with `DetailController`

```

    var panAction: ((UIPanGestureRecognizer) -> Void)?

```

In `setupView`, add gesture to cell 

```

    let panGesture = UIPanGestureRecognizer(target: self, action: #selector(detectPan))
    panGesture.delegate = self
    addGestureRecognizer(panGesture)

```

And handle the gesture

```

    @objc func detectPan(recognizer: UIPanGestureRecognizer) {
        panAction?(recognizer)
    }

```

Back to `DetailController`, in `itemForRowAtIndex`, add code to connect to cell 

```

    cell.panAction = { [weak self] gesture in self?.detectPan(gesture: gesture) }

```

Hanlde the gesture 

```

    func detectPan(gesture: UIPanGestureRecognizer) {
        let touchPoint = gesture.location(in: view)
        
        switch gesture.state {
        case .began:
            initialTouchPoint = touchPoint
        case .ended, .cancelled:
            // (1)
            if touchPoint.y - initialTouchPoint.y > 100 &&
                abs(touchPoint.x - initialTouchPoint.x) < 5 {
                closeView()
            }
        default:
            break
        }
    }

```

(1): If you swipe down in a straight line, we will close the the controller. Without condition `abs(touchPoint.x - initialTouchPoint.x) < 5`, we can close controller while swipe left and right to see other photos. 

### It's run time

Run and see your result. 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Ftransition.gif?alt=media&token=5b23d44c-54be-499e-b474-ea02f3954667" width="40%"/>

Ah, I change datasource for both HomeController and DetailController. Up to now, we are manually manage the datasource with fake data. We will change that next part. 

## Conclusion 

I do not cover all the app. This is the end of this series. Learn a lot from Unsplash app. Unsplash team is very precise. App is simple but very cool. Love what the team doing. Thanks Unplash team. 