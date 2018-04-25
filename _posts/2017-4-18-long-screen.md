---
layout: post
title:  Use UITableViewController to solve very long screen 
subtitle: This note is very useful for apps in market place category, like shopping, real estate, ... 
date:   2017-4-18
background: '/img/posts/long_screen.jpg'
---

![](https://raw.githubusercontent.com/nguyentruongky/Photos_storage/master/UITableViewController_static/Long_screen.jpg)

Do you have any solutions for long long screen like this? I'm sure have. This kind of screen is very popular in item detail (e-commercial app), room view (booking app)... I have to implement this screen in my class detail from an education app. This is how I did that. 

## My problem 
- The screen is too long, and too many controls are in every section. 
- Some sections can be missing, for instance, promotion code section is only available for specially class. 
- Some sections contain very complex controls, for instance, UICollectionView the editor's picks in the above photo. 

## Solution 

First I thought about the UIStackView. It can hide or show view easily. But hard to control the section's size and have to manage scrolling. 

I find my best solution, UITableViewController, static UITableView. 

## The idea
- A section is a cell in the TableView. 
- I make some custom views for complex section, like an embedded UICollectionView. 
- Other simple sections, like Summary, Promotion, Shipping can be define controls inside the controller. 
- Develop some functions to dynamically generate cells. Don't define variables for cells. 

## Sample code 

```

    func makeAvailibilityCell() -> knTableCell {
        let cell = knTableCell()

        let containerView = UIView()
        containerView.translatesAutoresizingMaskIntoConstraints = false
        cell.addSubview(containerView)
        containerView.fill(toView: cell)
        
        let titleLabel = makeHeaderLabel(text: "availability".classLocalized) // (1)
        
        let seeButton = ogeSupporter.createGradientBorderButton() // (2)
        seeButton.setTitle("seeAvailability".classLocalized, for: .normal)
        
        containerView.addSubview(titleLabel)
        containerView.addSubview(availabilityLabel) // (3)
        containerView.addSubview(seeButton)
        
        // (4)
        seeButton.width(160)
        seeButton.right(toView: containerView, space: -30)
        seeButton.centerY(toView: titleLabel)
        
        titleLabel.left(toView: containerView, space: 30)
        titleLabel.width(screenWidth - 60)
        
        availabilityLabel.horizontal(toView: titleLabel)
        
        containerView.addConstraints(withFormat: "V:|-15-[v0]-10-[v1]|",
                                    views: titleLabel, availabilityLabel)
        
        return cell
    }

```

(1) (2) - Just define a label/button. Nothing is important here. These controls are not changed text/UI, should be defined inside the functions to reduce the number of reference controls in the controllers 

(3) - This is the content of this section. Data changes up to different items. I need to keep this reference to update data when request completed. 

(4) - Set up the UI with auto layout. 

**Important: ** The space between 2 sections should be define on 1 cell. Look at the `Promotion code` and `Is that a gift?` in the photo. The padding between them should belong to `Is that a gift?` cell. 

It means, the `bottomAnchor` of `Apply button` in `Promotion code` to the cell is 0. And the `topAnchor` of `Is that a gift? label` is about 54. It's very useful in missing section. The space among sections still are good enough. 

#### How to use 

```

    cells.append(makeAboutSellerCell()) 
    cells.append(makeAddressCell())
            
    if promotion.isAvailable == true {
        cells.append(makePromotionCell())
    }
    tableView.reloadTable()
    
```

## Conclusion 

It's easy, right? It saves much time for me. If designer wants to change anything, go to exactly make cell function, change the UI.

Enjoy coding. 

I use my auto layout library in this project. You can find it here: [https://github.com/nguyentruongky/knConstraints](https://github.com/nguyentruongky/knConstraints)
