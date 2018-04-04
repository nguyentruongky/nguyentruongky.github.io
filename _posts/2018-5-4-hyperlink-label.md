---
layout: post
title: Hyperlink Label
subtitle: Make an easy hyperlink label
date:   2018-4-5
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fhyperlink-label%2Fhyperlink_label.jpg?alt=media&token=fd379e35-8455-4a20-bd33-9549eac9e6b6'
---
Clickable Label is very popular in iOS, especially in Login, Register screen. You can easily see some text like this:

By register, I agree to ... **Terms of Service** and **Private Policy**

This is how I make this label. 

### Define your texts 
Make sure the text you need to make clickable is exacly same to the full text. 

```
let termText = "By register, I agree to ... Terms of Service and Private Policy"
let term = "Terms of Service"
let policy = "Private Policy"
```

### Format the Label 
```
let termLabel = UILabel()
let formattedText = String.format(strings: [term, policy],
                                    boldFont: UIFont.boldSystemFont(ofSize: 15),
                                    boldColor: UIColor.blue,
                                    inString: termText,
                                    font: UIFont.systemFont(ofSize: 15),
                                    color: UIColor.black)
termLabel.attributedText = formattedText
termLabel.numberOfLines = 0
let tap = UITapGestureRecognizer(target: self, action: #selector(handleTermTapped))
termLabel.addGestureRecognizer(tap)
termLabel.isUserInteractionEnabled = true
termLabel.textAlignment = .center
```

`String.format` is an extension from my code collection. This is the full function. 

```
extension String {
    static func format(strings: [String],
                       boldFont: UIFont = UIFont.boldSystemFont(ofSize: 14),
                       boldColor: UIColor = UIColor.blue,
                       inString string: String,
                       font: UIFont = UIFont.systemFont(ofSize: 14),
                       color: UIColor = UIColor.black) -> NSAttributedString {
        let attributedString =
            NSMutableAttributedString(string: string,
                                      attributes: [
                                        NSAttributedStringKey.font: font,
                                        NSAttributedStringKey.foregroundColor: color])
        let boldFontAttribute = [NSAttributedStringKey.font: boldFont, NSAttributedStringKey.foregroundColor: boldColor]
        for bold in strings {
            attributedString.addAttributes(boldFontAttribute, range: (string as NSString).range(of: bold))
        }
        return attributedString
    }
}
```

### Handle Label Tap Gesture

I get the tap location in the Label and check if this location belongs to term or policy text range. 
```
@objc func handleTermTapped(gesture: UITapGestureRecognizer) {
    let termString = termText as NSString
    let termRange = termString.range(of: term)
    let policyRange = termString.range(of: policy)
    
    let tapLocation = gesture.location(in: termLabel)
    let index = termLabel.indexOfAttributedTextCharacterAtPoint(point: tapLocation)
    
    if checkRange(termRange, contain: index) == true {
        handleViewTermOfUse()
        return
    }
    
    if checkRange(policyRange, contain: index) {
        handleViewPrivacy()
        return
    }
}
```

### Supported code

- Check if a range contain an index 
```
func checkRange(_ range: NSRange, contain index: Int) -> Bool {
    return index > range.location && index < range.location + range.length
}
```

- Get index from a point in UILabel
```
extension UILabel {
    func indexOfAttributedTextCharacterAtPoint(point: CGPoint) -> Int {
        assert(self.attributedText != nil, "This method is developed for attributed string")
        let textStorage = NSTextStorage(attributedString: self.attributedText!)
        let layoutManager = NSLayoutManager()
        textStorage.addLayoutManager(layoutManager)
        let textContainer = NSTextContainer(size: self.frame.size)
        textContainer.lineFragmentPadding = 0
        textContainer.maximumNumberOfLines = self.numberOfLines
        textContainer.lineBreakMode = self.lineBreakMode
        layoutManager.addTextContainer(textContainer)
        
        let index = layoutManager.characterIndex(for: point, in: textContainer, fractionOfDistanceBetweenInsertionPoints: nil)
        return index
    }
}
```

### Demo

And result is: 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fhyperlink-label%2Fterm_demo.gif?alt=media&token=9cd27012-8f3f-4d42-adfd-c6fd45df6857" width="240px"/>

Demo is [here](https://github.com/nguyentruongky/HyperlinkLabel)