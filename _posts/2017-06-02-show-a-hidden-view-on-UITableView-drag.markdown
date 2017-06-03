---
layout: post
title:  "Hidden view to show on UITableView drag"
author: Christopher Fulford
date:   2017-06-02 17:00:00 +0100
categories: swift
---

Ever seen an iOS app where you drag a table down to reload the content? The goal here is to mimic that design, except when the UITableView is dragged down, the previously hidden view will appear and remain on-screen until the user drags the table back up.

## Creating a custom view for a UITableView

Create a new xib file, title it *HiddenView* and style as you wish. This xib will be the hidden view. 

Now, we're going to create an IBDesignable UIView. Create a new swift file and name it *HiddenView*. Replace the contents of the file with:

~~~swift
import UIKit

@IBDesignable class HiddenView: UIView  {
    let nibName = "HiddenView" // Name of your xib file
    var view: UIView!
    
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
        xibSetUp()
    }
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        xibSetUp()
    }
    
    func loadViewFromNib() -> UIView {        
        let bundle = Bundle(for: type(of: self))
        let nib = UINib(nibName: nibName, bundle: bundle)
        
        let view = nib.instantiate(withOwner: self, options: nil)[0] as! UIView
        return view
    }
    
    func xibSetUp() { // setup the view from .xib
        view = loadViewFromNib()
        view.frame = self.bounds
        view.autoresizingMask = [UIViewAutoresizing.flexibleWidth, UIViewAutoresizing.flexibleHeight]
        addSubview(view)
    } 
}
~~~

Now we have our hidden view setup, go to your desired UITableView, drag in an empty UIView toward the top, then set its class to *HiddenView*. With a bit of luck, when you run your app a custom view will be at the top of the table.

We've got it, let's hide it. 

## Hide/Show UIView in UITableView

Go to your UITableView controller and add the following:

~~~swift
override func viewWillAppear(_ animated: Bool) {
    tableView.contentInset = UIEdgeInsetsMake(Y, 0, 0, 0) // Where Y is your defined hidden view height.
}
~~ 

This ensures that when the table is displayed, the view will be hidden. Lastly, add:

~~~swift
override func scrollViewDidEndDragging(_ scrollView: UIScrollView, willDecelerate decelerate: Bool) {
    if(scrollView.contentOffset.y < -Y){ // Where Y is your defined hidden view height.         
        UIView.beginAnimations(nil, context: nil)
        UIView.setAnimationDuration(0.25)
        tableView.contentInset = UIEdgeInsetsMake(Y, 0, 0, 0) // Where Y is your defined hidden view height.
        UIView.commitAnimations()
    }
    else{
        UIView.beginAnimations(nil, context: nil)
        UIView.setAnimationDuration(0.25)
        tableView.contentInset = UIEdgeInsetsMake(0, 0, 0, 0)
        UIView.commitAnimations()
    }
}
~~~

This checks if the user has scrolled beyond the height of the hidden view, if it has, the content inset adjusts to then show the hidden view. Once the user scrolls the table down, the view goes back into hiding.

That's it. Check out the project [here](https://github.com/chrisjgf/Hidden-View-UITableView)
