---
layout: post
title:  Thinking about Coinhako history screen
subtitle: A small note about how Coinhako make their history screen with status and filter in the same screen. 
date:   2018-3-29
background: 'https://www.coinhako.com/assets/friendliest_wallet-13ccee4871179541b33bd8f1f974a484870de1daf3e210711b8d6ecca83a2c16.svg'
---
Have just joined cryptocurrencies market and select [Coinhako](https://www.coinhako.com) as a wallet because of its simplicity. The UI is really simple for newcomers like me. But the UI make me feel excited, some screens are interesting to implement. If you're new to crytocurrencies, you can try this app [here](https://itunes.apple.com/us/app/coinhako-wallet/id1137855704?mt=8).

When I get design from designers, I think a lot about how to make it done. Have to pick good one. If pick wrong one, it takes lots of time to change. The history screen in this is one of screen I had to think a lot. Here what I thought and what I did. 


Screenshots from Coinhako app

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fmulti_screens_in_one%2Fmulti_screen_in_one_1.png?alt=media&token=3294c068-91f5-4f5b-89e5-8c6226e9ad85" width="240px"/>
<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fmulti_screens_in_one%2Fmulti_screen_in_one_2.png?alt=media&token=8a2162c1-4c38-478b-ad21-8b272a63fd8a" width="240px"/>
<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fmulti_screens_in_one%2Fmulti_screen_in_one_3.png?alt=media&token=cad07161-1c66-48eb-99ec-4e268e7198f7" width="240px"/>
<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fmulti_screens_in_one%2Fmulti_screen_in_one_4.png?alt=media&token=425d0c31-48bd-4f61-8b8f-c48a368b647f" width="240px"/>
<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fmulti_screens_in_one%2Fmulti_screen_in_one_5.png?alt=media&token=6b1818c3-e292-4f52-984b-371d9a8caf11" width="240px"/>

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
<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fmulti_screens_in_one%2Fmulti_screen_in_one_demo.gif?alt=media&token=0efd1a87-910e-44b5-be56-31ac988b49e1" width="320px"/>

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
