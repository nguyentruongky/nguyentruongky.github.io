---
layout: post
title:  Thinking about Coinhako history screen
subtitle: A small note about how Coinhako make their history screen with status and filter in the same screen. 
date:   2018-3-29
background: 'https://www.coinhako.com/assets/friendliest_wallet-13ccee4871179541b33bd8f1f974a484870de1daf3e210711b8d6ecca83a2c16.svg'
---
Have just joined cryptocurrencies market and select [Coinhako](https://www.coinhako.com) as a wallet because of its simplicity. The UI is really simple for newcomers like me. But the UI make me feel excited, some screens are interesting to implement. If you're new to crytocurrencies, you can try this app [here](https://itunes.apple.com/us/app/coinhako-wallet/id1137855704?mt=8).

This is my thinking about how Coinhako's developers make the app, and try with a simple demo.

Screenshots from Coinhako app

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fmulti_screen_in_one_1.PNG?alt=media&token=76851c42-e60f-4ea4-999b-4831747b19f6" width="320px"/>
<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fmulti_screen_in_one_2.PNG?alt=media&token=8b86f562-bb3f-4fc8-b724-b497c2ac131f" width="320px"/>
<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fmulti_screen_in_one_3.PNG?alt=media&token=ef118078-1859-4624-a982-81456e43d934" width="320px"/>
<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fmulti_screen_in_one_4.PNG?alt=media&token=ba3a9c48-90f9-42ca-9576-48c837fc784a" width="320px"/>
<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fmulti_screen_in_one_5.PNG?alt=media&token=9e853be9-be5b-42ba-8c02-5538c95cf724" width="320px"/>

## How it works

- Display 2 lists of status: Pending transactions and Completed transactions. This can be done with a UITableView in a UIViewController. 
- There is filter, and filter result is a list of transaction, no matter what status. It looks like displayed in the same screen with 2 status lists before. 
- Every list has empty state. 
- Every list can load more if available. 
- Every list can reload when displayed.

## My thinking 
First thinking is a UITableView in a UIViewController. There are 4 datasources, `pendingDatasource`, `completedDatasource`, `filterDatasource` and `mainDatasource`. When user selects pending segment or completed segment or filter, app will update the correct datasource and `mainDatasource` then reload UITableView. Empty state will be dislay if the selected datasource is empty. This can be the good solution. But I am concern about load more and reload. Very difficult to handle all of them to reload and load more. 

New thinking, make a custom view contain UITableView to display list, handle load data, reload, load more event, empty state inside the view. When user does any actions, add the view to UIViewController and remove the previous one. I will do this solution in my demo. 

## Demo 
<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fmulti_screen_in_one_demo.gif?alt=media&token=5e529cff-97f5-4380-95fc-32619ad32727" width="320px"/>

### Create project
I am a big fan of Auto Layout Programatically, so that no Storyboard in my project. I added some libs for Auto Layout and some codebase. 

The demo has HistoryController and FilterController. 

HistoryController contains a UISegmentControl for status and a bar button to show filter. 
FilterController contains only UIButton to turn filter on/off. 

UI for controller is in `setupView()`. 

### Add ListView 

ListView is a base view for PendingView, CompletedView, FilterView. ListView contains a UITableView and conform UITableViewDelegate, UITableViewDataSource. ListView also handles load data or load more, status (empty, error, loading).

This is just a simple demo, so I make a Datasource (fake data). In my regular projects, I will load data/load more by workers to handle request result and separate code for the View. 

```
class ListView: knView {
    let maxItemCount = 20
    
    let fakeData = Datasource()
    var currentPage = 0
    var canLoadMore = true 
    var datasource = [String]() { didSet { tableView.reloadData() }}
    
    lazy var tableView: UITableView = { [weak self] in
        let tb = UITableView()
        tb.translatesAutoresizingMaskIntoConstraints = false
        tb.separatorStyle = .none
        tb.showsVerticalScrollIndicator = false
        tb.dataSource = self
        tb.delegate = self
        tb.register(knTableCell.self, forCellReuseIdentifier: "knTableCell")
        return tb
        }()
    
    override func setupView() {
        translatesAutoresizingMaskIntoConstraints = false
        addSubviews(views: tableView)
        tableView.fill(toView: self)
        backgroundColor = .white
    }
    
    func fetchMore() { }
    func fetchData() { }

    func checkLoadMoreAvailable(currentCount: Int) {
        canLoadMore = currentCount == maxItemCount
        currentPage += currentCount == maxItemCount ? 1 : 0
    }
}


extension ListView: UITableViewDelegate, UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return datasource.count }
    
    func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
        if indexPath.row == datasource.count - 1 {
            fetchMore()
        }
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "knTableCell", for: indexPath) as! knTableCell
        cell.textLabel?.text = datasource[indexPath.row]
        return cell
    }
    
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat { return 66 }
}
```

### Add PendingView, CompletedView, FilterView
Add more 3 views and override `fetchData()`, `fetchMore()`. Example for PendingView
```
class PendingView: ListView {
    override func fetchData() {
        let newData = fakeData.getPendingList(page: 0)
        checkLoadMoreAvailable(currentCount: newData.count)
        datasource = newData
    }
    
    override func fetchMore() {
        guard canLoadMore else { return }
        let newData = fakeData.getPendingList(page: currentPage)
        checkLoadMoreAvailable(currentCount: newData.count)
        datasource += newData
    }
}
```

### Handle logic 
In HistoryController, handle the add view and remove previous one by setting ListType.

```
func showList(_ listType: ListType) {
    switch listType {
        case .pending:
            if pendingView.datasource.count == 0 {
                pendingView.fetchData()
            }
            statusSegment.selectedSegmentIndex = 0
            view.addSubview(pendingView)
            pendingView.fill(toView: view, space: UIEdgeInsets(top: 16 * 2 + 44 + 84, left: 0, bottom: 0, right: 0))
            currentList?.removeFromParentView()
            currentList = pendingView
            
        case .completed:
            if completedView.datasource.count == 0 {
                completedView.fetchData()
            }
            statusSegment.selectedSegmentIndex = 1
            view.addSubview(completedView)
            completedView.fill(toView: view, space: UIEdgeInsets(top: 16 * 2 + 44 + 84, left: 0, bottom: 0, right: 0))
            currentList?.removeFromParentView()
            currentList = completedView
            
        case .filter:
            view.addSubview(filterView)
            filterView.fill(toView: view, space: UIEdgeInsets(top: 66, left: 0, bottom: 0, right: 0))
            currentList?.removeFromParentView()
            currentList = filterView
        }
    }
```

This is the most complicated portion in this demo. Other minor codes you can see in the sample on [Github](https://github.com/nguyentruongky/MultiScreensInOneDemo).

## Conclusion 

This is my thinking about how Coinhako's iOS app handle history screen. Not sure it is right or not. But this solution can be useful in screens like this.