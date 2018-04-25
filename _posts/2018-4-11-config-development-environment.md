---
layout: post
title: Short note - Configure the development environments
subtitle: Quick and easy to switch environments in coding
date:   2018-4-11
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fconfigure-environment%2Fconfigure-environment-coding.jpg?alt=media&token=0902f403-18b5-4d75-a430-cc22c3b072bb'
---
My previous note is about configuration to release multi apps in the same project. I can use these configurations to build different enviroments for internal testing via Fabric, HockeyApp or TestFlight. For instance, production, staging and development have different icons, endpoint. Detail is [here](https://kynguyen.space/2018/04/07/user-define-setting-ios.html).

But in development, I usually connect devices to Macbook, build and run directly from Xcode, no need to switch scheme. We have 3 different environments in development: `production`, `staging`, `development`. It means there are 3 endpoints. While coding, I have to switch to 3 of them very often, for coding or debugging for production. 

So I write some code to make it easier. 

````

    enum EndPoints: String {
        case dev = "http://dev.kynguyen.space"
        case pro = "https://kynguyen.space"
        case staging = "http://staging.kynguyen.space"
    }

    var baseUrl: String = {
        return EndPoints.pro.rawValue 
    }()

```

So everytime I need to change, I switch `baseUrl` to EndPoints I need, very easy. 

But I usually forget to switch to production endpoint when release, even one time I pushed to TestFlight. So do one more step. 

```

    var baseUrl: String = {
        #if DEBUG
            return EndPoints.dev.rawValue
        #else
            return EndPoints.pro.rawValue
        #endif
    }()

```

`#if DEBUG ... #else` is very useful for hard-coded values. Can be use widely in coding and testing. 

When switch environment, we need to logout the current account and sign in to another account to prevent unknown error, avoid wasting time for debug. It's a damn experience with error like this. 

So I need to remember current enviroment to UserDefault. 

```

    var currentUrl: EndPoint {
        get {
            if let url = UserDefaults.standard.value(forKeyPath: "currentUrl") as? String,
                let environment = EndPoint(rawValue: url) {
                return environment
            }
            return EndPoint.dev
        }
        set {
            UserDefaults.standard.setValue("\(newValue)", forKeyPath: "currentUrl")
        }
    }

```

And check every time app runs. Show login if EndPoint is changed. 

```

    var isNeedToLogout: Bool {
        if let baseUrl = EndPoint(rawValue: Setting.baseUrl),
            baseUrl == currentUrl { return false }
        return true
    }
    
```

<hr/>

Just some basic tips but they save me much time in development. 