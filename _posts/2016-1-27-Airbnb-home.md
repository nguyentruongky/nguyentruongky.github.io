---
layout: post
title:  Airbnb Home screen sample
subtitle: Simple Airbnb Home screen to demonstrate Collection View nested in a Table View
date:   2016-1-27
background: '/img/posts/code_2.jpg'
---
![demo.gif](https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fdemo.gif?alt=media&token=cad709ea-eb96-498c-8e0d-5af864263ef5)

You can see it anywhere. It's very popular in home screen, gallery screen. See how to do that.
-   Create new project.
-   Remove ViewController.swift and the view controller in Main storyboard.
-   Add TableViewController to storyboard. Don't forget check `Is Initial View Controller`
-   Name the Table View Cell and Identifier `HeaderCell`
-   Set HeaderCell height to 400
-   Design your cell look like this

![headerCellDesign.png](https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fheadercelldesign.png?alt=media&token=a2de60ef-625a-4864-bb73-d25f86d7ef54)

-   Add new file `HeaderCell.swift` and connect with our HeaderCell. Remember, `HeaderCell.swift` is subclass of UITableViewCell
-   Clear all codes in file and connect outlets.

![connectOutletHeaderCell.png](https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fconnectoutletheadercell.png?alt=media&token=ccc8ff44-0995-4699-a395-06d0af605458)

-   Write a method to make the search button as a circle. Create a new file name `UIExtension.swift`. You see how it's useful later.
```
extension UIView {    
	func createBorder(color: UIColor = .white, width: CGFloat = 2) {
		self.layer.borderColor = color.CGColor
		self.layer.borderWidth = width
	}
    
	func createRoundCorner(radius: CGFloat = 4) {
		self.layer.cornerRadius = radius
		self.clipsToBounds = true
	}
    
	func createRoundCorner() {
		createRoundCorner(4)
	}
    
	func createBorderCorner() {
		createBorder()
		createRoundCorner()
	}
    
	func createCircleShape() {
		createRoundCorner(self.frame.size.width / 2)
	}
}    
```
If you don't know about Extension yet, read more [here](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Extensions.html). It's awesome.

-   Back to HeaderCell, add `setupView()` method.
```
func setupView() {
	searchButton.createCircleShape()
}    
```
    
-   Add a subclass of UITableViewController file name `HomescreenViewController.swift`, connect it to your Table View Controller in IB and clear all codes in files.
-   Add some code
```
class HomescreenTableViewController: UITableViewController {    
	// It's the header of the sections. The first empty section header is use for header.
	let sections = ["", "Recently View", "Favourite", "Suggestion"]  

	override func numberOfSectionsInTableView(tableView: UITableView) -> Int { return 1 }
    
    override func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
		return sections.count }
	
	override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
		let cell = tableView.dequeueReusableCellWithIdentifier("HeaderCell", forIndexPath: indexPath) as! HeaderCell
		cell.setupView()
		return cell
	}
    
	override func tableView(tableView: UITableView, heightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat { return 400 }
}
```
-   Run app and you see header cell.

![homescreenHeaderComplete](https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fhomescreenheadercomplete.png?alt=media&token=fcc1ec80-71cf-4224-982c-9cef26b3f5f0)

-   Now is the main design of the screen. Add new Table View Cell, name and Reuse identifier is `PlaceCategory`
-   Change cell height to 400.
-   Add a Collection View to your cell, set delegate and datasource to your `PlaceCategory` cell.
-   Select the Collection View and set cell size to width: 300, height: 350. Set scroll direction to Horizontal.
-   Select the Collection View Cell and set reuse identifier is `PlaceCell`
-   Design the PlaceCell like the UI.

