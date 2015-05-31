---
layout:     post
title:      Implementing an Expandable Text Cell
date:       2014-06-11 15:31:19
summary:    Learn how to implement an expandable text cell which automatically cuts off the text at a specified number of lines; just like in the reviews section of the Apple Store!
categories: blog
---

Learn how to implement an expandable text cell which automatically cuts off the text at a specified number of lines; just like in the reviews section of the Apple Store!

<center>![]({{ site.url }}images/blog-expandablecell/ExpandableCellAnimation.gif)</center>

### Code
I've posted the full code for this blog post on GitHub: 
[http://www.github.com/kgaidis/blog-expandingcell](http://www.github.com/kgaidis/blog-expandingcell)

### Project Setup


* #### ExpandableLabelTableViewCell
A simple cell that doesn't do much else other than give us an outlet to an `ExpandableLabel`, which is just a subclass of `UILabel`.

* #### ExpandableLabelCellModel
An `NSObject` that contains information on how each cell will be presented. First we have `NSString *string`, which contains the text that will go inside the label. Then we have `BOOL expanded`, which is used to track whether the cell has been expanded already. This is a very important class because it controls how many lines the cell will display via `kMaxNumberOfLines`.

* #### ViewController
The view controller is pretty standard. We implement `UITableViewDelegate` and `UITableViewDataSource`, populate the cell models with random sized strings and just display the cells! The actual logic is handled by `ExpandableLabelCellModel` and `ExpandableLabel`.

* #### ExpandableLabel
The most important part of the project and why you came to see this blog post! We will talk about how this is implemented, step-by-step below, but I'll summarize the most important parts here. First we must decide whether we need to truncate the cell, if we decide to truncate, we append *"...More"* to the cut off text. Truncation is decided by the labels `numberOfLines` property. If the value is `0` it means that we do not want to truncate the label. If the value `>0` we must calculate whether the `text` of the `UILabel` fits the `numberOfLines`. If the text fits, we want to leave it alone and display it in its full form, otherwise we truncate it. 


### ExpandableLabel Functionality



### ----------- Below is test stuff ---------------


{% highlight objective-c %}
class Awesome < ActiveRecord::Base
  include EvenMoreAwesome

  validates_presence_of :something
  validates :email, email_format: true

  def initialize(email, name = nil)
    self.email = email
    self.name = name
  end
end
{% endhighlight %}

Outline
1. Overview
2. Project
- ViewController
- ExpandableLabel
- ExpandableLabelTableViewCell
- ExpandableLabel

`void lol`
