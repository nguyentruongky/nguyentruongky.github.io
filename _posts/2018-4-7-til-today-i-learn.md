---
layout: post
title: Today I learn
subtitle: A note about what I learn a day, it can be anything, not only iOS. 
date:   2018-4-7
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Ftoday-i-learn%2Fnever-stop-learning.jpg?alt=media&token=018d6c76-c64a-42a6-9b6c-317ec9b8b1d6'
---

# 04/04/2018

### How to remove pod in iOS project
```
sudo gem install cocoapods-deintegrate cocoapods-clean
pod deintegrate
pod clean
rm Podfile
```

### Integrate Zendesk SDK 

Do not follow pod instruction from Zendesk tutorial, take too much time to fix but can't run. 

Add SDK manual way: Add 2 more framework from iOS `MobileCoreServices.framework` and `Security.framework.`

Access to Tagets/General, add all Zendesk SDK files to Embedded Binaries.

### Merge unrelated histories in git 
Usually have this issue when create new project in local, create repo in git then add remote url from git to local project.

`git pull origin branch_name --allow-unrelated-histories`

### Basic Authorization 

Learn from [https://gist.github.com/cmoulton/c26dc371d771c5cbaff325de6bbe5c77](https://gist.github.com/cmoulton/c26dc371d771c5cbaff325de6bbe5c77)
```
let userName = "myUsername"
let password = "myPassword"
let authString = userName + ":" + password
let credentialData = authString.dataUsingEncoding(NSUTF8StringEncoding)!
let base64Credentials = credentialData.base64EncodedStringWithOptions([])

let headers = ["Authorization": "Basic " + base64Credentials]
let url = URL(string: api)!
Alamofire.request(url, method: .post,
                parameters: params,
                encoding: JSONEncoding.default,
                headers: headers)
    .validate().responseJSON { (response) in
    print(response)
}
```

# 05/04/2018
### Format string
Always display 2 characters for Int: 09:10, 10:15

`String(format: "%02d:%02d", hr, min)`

# 06/04/2018
### Open setting of my app
```
if let url = URL(string: UIApplicationOpenSettingsURLString) {
    UIApplication.shared.openURL(url)
}
```

### Add event to calendar 
```
struct ogeSystemCalendar {
    let eventStore = EKEventStore()
    func addEvent(title: String, startDate: Date,
                  endDate: Date, notes: String?) {
        eventStore.requestAccess(to: .event) { (granted, error) in
            if granted == false {
                DispatchQueue.main.async {
                    self.tellNoPermission() }
                return
            }
            
            self.addToCalendar(title: title, start: startDate,
                               end: endDate, notes: notes)
        }
    }
    
    private func tellNoPermission() {
        let alert = ogeMessage.showDialog(title: "no_permission".i18n, description: "no_calendar_permission".i18n)
        alert.addAction(PMAlertAction(title: "OK", style: .default, action: {
            DispatchQueue.main.async { ogeSystemInteractor.openCalendar() }
        }))
        appDelegate.ogeniiManager?.present(alert)
    }
    
    private func addToCalendar(title: String, start startDate: Date,
                  end endDate: Date, notes: String?) {
        let event = EKEvent(eventStore: eventStore)
        event.title = title
        event.startDate = startDate
        event.endDate = endDate
        event.notes = notes
        event.calendar = eventStore.defaultCalendarForNewEvents
        do {
            try eventStore.save(event, span: .thisEvent)
            DispatchQueue.main.async {
                ogeMessage.showMessage("saved_to_calendar".i18n) }
        } catch { }
    }
}
```

# 07/04/2018

### Make app icon iOS 
[https://makeappicon.com/](https://makeappicon.com/) is a really wonderful tool. Just copy folder `AppIcon.appiconset` and replace the old one in project and App Icon is set. Thanks a lot, awesome app. 

### Update SQL by removing text in text 

`UPDATE My_Table set my_column = REPLACE(my_column, "text_need_removed", '') where my_condition`

### Error in git 
Error: `The requested URL returned error: 403 while accessing`

Solution: 
- Edit `.git/config` file under your repo directory
- Find `url=` entry under section `[remote "origin"]`
- Change all the texts before @ symbol to `ssh://git`
- Save

Source: [StackOverflow](https://stackoverflow.com/questions/7438313/pushing-to-git-returning-error-code-403-fatal-http-request-failed)

# 11/04/2018
### Prevent rotation in iOS
There are three kinds of Device orientation keys there in the info.plist now.

- Supported interface orientations (iPad)
- Supported interface orientations (iPhone)
- Supported interface orientations

Source: [StackOverflow](https://stackoverflow.com/questions/10125050/can-you-disable-rotation-globally-in-an-ios-app)
