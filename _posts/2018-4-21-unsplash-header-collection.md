---
layout: post
title: Unsplash - Homescreen - Header and Collection Scroll
subtitle: TableHeaderView and Custom UICollectionView in Homescreen
date:   2018-4-21
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Funsplash.png?alt=media&token=b4e42f63-f873-4161-ab0d-1c865e971c9a'
---

I am making an app like Unsplash iOs to practise skills. This is the part 2 in the series. First part is <a href="/2018/04/20/unsplash-homescreen.html" target="_blank">here</a>.

### PhotoGroupView
- Add new file name `PhotoGroupView.swift`. 

```

    class PhotoGroupView: knView {
        var datasource = [PhotoGroup]() { didSet { collectionView.reloadData() }}
        lazy var collectionView: UICollectionView = { [weak self] in

            // (1)
            let layout = FAPaginationLayout()
            layout.scrollDirection = .horizontal
            let cv = UICollectionView(frame: .zero, collectionViewLayout: layout)
            cv.translatesAutoresizingMaskIntoConstraints = false
            cv.backgroundColor = .white
            // (2)
            cv.contentInset = UIEdgeInsets(top: 0, left: 16, bottom: 0, right: 16)
            cv.register(PhotoGroupCell.self, forCellWithReuseIdentifier: "PhotoGroupCell")
            cv.showsHorizontalScrollIndicator = false
            cv.showsVerticalScrollIndicator = false
            cv.delegate = self
            cv.dataSource = self
            return cv
        }()
        
        override func setupView() {
            addSubviews(views: collectionView)
            collectionView.fill(toView: self)
        }
    }

    extension PhotoGroupView: UICollectionViewDelegate, UICollectionViewDataSource, UICollectionViewDelegateFlowLayout {
        func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int { return datasource.count }
        
        func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
            let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "PhotoGroupCell", for: indexPath) as! PhotoGroupCell
            cell.data = datasource[indexPath.row]
            return cell
        }
        
        func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, minimumLineSpacingForSectionAt section: Int) -> CGFloat { return 8 }
        
        // (3)
        func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
            var cellSize: CGSize = collectionView.bounds.size
            cellSize.width -= collectionView.contentInset.left
            cellSize.width -= collectionView.contentInset.right
            return cellSize
        }
    }

```
(1): `FAPaginationLayout` is a layout to center the item. It's good library I used in some recently projects. You can see it on [github](https://github.com/fahidattique55/FAPaginationLayout).

(2): ContentInset left and right are 16 to make a space 16px when scroll to the first and last items. 

(3): calculate the item size, item stretches all the horizontal without the left and right inset like (2)

- Add new file name `PhotoGroupCell.swift`

```

    class PhotoGroupCell: knCollectionCell {
        var data: PhotoGroup? {
            didSet {
                titleLabel.text = data?.name
                imageView.downloadImage(from: data?.url)
            }}
        
        let imageView = knUIMaker.makeImageView(contentMode: .scaleAspectFill)
        let titleLabel = knUIMaker.makeLabel(font: UIFont.systemFont(ofSize: 16), color: .white, numberOfLines: 3, alignment: .center)
        
        override func setupView() {
            addSubviews(views: imageView, titleLabel)
            
            imageView.fill(toView: self)
            titleLabel.horizontal(toView: self, space: 8)
            titleLabel.centerY(toView: self)
            
            createRoundCorner(7)
        }
    }

```

- An error about missing `PhotoGroup`. Add new file name `PhotoGroupModel.swift`

```

    struct PhotoGroup {
        var name: String
        var url: String
        var id: String        
    }

```

- One more step. Add some code to class `HomeController` 

```

    // (1)
    let categoryView = PhotoGroupView()

    // (2)
    lazy var photoGroupCell = self.makePhotoGroupCell()
    func makePhotoGroupCell() -> knTableCell {
        let font = UIFont.boldSystemFont(ofSize: 16)
        let explore = knUIMaker.makeLabel(text: "Explore", font: font, color: .black)
        let new = knUIMaker.makeLabel(text: "New", font: font, color: .black)
        
        let cell = knTableCell()
        cell.addSubviews(views: categoryView, explore, new)
        cell.addConstraints(withFormat: "V:|-12-[v0]-8-[v1]-12-[v2]-8-|", views: explore, categoryView, new)
        
        categoryView.horizontal(toView: cell)
        explore.left(toView: cell, space: 16)
        new.left(toView: explore)
        
        categoryView.height(150)
        return cell
    }

```

(1): You can add `UICollectionView` inside `UITableViewCell` instead of `UIView` as what I am doing. It is shorter way, but can't reuse in other project. This kind of design is very popular, maybe we need in other projects, so I make a custom view, instead of a custom cell. 

(2): Define `photoGroupCell` is a `lazy var` property. 

> Lazy properties are useful when the initial value for a property is dependent on outside factors whose values are not known until after an instance’s initialization is complete. Lazy properties are also useful when the initial value for a property requires complex or computationally expensive setup that should not be performed unless or until it is needed. (from [Apple Developer Document](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Properties.html))

I usually use this way to define properties need to complex actions to initialize. This way can help to manage to code easier, you are able to fold/unfold codes, move initialization method to extension or somewhere. 

- Update `UITableViewDatasource`: Datasource for homescreen is a Photo array, so we can't put an extra Photo into that array. Need a trick here. 

