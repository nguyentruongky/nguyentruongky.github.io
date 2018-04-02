---
layout: post
title:  UITableViewController vs UIViewController + UITableView
subtitle: A small compare advantages and disadvantages in TableViewController
date:   2017-10-12
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Ftableview_compare_header.png?alt=media&token=f93ba5e6-1a9b-4b07-a75c-29bf6a14359c'
---
Have you ever used `UITableViewController`? Have you ever used `UIViewController and UITableView`? What are the differences between these two solutions? What are good practice for them? This note is a quick compare about UITableViewController and `UIViewController + UITableView`. I will call the `UIViewController + UITableView` is `Free TableView`

### UITableViewController 
`UITableViewController` is a UIViewController. Its view is a UITableView, can't change constraints. You can't add view into `UITableView`, without a cell. 
#### Advantages
It's good at handling text field become first responder. `UITableViewController` will automatically move contentOffset to the current active text field. 
Good practice for chat log screen, form screen. 
Accept static table view: very easy and fast to design form with Storyboard

<img
src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2FUITableViewController_demo.gif?alt=media&token=38ba3d36-a2e1-428d-9eb8-873811278776" width="240px"/>

#### Disadvantages
It's bad for state view (empty state, loading state, error state). Can't display state view by adding to view. You can do that by adding to tableHeaderView or tableFooterView, but you have to handle much things. 
It's bad for sticked top or bottom view. 

<img
src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Ffooter_sticked.jpeg?alt=media&token=9aa2b05e-3e08-43ba-9ebc-a617f0dce5ce" width="240px"/> <img
src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fheader_sticked.png?alt=media&token=33755b92-a998-4350-9f35-53623c0f6fce" width="240px"/>

Can't inherit from BaseController. You have a BaseController with many setting inside, such as, stateView, loadData, setupUI, etc... and you can't inherit that with `UITableViewController`.

### Free TableView
Free TableView is a `UIViewController` and add a `UITableView` into the view. You can easily set constraints for `UITableView`. 

#### Advantages
Very flexible. You can show/hide the tableView, set padding for tableView. 
Good for state display. In the demo, I show an empty state is a green UIView, at the center of the screen. 

<img
src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Ftableview_empty_state.gif?alt=media&token=0d79ce30-5071-4c93-a68f-046518cbf833" width="240px"/>

#### Disadvantages
Not accept static table view. I have a trick for this: use Dynamic table view, but datasource is a collection of `UITableCell`. 
But it will not automatically adjust when the textfield activate. We need lib to help us take care of it. I suggest this lib from Håkon Bogen (download at [Github](https://github.com/haaakon/SingleLineKeyboardResize/blob/master/UIViewController%2BKeyboard.swift)). Really good lib. 

<img
src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Ftableview_wrong_keyboard.gif?alt=media&token=242a50e8-db70-40f6-bab9-10bb5a303bdd" width="240px"/> <img
src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Ftableview_right_keyboard.gif?alt=media&token=5576301e-bcc6-46dd-a6c1-4b1784a00c35" width="240px"/>

### Explain for demo 
I have a demo for this. Download [here](https://github.com/nguyentruongky/TableViewController-vs-FreeTableView). Some code in demo I need to tell you. 
- `knController` is UIViewController. I made my own controller to manage building UI by code. 
- `knCustomTableController` is Free TableView
- Some functions like `horizontal:toView:space`, `fill:toView:space` are my Auto Layout Libs, called [knContrainsts](https://github.com/nguyentruongky/knConstraints). 
- Empty state in demo is just a green UIView. I don't want to add more code to make it more complicated. 

### Conclusion 
`Free TableView` has disadvantages but can be solved easily. This solution is much more flexible and easier to manage. But `UITableViewController` still has good practice which can't replace by `Free TableView`