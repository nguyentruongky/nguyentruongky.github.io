---
layout: post
title: Tasks iOs - Create task
subtitle: Make a cool todo app like Google Tasks
date:   2018-05-17
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Ftodo%2Ftodo.jpg?alt=media&token=58593177-6f60-45cd-b801-c0b73548fc95'
---

Keep moving in Google Tasks series today with Create Task. In this note, I will show you how to 

- Display a list of task 
- Add new tasks 
- Make some animations 

### Add UITableView to controller

- Open file `ViewController.swift` and change `ViewController` class to `tkTasksListController`.

- Change file `ViewController.swift` to `TaskList.swift`

- Open `AppDelegate` and clear all content of class `AppDelegate`. Paste these code to it 

```

    var window: UIWindow?
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        setupApp()
        return true
    }
    
    func setupApp() {
        window = UIWindow(frame: UIScreen.main.bounds)
        // (1)
        window!.rootViewController = tkTasksListController()
        window!.backgroundColor = UIColor.white
        window?.makeKeyAndVisible()
    }

```

I am a fan of auto layout by code, so I usually use this way to show the controller instead of Storyboard. You can replace `takTaskListController` to any controller you want. 

I will wrap the rootViewController to a UINavigationController, but not now. 

- Keep moving, open file `Info.plist`, and change data of line `Main storyboard file base name` to empty

- Run the app. App works well. We can safely remove `Main.storyboard`. 

- Back to class `tkTasksListController`. We change a little bit. Class now look like this 

```

    class tkTasksListController: knCustomTableController {
        var datasource = [tkTask]()

        override func setupView() {
            // (1)
            view.addSubviews(views: titleLabel, tableView, stateView, functionView)
            
            tableView.horizontal(toView: view)
            tableView.verticalSpacing(toView: titleLabel, space: 24)
            tableView.bottom(toAnchor: functionView.topAnchor, space: -24)
            ...
        }
        ...
    }

    extension tkTasksListController {
        // (2)
        override func tableView(_ tableView: UITableView,
                            numberOfRowsInSection section: Int) -> Int {
            return datasource.count }
        
        // (3)
        override func tableView(_ tableView: UITableView,
                                heightForRowAt indexPath: IndexPath) -> CGFloat {
            return UITableViewAutomaticDimension }
        
        // (4)
        override func tableView(_ tableView: UITableView,
                                cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: "tkTaskCell",
                                                    for: indexPath) as! tkTaskCell
            cell.data = datasource[indexPath.row]
            return cell
        }
    }

    // (5)
    override func registerCells() {
        tableView.register(tkTaskCell.self, forCellReuseIdentifier: "tkTaskCell")
    }

```

(1): Add `tableView` to view and set auto layout for it. `tableView` is stretched from the bottom of `titleLabel` to the top of `functionView`

(2), (3), (4): is tableView DataSource. It's basic for `UITableView`, you know it very well, I think. 

(5): We need to register the cell to reuse, without this step, `UITableView` doesn't know your cell and throw an exception. 

### Design Cell

`UITableView` needs `UITableCell`. Help it with new file name `TaskCell.swift` with content below 

```

    import UIKit

    // (1)
    class tkTask {
        var name: String?
        var finish = false
        
        init() {}
        init(name: String) {
            self.name = name
        }
    }

    // (2)
    class tkTaskCell: knTableCell {
        var data: tkTask? {
            didSet {
                guard let data = data else { return }
                nameLabel.text = data.name
            }
        }
        
        let doneButton = knUIMaker.makeButton()
        let nameLabel = knUIMaker.makeLabel(font: .systemFont(ofSize: 16),
                                            color: .black,
                                            numberOfLines: 0)
        
        override func setupView() {
            let line = knUIMaker.makeLine(color: UIColor.color(value: 230), height: 1)
            addSubviews(views: doneButton, nameLabel, line)
            
            doneButton.left(toView: self, space: 24)
            doneButton.centerY(toView: nameLabel)
            
            nameLabel.vertical(toView: self, space: 24)
            nameLabel.leftHorizontalSpacing(toView: doneButton, space: -24)
            nameLabel.right(toView: self, space: -24)
            
            line.horizontal(toView: self)
            line.bottom(toView: self)
            
            doneButton.createBorder(2, color: UIColor.color(value: 110))
            doneButton.square(edge: 24)
            doneButton.createRoundCorner(12)
        }
    }

```

