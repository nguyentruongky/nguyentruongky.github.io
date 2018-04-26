---
layout: post
title: Unsplash iOs - Homescreen - List 
subtitle: Make an iOs application like Unsplash
date:   2018-4-20
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Funsplash.png?alt=media&token=b4e42f63-f873-4161-ab0d-1c865e971c9a'
---
I found an excellent app, Unsplash. It's simple and good at UX. Some techniques are very interesting. I will implement a demo for this app. Just for practise my skills and hope this can help junior developers can enjoy new techniques. 

This note is about homescreen. I will make a basic homescreen with photos like Unsplash. Animation, category, load more photos are in other notes. 

## Preparation
I use some code collection from my personal codebase. You can download it at [github](https://github.com/nguyentruongky/codebase)

Init an empty project and added my `mustHave` folder

## Add TableView to HomeController
Change something

- File name: `ViewController.swift` to `HomeController.swift`
- Class name: `ViewController` to `HomeController`
- Main Storyboard custome class: `ViewController` to `HomeController`
- Class `HomeController` content is below. 

```

    // (1)
    class HomeController: knCustomTableController {
        override func viewDidLoad() {
            super.viewDidLoad()
            setupView()
        }
        override func setupView() {
            view.addSubview(tableView)
            tableView.fill(toView: view)
        }
        override func registerCells() {
            tableView.register(UITableViewCell.self, forCellReuseIdentifier: "UITableViewCell")
        }
    }

    // (2)
    extension HomeController {
        override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int { 
            return 5
        }
        
        override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: "UITableViewCell", for: indexPath) as! UITableViewCell
            cell.backgroundColor = UIColor.color(value: CGFloat(UInt32.random(lower: 0, upper: 255)))
            return cell
        }
        
        override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
            return 35
        }
    }

```

(1): `knCustomTableController` is my favourite controller in my codebase. It's a view controller, contain `UITableView`, conform `UITableViewDataSource` and `UITableViewDelegate`. 

Why don't I use `UITableViewController`? I have a note about this [here](https://kynguyen.space/2017/10/12/UITableViewController-vs-UIViewController.html). 

`knCustomTableController` is setup everything to be ready to display. You need override `registerCells`, some needed methods for table datasource like (2). 

The `setupView` method contains something weird. It's my constraints setting. I am a fan of Auto Layout Programmatically, so usually use my library, [**knConstraints**](https://github.com/nguyentruongky/knConstraints) for setting constraints.

Run and see a tableView with 5 rows, different colors. 

## Add Model and fake data 
- Add new file name `PhotoModel.swift`
- Add some code

```

    class Photo {
        var author: String
        var url: String
        var ratio: CGFloat = 1
        var displayHeight: CGFloat {
            return CGFloat(floor(Double(ratio * screenWidth)))
        }
        
        init(author: String, url: String, ratio: CGFloat) {
            self.author = author
            self.url = url
            self.ratio = ratio
        }
    }

```

- Update code in file `HomeController.swift`

```
    var datasource = [Photo]()

    override func registerCells() {
        tableView.register(PhotoCell.self, forCellReuseIdentifier: "PhotoCell")
    }

    override func fetchData() {
        for _ in 0 ..< 5 {
            datasource.append(Photo(author: "Kyle", url: "https://unsplash.com/photos/zydhjnjppEc/download", ratio: 0.667954600338083))
            datasource.append(Photo(author: "Mark", url: "https://unsplash.com/photos/Y6N_w94x8ik/download", ratio: 1.5))
        }
        tableView.reloadData()
    }

    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int { 
        return datasource.count }
        
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "PhotoCell", for: indexPath) as! PhotoCell
        cell.data = datasource[indexPath.row]
        return cell
    }

    override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat { 
        return UITableViewAutomaticDimension }

```

- Add `fetchData()` to `viewDidLoad`
- Can't run because of missing PhotoCell. Add new file `PhotoCell.swift`. Add content below to `PhotoCell.swift`

```

    import UIKit
    class PhotoCell: knTableCell {
        var heightConstraint: NSLayoutConstraint?
        var data: Photo? {
            didSet {
                guard let data = data else { return }
                authorLabel.text = data.author
                photoImageView.downloadImage(from: data.url)
                // (1)
                heightConstraint?.constant = data.displayHeight
            }
        }

        private let authorLabel = knUIMaker.makeLabel(color: .white)
        private let photoImageView = knUIMaker.makeImageView(contentMode: .scaleAspectFill)
        
        override func setupView() {
            let line = knUIMaker.makeLine(color: .white, height: 1)
            
            addSubviews(views: photoImageView, authorLabel, line)
            photoImageView.fill(toView: self)
            
            authorLabel.bottomLeft(toView: self, bottom: -8, left: 8)
            
            line.horizontal(toView: self)
            line.bottom(toView: self)
            
            // (2)
            heightConstraint = height(100, isActive: false)
            heightConstraint?.priority = UILayoutPriority(999)
            heightConstraint?.isActive = true
        }
    }
    
```

I always setup Auto Layout in `setupView` method. So, want to change any layout, UI, just find `setupView`.

(1): Every photo has different ratio, and we need to keep the photoImageView the same ratio. So every photo, update the height constraint (calculated when ratio set). 

(2): Need to descrease the priority to avoid conflict with `UIView-Encapsulated-Layout-Height`. `UIView-Encapsulated-Layout-Height` is a constraints automatically added by iOs. 

## Install pod 

Still have error at `photoImageView.downloadImage(from: data.url)`. You comment it before because missing `Kingfisher` library. We need to install `Kingfisher` to help us download image. It's very cool lib. More detail about Kingfisher is [here](https://github.com/onevcat/Kingfisher).

- Use Terminal to access to your project, and `pod init` then follow [this instruction](https://github.com/onevcat/Kingfisher/wiki/Installation-Guide). 

- Uncomment code in `knUIImage.swift` and run again

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Funsplash_photo_list_1.gif?alt=media&token=5e2786f0-e428-4771-8aee-879d6c3f2404" width="40%"/>