---
layout: post
title: Unsplash iOs - Homescreen - Drag and Drop in UITableView 
subtitle: Animation to drag and drop from UITableView to UIView
date:   2018-4-28
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Funsplash.png?alt=media&token=b4e42f63-f873-4161-ab0d-1c865e971c9a'
---
## Whole series 

I am making an app like Unsplash iOs to practise skills. This is the part 3 in the series.

- Part 1: <a href="/2018/04/20/unsplash-homescreen.html" target="_blank">Note</a> <a href="https://drive.google.com/open?id=1eJWCPYe0mrLj30yUhHixL3kKe1vXNipX" target="_blank">Source</a>

- Part 2: <a href="/2018/04/23/unsplash-header-collection.html" target="_blank">Note</a> <a href="https://drive.google.com/open?id=17nrIA8hjXWOiofVlAmR-G1aL-xx65vvY" target="_blank">Source</a>

- Part 3: <a href="/2018/04/26/unsplash-home-animation.html" target="_blank">Note</a> <a href="https://drive.google.com/open?id=1BWS51BVmJ0HwpDxTv3AXhGoINOPEvyOY" target="_blank">Source</a>

- **Part 4**: <a href="/2018/04/28/drap-drop-tableview-animation.html" target="_blank">Note</a> <a href="https://drive.google.com/open?id=1sPk9e72ToR0VFbaK1IFFPpHHrQb4zOpX" target="_blank">Source</a>

- Part 5: <a href="/2018/04/30/transition-to-detail.html" target="_blank">Note</a> <a href="https://drive.google.com/open?id=1pTvT1MDiIhdQbvu2PshwEgyrNPnmc0fL" target="_blank">Source</a>


## Introduction
We add animation in previous part (Part 3) to animate table header view when scrolling and zooming. Today, we will download photo when long pressing at a photo and moving to the download zone. These action needs a series of animations. Not too easy but not too hard. We can do it. 

