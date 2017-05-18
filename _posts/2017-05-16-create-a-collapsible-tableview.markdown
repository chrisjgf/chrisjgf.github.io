---
layout: post
title:  "Create A Collapsible UITableView"
author: Christopher Fulford
date:   2017-05-16 20:10:00
categories: swift
---

This is a fairly simple, no bells and whistles, collapsible UITableView. The table takes user input through row selection, then reloads the table to show data relevant to that row title. Selecting the title row again returns the user to the main view.

First up, we want to create a new object for our data. In this example I'm using an array of strings, but you can re-jiggle and play around with the code to suit your needs.
~~~ swift
class TableData{
    var title: String? // Title for a row
    var content: [String]? // Content after row selection
    init (_ str: String?){ // Takes row title in initialisation 
        title = str
    }
}
~~~

Now we've got a way to structure our data, lets give the object our data. First of all, add the following two class variables, then alter your *ViewDidLoad()* and change as desired.
~~~swift
var dataForTable = [TableData]()
var selectedIndexPath: IndexPath?

override func viewDidLoad() {
    super.viewDidLoad()
    
    let dataSet1 = TableData("Oranges")
    dataSet1.content = ["A", "B", "C"]
    
    let dataSet2 = TableData("Pears")
    dataSet2.content = ["D", "E", "F", "G"]
    
    let dataSet3 = TableData("Apples")
    dataSet3.content = ["H", "I"]
    
    dataForTable.append(dataSet1)
    dataForTable.append(dataSet2)
    dataForTable.append(dataSet3)
}
~~~

We have the data, now we need to configure the tableView to display it. Lets first tell the table how much data we want to show.
~~~swift
override func numberOfSections(in tableView: UITableView) -> Int {
    return dataForTable.count
}
override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    if let selected = selectedIndexPath{ // if row has been selected
        if section == selected.section{ // if selected row == current row
            return dataForTable[selected.section].content!.count + 1 // +1 for section title
        } else { // remove rows
            return 0
        }
    } else { // return title rows
        return 1
    }
}
~~~

Lets handle the cell selection now. This allows the the user to touch the row title, but no other sub-rows.
~~~swift
override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    if (indexPath.row == 0){ // section titles
        if let selected = selectedIndexPath{ // has value
            if selected == indexPath{
                tableView.deselectRow(at: indexPath, animated: true)
                selectedIndexPath = nil
            }
        } else{
            selectedIndexPath = indexPath
        }
    }
    tableView.reloadData()
}
~~~

Last, but not least, we need to show the data.
~~~swift
override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "identifier", for: indexPath)

    if indexPath.row == 0{ // header cell
        cell.textLabel?.text = dataForTable[indexPath.section].title
        cell.backgroundColor = UIColor.gray
    } else{
        cell.textLabel?.text = dataForTable[indexPath.section].content?[indexPath.row-1] // -1 because of header cell
        cell.backgroundColor = UIColor.white
    }

    return cell
}
~~~

With a bit of luck, your UITableView will now be comprised of three gray rows. Selecting one of these will reload the table so that at the top is the row title, followed subsequently by the data in the section's *content* array. Selecting the cell again will return the user back to the three gray rows.

Check out the project [here](https://github.com/chrisjgf/Collapsible-UITableView)!