![placeCategory](https://truongky.files.wordpress.com/2016/01/placecategory.png)

-   Add new file (subclass of UITableViewCell), name `PlaceCategoryCell.swift`. Connect it to your PlaceCategoryCell and connect outlets to your file.
-   Add new file (subclass of UICollectionViewCell), name `PlaceCollectionViewCell`. Connect it to your PlaceCell. Connect outlets to your file.

![placeCollectionViewCell_connectOutlet](https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fplacecollectionviewcell_connectoutlet.png?alt=media&token=244b013a-577a-4439-9474-2d49fd256aee)

-   Add `setupView()` method to your code
```
func setupView() {  
	avatarImageView.createCircleShape()
}    
```
-   Back to `PlaceCategoryCell.swift` and add some codes.
    
```
// Model
class PlaceModel : NSObject {    
	var placeName = ""
	var shortDescription = ""
	var price: Int64 = 0
	var currency = ""
	var unit = ""
	var thumbnailImage: UIImage?
	var avatarImage: UIImage?
	var didWish = false
}
    
class PlaceCategoryTableViewCell: UITableViewCell, UICollectionViewDelegateFlowLayout, UICollectionViewDataSource {
	@IBOutlet weak var sectionLabel: UILabel!
	var placeList = [PlaceModel]()
	
	func collectionView(collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int { return placeList.count }

	func collectionView(collectionView: UICollectionView, cellForItemAtIndexPath indexPath: NSIndexPath) -> UICollectionViewCell {
		let cell = collectionView.dequeueReusableCellWithReuseIdentifier("PlaceCell", forIndexPath: indexPath) as! PlaceCollectionViewCell
		cell.setupView()
		let place = placeList[indexPath.row]
		cell.thumbnailImageView.image = place.thumbnailImage
		cell.avatarImageView.image = place.avatarImage
		cell.priceLabel.text = String(place.price)
		cell.currencyLabel.text = place.currency
		cell.unitLabel.text = place.unit
		cell.placeNameLabel.text = place.placeName
		cell.placeDescriptionLabel.text = place.shortDescription
		return cell
	}
    
	func collectionView(collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAtIndexPath indexPath: NSIndexPath) -> CGSize {
		return CGSize(width: 300, height: 350) }
}
```
    
-   It's almost done. Your UI setup is ready to display data. We create some sample data to show now.
-   Create new file `SampleData.swift` and add some codes
```
class SampleData {
	static let places = ["Ha Noi", "Hai Phong", "Trang An", "Hoi An", "Ho Chi Minh", "Bien Hoa", "My Tho", "Can Tho", "Tay Ninh", "Con Dao"]
	static let avatars = ["Avatar1", "Avatar2", "Avatar3", "Avatar4", "Avatar5", "Avatar6", "Avatar7", "Avatar8", "Avatar9", "Avatar10"]
	static let thumbnails = ["Place1", "Place2",  "Place3",  "Place4",  "Place5",  "Place6",  "Place7",  "Place8",  "Place9",  "Place10"]
	static let prices: [Int64] = [5000000, 15000000, 18000000,  2000000, 25000000, 15000000, 9000000, 1000000, 12000000, 6000000]  
	static let shortDescriptions = ["New BedRm by C.Park",
            "Cozy & Private Floor of Brownstone",
            "AMAZING MANHATTAN SKYLIN",
            "Studio Apartment with King Bed",
            "Beautiful Midtown East",
            "Brooklyn Style - With a Balcony!!!",
            "Cozy Bedroom in Apartment",
            "Loft with Manhattan skyline view!",
            "Hell's Kitchen 1-Bdr. Apt. Share",
            "Green Room"]
    
	static func generateFavouriteList() -> [PlaceModel] {
		var favouriteList = [PlaceModel]()
		for _ in 0..<10 {
			let randomIndex = randomInt(10)
			let place = PlaceModel()
			place.placeName = places[randomIndex]
			place.avatarImage = UIImage(named: avatars[randomIndex])
			place.thumbnailImage = UIImage(named: thumbnails[randomIndex])
			place.unit = "PER NIGHT"
			place.currency = "VND"
			place.price = prices[randomIndex]
			place.shortDescription = shortDescriptions[randomIndex]
			favouriteList.append(place)
		}
		return favouriteList
	}
    
	static func generateRecentlyList() -> [PlaceModel] {
		var recentlyList = [PlaceModel]()    
		for _ in 0..<10 {
			let randomIndex = randomInt(10)
			let place = PlaceModel()
			place.placeName = places[randomIndex]
			place.avatarImage = UIImage(named: avatars[randomIndex])
			place.thumbnailImage = UIImage(named: thumbnails[randomIndex])
			place.unit = "PER NIGHT"
			place.currency = "VND"
			place.price = prices[randomIndex]
			place.shortDescription = shortDescriptions[randomIndex]
			recentlyList.append(place)
		}
	    return recentlyList;
	}
    
	static func generateSuggestionList() -> [PlaceModel] {    
		var suggestionList = [PlaceModel]()
		for _ in 0..<10 {
			let randomIndex = randomInt(10)
			let place = PlaceModel()
			place.placeName = places[randomIndex]
			place.avatarImage = UIImage(named: avatars[randomIndex])
			place.thumbnailImage = UIImage(named: thumbnails[randomIndex])
			place.unit = "PER NIGHT"
			place.currency = "VND"
			place.price = prices[randomIndex]
			place.shortDescription = shortDescriptions[randomIndex]
			suggestionList.append(place)
		}
		return suggestionList
	}
    
	static func randomInt(max: UInt32) -> Int { return Int(arc4random_uniform(max)) }
}    
```
    
-   Please make sure you add 10 avatar images (Avatar1.png, Avatar2.png..., Avatar10.png) and 10 place images (Place1.png, Place2.png..., Place10.png)

Run and enjoy.

# Conclusion

The main purpose is to show how nest a Collection View in a Table View. The Airbnb home has many things to do. 
You can download my project [at my github](https://github.com/nguyentruongky/AirbnbHomescreenSample)