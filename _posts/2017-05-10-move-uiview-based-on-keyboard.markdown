---
layout: post
title:  "Move UIView when an iOS keyboard appears or changes"
date:   2017-05-10 17:23:00 +0100
categories: swift
---

{::comment} 
An image: ![View moving upon keyboard load](/assets/img/uiview-movement.gif) 
{:/comment}

So, your app looks great but, when a user goes to enter an input, the keyboard pops up and now your UITextField/UIView is covered -- the user can't see what they're typing. Let's fix that. This post is broken down into two sections: retrieving the keyboard height and moving a UIView based on that value.

## Retrieving the keyboard height 

First up you're going to want to set a couple of *notifications*, one to be called when the keyboard shows, the other, when it hides. Place the following in your *viewDidLoad()* or equivalent. 
~~~ swift
NotificationCenter.default.addObserver(self,
                                       selector: #selector(self.keyboardWillShow(sender:)), name:NSNotification.Name.UIKeyboardWillShow, object: nil)
NotificationCenter.default.addObserver(self,
                                       selector: #selector(self.keyboardWillHide(sender:)), name:NSNotification.Name.UIKeyboardWillHide, object: nil)
~~~

By declaring these observers, when the keyboard appears, the program will now look for the functions *keyboardWillShow()* and *keyboardWillHide()*. So let's add those in now too.
~~~ swift
func keyboardWillShow(sender: Notification) {    
    guard let keyboardSize = sender.userInfo?[UIKeyboardFrameBeginUserInfoKey] as? NSValue else{
        return
    }
    let keyboardHeight = keyboardSize.cgRectValue.height // keyboard height as CGFloat
}
func keyboardWillHide(sender: Notification) {    
    // You can duplicate the above code in here if you'd like.
    // Personally, I use this function to restore the UIView's position, so the keyboard height isn't necessarily needed.
}
~~~

With this code we are now able to retreive the keyboard height -- whatever the state, be that *predictive*, *emoji*, or *standard*.

## Moving a view based on keyboard height

Personally, I like to move my UIView up a distance of half the remaining screen space. If your UIView is not too large, the following code will center the view in the free space.
~~~ swift  
func keyboardWillShow(sender: Notification) {  
    guard let keyboardSize = sender.userInfo?[UIKeyboardFrameBeginUserInfoKey] as? NSValue else{
        return
    }

    let keyboardHeight = keyboardSize.cgRectValue.height // keyboard height as CGFloat; blue
    let viewHeight = UIScreen.main.bounds.size.height // screen height; magenta
    let remainingScreenSpace = viewHeight-keyboardHeight // leftover screen space; yellow
    let chosenUIViewHeight = chosenUIView.frame.size.height // change to your desired UIView height; green
    let posY = (remainingScreenSpace/2) - (chosenUIViewHeight/2) // the distance from the top of screen/keyboard; orange 

    UIView.animate(withDuration: 0.25, animations: { 
        self.chosenUIView.frame = CGRect( x: self.chosenUIView.frame.origin.x, y: posY, width: self.chosenUIView.frame.size.width, height: self.chosenUIView.frame.size.height );
    });
}
~~~

Here's a pic showing the result of this code, note that the UIView is centrally positioned:

![View moving upon keyboard load](/assets/img/uiview-visualised.png) 

An alternative is to simply move the UIView up the distance of the keyboard height. The code for that would be
~~~ swift
func keyboardWillShow(sender: Notification) {  
    guard let keyboardSize = sender.userInfo?[UIKeyboardFrameBeginUserInfoKey] as? NSValue else{
        return
    }

    let keyboardHeight = keyboardSize.cgRectValue.height // keyboard height as CGFloat
    let posY = self.chosenUIView.frame.origin.y - keyboardHeight 

    UIView.animate(withDuration: 0.25, animations: { 
        self.chosenUIView.frame = CGRect( x: self.chosenUIView.frame.origin.x, y: posY, width: self.chosenUIView.frame.size.width, height: self.chosenUIView.frame.size.height );
    });
}
~~~

Well, that's it. Hope it helps.