```

    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int { 
        return datasource.count + 1 }

    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        
        if indexPath.row == 0 { return photoGroupCell }
        
        let cell = tableView.dequeueReusableCell(withIdentifier: "PhotoCell", for: indexPath) as! PhotoCell
        cell.data = datasource[indexPath.row - 1] // (*)
        return cell
    }

```

We will return one more row in the table, and cell for this row is `photoGroupCell` created before. 

(*) is very important. Without minus 1, app crashes when scroll to the final item. Example will be easier to understand. 

You have 10 Photo instances in array `datasource`, but have 11 rows to display. So the final row will get item at index 10 in `datasource`. That's `Fatal error: Index out of range`

- Run and see. 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Fempty_collection.png?alt=media&token=76cf0e38-5a1e-44c0-a7f6-b21464849af4" width="40%"/>

Empty space in top. It's work. But no data setup, so it's empty. Add test data into `fetchData` method

```

    var categories = [PhotoGroup]()
    for _ in 0 ..< 5 {
        categories.append(PhotoGroup(name: "Sport", url: "https://unsplash.com/photos/zydhjnjppEc/download", id: "12bawr"))
        categories.append(PhotoGroup(name: "Computer", url: "https://unsplash.com/photos/Y6N_w94x8ik/download", id: "nvljx91"))
    }
    categoryView.datasource = categories

```

Run again and see

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Fdata_collection.png?alt=media&token=32ba9c94-e280-4569-a2ab-0540f77ae774" width="40%"/>

### HeaderView

- Add new file `HeaderView.swift`

```

    class HeaderView: knView {
    
        var data: Photo? {didSet {
            guard let data = data else { return }
            authorLabel.text = "photo by " + data.author
            imageView.downloadImage(from: data.url)
            }}
        
        let imageView = knUIMaker.makeImageView(contentMode: .scaleAspectFill)
        let authorLabel = knUIMaker.makeLabel(font: UIFont.systemFont(ofSize: 16), color: .white, alignment: .center)
        let titleLabel = knUIMaker.makeLabel(text: "Photos for everyone", font: UIFont.boldSystemFont(ofSize: 32), color: .white, alignment: .center)
        let searchTextField = knUIMaker.makeTextField(placeholder: "Search photos")
        
        override func setupView() {
            // (1)
            translatesAutoresizingMaskIntoConstraints = true
            
            searchTextField.createRoundCorner(8)
            searchTextField.setLeftViewWithImage(#imageLiteral(resourceName: "search"))
            searchTextField.height(44)
            searchTextField.changePlaceholderTextColor(.white)
            addBlur(to: searchTextField, size: CGSize(width: screenWidth - 16 * 2, height: 44))
            
            addSubviews(views: imageView, titleLabel, searchTextField, authorLabel)
            
            imageView.fill(toView: self)
            
            titleLabel.centerY(toView: self, space: -48)
            titleLabel.centerX(toView: self)
            
            searchTextField.horizontal(toView: self, space: 16)
            searchTextField.verticalSpacing(toView: titleLabel, space: 16)
            
            authorLabel.centerX(toView: self)
            authorLabel.bottom(toView: self, space: -16)
        }
        
        func addBlur(to view: UIView, size: CGSize) {
            let blurEffect = UIBlurEffect(style: UIBlurEffectStyle.dark)
            let blurEffectView = UIVisualEffectView(effect: blurEffect)
            blurEffectView.frame = CGRect(x: 0, y: 0, width: size.width, height: size.height)
            blurEffectView.autoresizingMask = [.flexibleWidth, .flexibleHeight]
            view.addSubview(blurEffectView)
        }
    }

```

(1): Need to set `translatesAutoresizingMaskIntoConstraints` to true because in knView, it's set to false by default. `translatesAutoresizingMaskIntoConstraints` is used for auto layout by code. But here, we will set frame for headerView. 

The search textfield in Unsplash is very nice. It has a blur background. That's why I add blur to it by `addBlur(to:size)`

- Back to class `HomeController` and add new view `let headerView = HeaderView()`

- Open `setupView` method, add these codes at before method close bracket

```

    // (1)
    let headerHeight: CGFloat = 350
    headerView.frame = CGRect(x: 0, y: 0, width: screenWidth, height: headerHeight)
    tableView.tableHeaderView = headerView

```

(1): We don't need headerView automatically adjust height, so set a fixed height for it. Without a height, table view can't render properly.

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Fstatus_bar.png?alt=media&token=967b3402-7b86-4e81-b23e-5b535445bbf3" width="40%"/>

- It's better when the photo fill the status bar. Just one line in `setupView`

```

    tableView.contentInset = UIEdgeInsets(top: -20, left: 0, bottom: 0, right: 0)

```

- Wait, the status bar is black and the header is black, too. It's bad. Change the status bar to light content

Right click on `Info.plist` and `Open As\Source Code`, paste this the before `</dict>`

```

    <key>UIViewControllerBasedStatusBarAppearance</key>
    <false/>

```

Back to `HomeController`, `setupView` method, add one more line 

```

    statusBarStyle = .lightContent

```

`statusBarStyle` is my custom property, you can see it more detail by press `control + command` and click on it. 

- Run again and see

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Funsplash%2Ffinal_part_2.png?alt=media&token=93cbf937-735a-4705-b64d-ac14be383f72" width="40%"/>

<hr/>

That's all for today. I will make animation next note. 