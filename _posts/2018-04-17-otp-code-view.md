---
layout: post
title: OTP Code View
subtitle: Make your own OTP code input view
date:   2018-4-17
background: 'https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fotp%2Fotp_hero.jpg?alt=media&token=d025c7c1-3396-4eb7-bbb0-f98b831ef485'
---

Two years ago, I did a view for this purpose. Detail is on my github: [https://github.com/nguyentruongky/ActiveCode](https://github.com/nguyentruongky/ActiveCode).

<img src="https://raw.githubusercontent.com/nguyentruongky/ActiveCode/master/E7AVZubaOT.gif" width="40%"/> 

It's 100% code, without auto layout. Today, I make another view, with same purpose, but use auto layout, better UI and much easier to maintain. 

UI will be like this. 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fotp%2Fotp_design.png?alt=media&token=2355a603-7056-45c1-94fa-dfb18e6e0318" width="40%"/>

### Initilize view 

Init view with number of digits you need and an action to validate your OTP from your controller. The number of digits should be less than 6 for good UI. 

Anyone can customize UI themselves for more digits. Digit indicator is • or _ can save more space.

```
private var digitCount = 0
private var validate: ((String) -> Void)?
convenience init(digitCount: Int, validate: @escaping ((String) -> Void)) {
    self.init(frame: CGRect.zero)
    self.digitCount = digitCount
    self.validate = validate
    setupView()
}
```

### Setup the UI 

```
override func setupView() {
    // (1)
    guard digitCount > 0 else { return }

    // (2)
    var constraints = "H:|-8-"
    for i in 0 ..< digitCount {
        let label = makeLabel()
        if i > 0 {
            label.width(toView: labels[0])
        }
        constraints += "[v\(i)]-8-"
    }
    constraints += "|"
    addConstraints(withFormat: constraints, arrayOf: labels)
    height(60)
    
    // (3)
    setCode(at: 0, active: true)
    hiddenTextField.becomeFirstResponder()
    addGestureRecognizer(UITapGestureRecognizer(target: self, action: #selector(becomeFirstResponder)))
}

private func makeLabel() -> UILabel {
    let label = knUIMaker.makeLabel(font: UIFont.systemFont(ofSize: 45),
                                color: color_69_125_245,
                                alignment: .center)
    // (4)
    label.createRoundCorner(5)
    label.createBorder(0.5, color: color_102)

    // (5)
    addSubview(label)
    label.vertical(toView: self)
    labels.append(label)
    return label
}
```
(1): Prevent code running while no digit. Only support init view by code with init(digitCount:validate), so should prevent other init way to make wrong behaviour. 

(2): Set Auto layout code. Create a Visual Format Language string (like this: "H:|-8-[v0]-[v1]-8-|"). Detail is [here](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html). 

To make sure all digit indicators have same width, I set widthConstraint equal to the first indicator. 

(3): Minor stuffs, make first indicator active, set focus to the hiddenTextField to show keyboard. 

(4): This is how the indicator look. Want to change UI, just do it here. Please keep the (5) stay here. it is for auto layout setting. 

### Set hidden UITextField 
Just add a UITextField, which is overlapped to use its keyboard and delegate. Every key input, I will update the digit indicator by catch the character in UITextFieldDelegate
```
lazy var hiddenTextField = self.addHiddenTextField()
private func addHiddenTextField() -> UITextField {
    let tf = UITextField()
    tf.translatesAutoresizingMaskIntoConstraints = false
    tf.keyboardType = .numberPad
    tf.isHidden = true
    tf.delegate = self
    
    addSubviews(views: tf)
    tf.fill(toView: self)
    
    return tf
}
override func becomeFirstResponder() -> Bool {
    hiddenTextField.becomeFirstResponder()
    return true
}
```

Conform `UITextFieldDelegate` and update method textfield(shouldChangeCharactersIn:replacementString)

```
func textField(_ textField: UITextField, shouldChangeCharactersIn range: NSRange, replacementString string: String) -> Bool {
    var newText = string
    // (1)
    if isInvalid {
        isInvalid = false
    }
    else {
        newText = (textField.text! as NSString).replacingCharacters(in: range, with: string)
    }

    // (2)
    let codeLength = newText.length
    guard codeLength <= digitCount else { return false }
    textField.text = newText
    
    // (3)
    func setTextToActiveBox() {
        for i in 0 ..< codeLength {
            let char = textField.text!.substring(from: i, to: i)
            labels[i].text = char
            setCode(at: i, active: true)
        }
    }
    
    // (4)
    func setTextToInactiveBox() {
        for i in codeLength ..< digitCount {
            labels[i].text = ""
            setCode(at: i, active: false)
        }
        
        if codeLength <= digitCount - 1 {
            setCode(at: codeLength, active: true)
        }
    }
    
    setTextToActiveBox()
    setTextToInactiveBox()
    
    if codeLength == digitCount {
        validateCode(code: textField.text!)
    }
    return false
}
```

(1): Reset all boxes when the code is invalid and type new character. 

(2)(3)(4): Update every character from `hiddenTextField` to every indicator box. Rest of boxes will be set to empty. 

### Result
The main part is ready. Minors methods, properties are in the lib in github. Link is below. 

<img src="https://firebasestorage.googleapis.com/v0/b/blogs-1de93.appspot.com/o/assets%2Fotp%2Fotp_final.gif?alt=media&token=a60f523c-40ac-429d-9546-804344e27d1f" width="40%"/>