(1): We need a model to keep the task content, this model is not as simple as this, it will expand to bigger soon. Keep it simple first. 

(2): Same to model, the cell is designed in the simplest way first. 

- Now we can run the app. But I am sure, you can't see the `tableView`. Add `tableView.backgroundColor = .green` to the last of `setupView` method. App will be like this 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Ftodo%2Ftask_tableview.png?alt=media&token=b309df22-4e15-4b05-850b-7e23ac9618db" width="40%"/>

- Problem found. We will hide the `tableView` show the `stateView` when there is no task. We can solve it by replace

```

    var datasource = [tkTask]()

```

to 

```
    
    var datasource = [tkTask]() { didSet {
        tableView.reloadData()
        let isEmpty = datasource.count > 0
        stateView.isHidden = isEmpty
        tableView.isHidden = !isEmpty
        }
    }

```

- Add dummy data

```

    override func fetchData() {
        datasource = [
            tkTask(name: "Hehe"),
            tkTask(name: "Aloha"),
        ]
    }

```

- Run and see. Don't forget to remove `tableView.backgroundColor = .green` in `setupView` method.

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Ftodo%2Ftask_tableview_data.png?alt=media&token=34ddc947-a628-4ce2-b8c5-08b453004571" width="40%"/>

### Add new task 

- Add these 2 lines to the end of the `setupView` method 

```

    newTaskView.saveButton.addTarget(self, action: #selector(addNewTask))
    newTaskView.textField.delegate = self

```

We can select save button or tap return key on the keyboard to save task. 

```

    // (1)
    @objc func addNewTask() {
        datasource.insert(tkTask(name: newTaskView.textField.text!), at: 0)
        newTaskView.textField.text = ""
        hideCreateTaskView()
    }

    // (2)
    extension tkTasksListController: UITextFieldDelegate {
        func textFieldShouldReturn(_ textField: UITextField) -> Bool {
            addNewTask()
            return true
        }
    }
```

(1): We will add new task directly to datasource, and `tableView` will reload to display new data. Same to the Google Task, we need to clear the textfield content and hide the `newTaskView`.

(2): Action for event return key pressed. 

The textfield is automatically correct my text, but I am using Vietnamese to test, so it makes me annoy. I turn if off in `tkCreateTaskView`, `setupView` method. 

### Animate to show more options 

App is not as simple as text. App needs animation. 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Ftodo%2Ftask_animate_show_task_option.gif?alt=media&token=4bb7773c-7ecb-441c-828f-dd1f8cf3f916" width="40%"/>

Very simple animation. We can see 4 things we need to do. 

1. Zoom animation for `addButton`
2. Rotate `addButton`
3. Change `addButton` color 
4. Animate `detailButton` and `calendarButton`

Do step by step. 

- Press `cmd + shift + O` to quick open file `knUIButton.swift`. This is my extensions for UIButton. We need to add 2 extensions here. 

```

    extension UIButton {
        func animateZooming() {
            // (1)
            guard let parentView = superview,
                  let edge = imageView?.frame.size.height else { return }
            let circo = UIView(frame: CGRect(x: 0, y: 0, width: edge, height: edge))
            circo.backgroundColor = UIColor.lightGray.withAlphaComponent(0.1)
            
            parentView.addSubviews(views: circo)
            circo.center = center
            circo.createRoundCorner(edge / 2)

            // (2)
            let level: CGFloat = 2.5
            UIView.animate(withDuration: 0.3, animations: {
                circo.transform = CGAffineTransform(scaleX: level, y: level)
            }, completion: { _ in
                circo.removeFromSuperview()
            })
        }
        
        // (3)
        func animateRotation(angle: CGFloat) {
            UIView.animate(withDuration: 0.3, animations: { [weak self] in
                guard let `self` = self else { return }
                self.transform = self.transform.rotated(by: angle)
            })
        }
    }

```