## Preparation
You need to install the Unsplash app in [Appstore](https://itunes.apple.com/us/app/unsplash/id1290631746?mt=8) and try to understand the download behaviour from the original app. 

## Implementation 
### Add long press gesture recognizer 

- Open class `PhotoCell` and add `UILongPressGestureRecognizer` in `setupView` method

```

    let longGesture = UILongPressGestureRecognizer(target: self, action: #selector(detectLongPress))
    addGestureRecognizer(longGesture)

```

- Add method `detectLongPress`

```

    @objc func detectLongPress(gesture: UILongPressGestureRecognizer) {
        longGestureAction?(gesture)
    }

```

- Add property `longGestureAction`. This property will be passed from `HomeController`. 

```

    var longGestureAction: ((_ gesture: UILongPressGestureRecognizer) -> Void)?

```

### Add gesture handler in HomeController

- Add code to tableView(cellForRowAt)

```

    cell.longGestureAction = { [weak self] recognizer in self?.didLongPressCell(recognizer: recognizer)}

```

- Add method 

```

    @objc func didLongPressCell (recognizer: UILongPressGestureRecognizer) {
        animator.recognizer = recognizer
        animator.detectDownloading()
    }

```

Compiler tells it doesn't `animator`. No worries, leave it there. We get back to it later. 

### Add animator 

- Add new file `Animator.swift` and class `Animator`

- We need some properties to handle these animations

```

    var view = UIView()
    lazy var dropZone = self.makeDropZone()
    var dragImageView: UIImageView?
    var startPoint = CGPoint.zero
    var recognizer: UILongPressGestureRecognizer?

```

- Add method `detectDownloading`

```

    func detectDownloading() {
        guard let recognizer = recognizer, let imageView = recognizer.view?.viewWithTag(1001) as? UIImageView else { return }
        
        switch recognizer.state {
        case .began:
            dragImageView = cloneImageView(from: imageView)
            view.addSubviews(views: dragImageView!)
            dragImageView!.frame = changeFrameToView(from: imageView)
            startPoint = locationInView(from: recognizer)
            zoom(view: dragImageView!, to: 1.02)
        case .changed:
            guard let dragImageView = dragImageView else { return }
            let newPoint = locationInView(from: recognizer)
            if newPoint == startPoint { return }
            zoom(view: dragImageView, to: 0.5)
            dragImageView.center = newPoint
            animateDropZone(visible: true)
            view.bringSubview(toFront: dragImageView)
            
            if checkContain(bigView: dragImageView, smallView: dropZone) {
                zoomDropZone(bigger: true)
            }
            else {
                zoomDropZone(bigger: false)
            }
            
        case .ended:
            guard let dragImageView = dragImageView else { return }
            if (checkContain(bigView: dragImageView, smallView: dropZone)) {
                print("Downloading")
                animateImageWhenDownload()
                animateDropZone(visible: false)
            } else {
                print("Cancel download")
                let originalFrame = changeFrameToView(from: imageView)
                animateImageWhenCancel(to: originalFrame)
                animateDropZone(visible: false)
            }
        default:
            return
        }
    }

```

This is an important method. I will go slowly here and hope it is easy to understand for you. 

```
    guard let recognizer = recognizer, let imageView = recognizer.view?.viewWithTag(1001) as? UIImageView else { return }

```

`recognizer` is passed from `HomeController`. We need UIImageView from the view in `UIGestureRecognizer`. We added gesture to the `PhotoCell`, so the `recognizer.view` is `PhotoCell`. We have 2 options: 

1. Open `photoImageView` in `PhotoCell` to public and we can access by `(recognizer?.view as? PhotoCell)?.photoImageView`

2. Add a tag 1001 to `photoImageView` in `PhotoCell` and access by `viewWithTag` like this `recognizer.view?.viewWithTag(1001) as? UIImageView`

I follow the second option, because I don't want to open photoImageView to public. Keep everything in private if I can. 

Next, the switch case of `UIGestureRecognizerState`. We need to focus on the following state 

```

    case .began:
        // (1)
        dragImageView = cloneImageView(from: imageView)
        view.addSubviews(views: dragImageView!)

        // (2)
        dragImageView!.frame = changeFrameToView(from: imageView)

        // (3)
        startPoint = locationInView(from: recognizer)

        // (4)
        zoom(view: dragImageView!, to: 1.02)

```

(1): We need to clone the `photoImageView` to the `dragImageView` in the selected `PhotoCell` and add it to the main view. Why do we have to clone it? Without this action, we will move the `photoImageView` in `PhotoCell` around, and the cell will have a blank space for this imageView. 

You can see the implementation of `cloneImageView`, `changeFrameToView`, `locationInView` methods in 
 <a href="https://gist.github.com/nguyentruongky/a484459983ee52bf11a750d682e6dd9b" target="_blank">this file</a>.

(2): Why do we need to `changeFrameToView`? The `dragImageView` frame currently is in the UITableView, because it belongs to a cell. We need to move it out of the tableView. Changing frame to main view is neccessary. 

(3): When `UIGestureRecognizerState` is `.began`, it calls `.changed` also and calls several times. We have to save `startPoint` to prevent it state `.changed` called unpredictably. 

(4): A slight animation zoom in make user focus on what happen. Only zoom in to 1.02 or 1.04 is great, not larger. 

<hr/>

```

    case .changed:
        guard let dragImageView = dragImageView else { return }

        // (1)
        let newPoint = locationInView(from: recognizer)
        if newPoint == startPoint { return }

        // (2)
        zoom(view: dragImageView, to: 0.5)

        // (3)
        dragImageView.center = newPoint

        // (4)
        animateDropZone(visible: true)

        // (5)
        if checkContain(bigView: dragImageView, smallView: dropZone) {
            zoomDropZone(bigger: true)
        }
        else {
            zoomDropZone(bigger: false)
        }

        // (6)
        view.bringSubview(toFront: dragImageView)

```

The state `.changed` when we move the finger around. The selected photo needs to be moved around while moving. 

(1): This is how we prevent unpreditable action in `.begin` state. It in the same location but don't know why it fires `.changed` state. 

(2): OK, animation here. Zoom out the photo to smaller while moving. You can zoom out to 0.3 - 0.6. But when the `dragImageView` is in the dropZone, the `downloadZone` will zoom in a little bit, and zoom out when it is out. That's what (5) do. 

(3): Move the image view around with your finger. Please take note here, in the `cloneImageView` method, we have to change `translatesAutoresizingMaskIntoConstraints` to true. `translatesAutoresizingMaskIntoConstraints` was false because it is setup by auto layout, but now we have to change frame and animate by frame. 

(4): The original app has a download zone at the bottom right. It zooms in to appear while moving. The implementation is in the file. I write detail about download zone later. 

(6): We need to bring th `dragImagView` to the top unless it is overlaped by the download zone. 

<hr/>

```

    case .ended:
        guard let dragImageView = dragImageView else { return }
        if (checkContain(bigView: dragImageView, smallView: dropZone)) {
            if let image = dragImageView.image {
                UIImageWriteToSavedPhotosAlbum(image, nil, nil, nil)
            }
            animateImageWhenDownload()
            animateDropZone(visible: false)
        } else {
            print("Cancel download")
            let originalFrame = changeFrameToView(from: imageView)
            animateImageWhenCancel(to: originalFrame)
            animateDropZone(visible: false)
        }

```

State `.end` is fired when you lift your finger. If your lift point is in dropZone, the photo will be downloaded and saved into your Photos Library. And do nothing when it's out the zone. 

### Request permisson to save photo 

Without request permisson, the app will be crashed, and you can see the mesage like this. 

>This app has crashed because it attempted to access privacy-sensitive data without a usage description.  The app's Info.plist must contain an NSPhotoLibraryAddUsageDescription key with a string value explaining to the user how the app uses this data.

Very easy to fix. Just open `Info.plist` as Source code and add lines before `</dict>`

```

    <key>NSPhotoLibraryAddUsageDescription</key>
    <string>Save photo from Unplash to your Photos Library</string>

```

### Add animator to HomeController

- Back to `HomeController`, add new property `let animator = Animator()`

- In `setupView`, add these code to the end of the method 

```

    animator.view = view
    view.addSubviews(views: animator.dropZone)

```

- Replace method `didLongPressCell`

```

    @objc func didLongPressCell(recognizer: UILongPressGestureRecognizer) {
        animator.recognizer = recognizer
        animator.detectDownloading()
    }

```

Run and see

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Fdownload_animation.gif?alt=media&token=4770cc2f-6e58-4ee3-bb9c-a7ad378341d3" width="40%"/>

### DownloadZone

I write a little bit about how I make the download zone view and how to animated it, not too detail, just the idea to do. 

The download zone view is created and animated by frame, not auto layout. Same to the label in side the view. When make the view, we will set its frame out of the main frame, so it's not visible in the view. And will update frame when appear and zoom. If we just update the view only, the label inside will be moved unwanted. So we need to update frame to both download view and the label inside. 

## Conclusion 

There are many things we can improve in this code. I will do that at the end of the project. You can do it yourself and drop me text in github or skype. We can discuss more about this. Love to hear from you. 

PS: After this note, I found the Unsplash app only support for iOs 11, so maybe the team is using Drag and Drop in iOs 11 only. You can try it yourself. 

