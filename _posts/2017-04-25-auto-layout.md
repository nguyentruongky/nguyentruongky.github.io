---
layout: post
title:  Auto Layout with Storyboard and Programmatically 
subtitle: My conclusion for argument about Auto Layout with Storyboard and Programmatically 
date:   2017-4-25
background: '/img/posts/storyboard-vs-programtically.jpg'
---
I started iOS development as a .NET developer. My first impression, Storyboard is really really interesting. 80% UI of my apps is built in Storyboard. I can't imagine how I can develop iOS without Storyboard. 

There are many discussions about Auto Layout with Storyboard or Programmatically is better. But no one wins. Everyone has own reason and belief. 

This post is my personal opinion. It's written by a developer can't live without Storyboard, and now he abandons it.

3 reasons why he changed to Auto Layout by programmatically.

## Why do I abandon Storyboard? 

### 1. Too many steps to changes
I designed a very good UI, and connected outlet, started logic coding, finished that feature. One day, I had to refactor code and found some improvement. Some `UILabel` had gestures, should be changed to `UIButton`. No problem, made changes. What did I do? 

>- Found the `UILabel` need change
- Delete one by one. 
- Add new `UIButton`
- Connect new outlets 
- Setup Auto Layout constraints. (This is most painful step, dozens of constraints need setup)
- Find and change many things in code

And when I changed to Auto Layout programmatically: 

>- Found the `UILabel` need change
- Change `UILabel` to `UIButton`
- Find and change many things in code

Easier a lot, right? Especially, no change in setting up constraints. 

### 2. Open Storyboard is a pain in the ass
My first project had a huge Storyboard, all-in-one. More than **30 Controllers** were in **1 Storyboard file**. It's a nightmare. Anytime I opened it, change the `UILabel`'s textColor, and committed. 99% had conflicts. Fix conflicts in XML is another nightmare. 

Thanks God, some experienced developers tell about split it up. A Storyboard should contain 3 to 5 Controllers, not more. It should be splitted up as features, for instance, `Membership` Storyboard contains `Landing`, `Register`, `Login` Controllers, or `Password` Controller contains `ForgotPassword`, `Confirmation`, `NewPassword` Controllers. It's a big improvement and save a lot of iOS developer's lives. 

It's better but not the best choice. Conflicts usually occur with Storyboard, and can't avoid. 
Besides that, constraints list is a mess, really mess and easy to make you mad. 

![](https://raw.githubusercontent.com/nguyentruongky/Photos_storage/master/Auto_Layout_Programmatically/Constraint_list.png)

You can debate about select a control, and select the constraint you want instead of find in the list. Believe me, many times you can't do that, and we soon get mad with it. 

### 3. Dynamically customize control with Auto Layout programmatically 



<img src="https://raw.githubusercontent.com/nguyentruongky/Photos_storage/master/Auto_Layout_Programmatically/Custom_Control.png" width="500">

What would you do with this control in Storyboard? You can make a custom view with 1 `UILabel` for title, 1 `UITextField` for text input, and 1 `UIView` for underline, 1 `UIButton` for right button. 

How to use it? Drag a `UIView` to Storyboard, choose subclass. Remember, it's a UIView. You have to access to the view instance, textField and change the properties.

But with Auto Layout programmatically, it's a  `UITextField`. You can change textColor,  font,  backgroundColor easily. Again, It's a `UITextField`.

You can see how I implement this custom `UITextField` at [knGistTextField](https://github.com/nguyentruongky/knCollection/blob/develop/knCollection/Control/knGistTextField.swift)

## Reasons prevent you from using Auto Layout programmatically

### Too many code in a file

Yes, I have to admit that. There are 600 - 700 lines of code in my file. But it's easy to solve this problem with **Extension**. 
Why don't we define and setup constraints in the main file, (`LoginController.swift`, for instance), and move all handler, logic into new file, `LoginControllerHandler.swift`. 

### Write the same codes many times

Define a `UIButton` with Auto Layout programmatically 
```
let button: UIButton = {
    let title = "Sample"
    let image = "Sample"
    let color = UIColor.black
    
    let button = UIButton()
    button.translatesAutoresizingMaskIntoConstraints = false
    button.setTitle(title, for: .normal)
    button.setTitleColor(color, for: .normal)
    button.setImage(UIImage(named: image), for: .normal)
    return button
}()
```

Every time I need a `UIButton`, do I have to find and copy this code? No, code snippet can help you. 

Find out how it helps [here](http://nshipster.com/xcode-snippets/).
I define above button with `knButton` key in my XCode. It's easier that find and drag a `UIButton` and then format it in Storyboard, right? 

### Code is too long and unreadable 
```
// (1): can't change anything 
let underline = UIView()
underline.translatesAutoresizingMaskIntoConstraints = false
addSubview(underline)

// (2): Constraints setup
underline.bottomAnchor.constraint(equalTo: bottomAnchor, constant: -8).isActive = true
underline.rightAnchor.constraint(equalTo: rightAnchor).isActive = true
underline.leftAnchor.constraint(equalTo: leftAnchor).isActive = true
underline.heightAnchor.constraint(equalToConstant: 1).isActive = true

// Format control
underline.backgroundColor = UIColor.lightGray
```

The (1) is same to all controls in Auto Layout programmatically, you have to keep this format for all controls. You can make it better with this code (if you want) 
```
extension UIView {
    func addToView(view: UIView) {
        translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(self)
    }
}
```

(1) can be changed to 
```
let underline = UIView()
underline.addToView(view)
```
The (2), they're default code from Apple, why don't we make it more readable and beautiful? 

```
underline.bottom(toView: view, space: -8)
underline.horizontal(toView: view)
underline.height(1)
```

Is it beautiful enough? It comes from my own Auto Layout library. You can find it at [https://github.com/nguyentruongky/knConstraints](https://github.com/nguyentruongky/knConstraints)


## What can I do with Auto Layout programmatically? 

If you wan to try a life without Storyboard, why don't you try these following instructions? 

- Setup your own code snippets 
- Try my Auto Layout library [**knConstraints**](https://github.com/nguyentruongky/knConstraints)
- Develop some simple custom views, with Auto Layout programmatically. 

## How can I help? 

Please feel free to touch to me by leaving comments below or Skype: nguyentruongky3390. 