Method `animateZooming` only used for `addButton` or other buttons with same size at this moment. Why? Because the level (2) is specific for it. We will make it flexible for other buttons later if needed. We can do this yourself. Just try and find a good number for you. 

(1): The idea is to add a view to the superView of button and animate this view. Nothing affects to button. There are many ways to make this animation, but I choose the easiest one. 

(2): Just simple rotate the whole button with an angle. 

 - Open file `CreateTaskView.swift` to add new buttons and animation

 ```

    let detailButton = knUIMaker.makeButton(image: UIImage(named: "detail"))
    let calendarButton = knUIMaker.makeButton(image: UIImage(named: "event"))

    override func setupView() {
        addSubviews(views: textField, addButton, saveButton, detailButton, calendarButton)
        ...
        detailButton.fill(toView: addButton)
        calendarButton.fill(toView: addButton)

        detailButton.alpha = 0
        calendarButton.alpha = 0
        ...

        addButton.addTarget(self, action: #selector(showTaskOption))
    }
 ```

 Setup layout for new 2 buttons, and hide them by changing their opacity to 0, we will animate to display them. 

 ```

    var taskOptionShown = false
    @objc func showTaskOption() {

        addButton.animateZooming()
        if taskOptionShown == false {
            changeAddButtonColor(UIColor.color(r: 96, g: 99, b: 104))
            addButton.animateRotation(angle: CGFloat.pi / 4)
            animateDetailButton(visible: true)
            animateCalendarButton(visible: true)
        }
        else {
            changeAddButtonColor(UIColor.color(r: 71, g: 136, b: 241))
            addButton.animateRotation(angle: -CGFloat.pi / 4)
            animateDetailButton(visible: false)
            animateCalendarButton(visible: false)
        }
        
        taskOptionShown = !taskOptionShown
    }

    func changeAddButtonColor(_ color: UIColor) {
        addButton.tintColor = color
    }
    
    func animateDetailButton(visible: Bool) {
        let xPos = detailButton.center.x + 8 * (visible ? 1 : -1)
        let alpha: CGFloat = visible ? 1 : 0
        UIView.animate(withDuration: 0.25, animations: { [weak self] in
            guard let detailButton = self?.detailButton else { return }
            detailButton.transform = CGAffineTransform(translationX: xPos, y: 0)
            detailButton.alpha = alpha
        })
    }
    
    func animateCalendarButton(visible: Bool) {
        let xPos = detailButton.center.x + 64 * (visible ? 1 : -1)
        let alpha: CGFloat = visible ? 1 : 0
        UIView.animate(withDuration: 0.25, animations: { [weak self] in
            guard let calendarButton = self?.calendarButton else { return }
            calendarButton.transform = CGAffineTransform(translationX: xPos, y: 0)
            calendarButton.alpha = alpha
        })
    }

```

We have a flag `taskOptionShown` to mark whether buttons displayed. 

As I said before, we have 4 things to do in this animation, these are 4 things. 

```

    addButton.animateZooming()
    changeAddButtonColor
    addButton.animateRotation
    animateDetailButton + animateCalendarButton

```

Zoom animation in the gif image is too fast. You can run and see in your project. 

Run the app and you can see the animation but the addButton color is not changed. We need extra step. 

```

    let addButton = knUIMaker.makeButton(image: UIImage(named: "add_fill")?.changeColor())

    override func setupView() {
        ...
        addButton.tintColor = UIColor.color(r: 71, g: 136, b: 241)
    }

```

We need to make the image for imageView inside the button available to be changed color and set default color for it. 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Ftodo%2Ftask_demo_part_2.gif?alt=media&token=f7ad3b36-69bc-4285-a533-b9bc4fef7dd6" width="40%"/>

### Conclusion

That's enough for today. Keep reading, I'll come back soon. 