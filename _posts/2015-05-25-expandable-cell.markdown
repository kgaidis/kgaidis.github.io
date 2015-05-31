---
layout:     post
title:      Implementing an Expandable Text Cell
date:       2014-06-11 15:31:19
summary:    Learn how to implement an expandable text cell which automatically cuts off the text at a specified number of lines; just like in the reviews section of the Apple Store!
categories: blog
---

Learn how to implement an expandable text cell which automatically cuts off the text at a specified number of lines; just like in the reviews section of the Apple Store!

<center>![]({{ site.url }}images/blog-expandablecell/ExpandableCellAnimation.gif)</center>

## Code
I've posted the full code for this blog post on GitHub: 
[http://www.github.com/kgaidis/blog-expandingcell](http://www.github.com/kgaidis/blog-expandingcell)

## Project Classes


#### ExpandableLabelTableViewCell
A simple cell that doesn't do much else other than give us an outlet to an `ExpandableLabel`, which is just a subclass of `UILabel`.

#### ExpandableLabelCellModel
An `NSObject` that contains information on how each cell will be presented. First we have `NSString *string`, which contains the text that will go inside the label. Then we have `BOOL expanded`, which is used to track whether the cell has been expanded already. This is a very important class because it controls how many lines the cell will display via `kMaxNumberOfLines`.

#### ViewController
The view controller is pretty standard. We populate the cell models with random sized strings, implement `UITableViewDelegate` / `UITableViewDataSource`, and just display the cells! Most of the actual logic is handled by `ExpandableLabelCellModel` and `ExpandableLabel`.

#### ExpandableLabel
The most important part of the project and why you came to see this blog post! We will talk about how this is implemented step-by-step below, but I'll summarize the most important parts here. First we must decide whether we need to truncate the cell, if we decide to truncate, we append *"...More"* to the cut off text. Truncation is decided by the labels `numberOfLines` property. If the value is `0` it means that we do not want to truncate the label. If the value `>0` we must calculate whether the `text` of the `UILabel` fits the `numberOfLines`. If the text fits, we want to leave it alone and display it in its full form, otherwise we truncate it. 


## ExpandableLabel Functionality

#### Let's truncate!

The `-(void)truncate` function first checks whether we should truncate the text in the label. If it decides not to truncate, we don't do anything! Otherwise, the function decides how many characters fit the label via `numberOfCharactersThatFitLabel` function. Once we know that number, we just do simple truncating math, append *"...More"*, add some `NSAttributedString` attributes and set the label text!

{% highlight objective-c %}
static NSString *const kExpandableLabelText = @"More";

- (void)truncate {
    
    self.lineBreakMode = NSLineBreakByClipping;

    if ([self shouldTruncate]) {
        
        // Truncate the original string to 'self.numberOfLines' and append a special suffix
        NSInteger numberOfCharactersThatFitLabel = [[self numberOfCharactersThatFitLabel] integerValue];

        NSString *specialSuffix = [NSString stringWithFormat:@" ...%@", kExpandableLabelText];
        NSString *truncatedOriginalString = [self.text substringToIndex:numberOfCharactersThatFitLabel-specialSuffix.length];
        NSString *truncatedNewString = [truncatedOriginalString stringByAppendingString:specialSuffix];

        NSMutableAttributedString *attributedTruncatedNewString = [[NSMutableAttributedString alloc] initWithString:truncatedNewString attributes:nil];
        
        // Re-apply all of the attributes of the original string
        [self.attributedText enumerateAttributesInRange:NSMakeRange(0, numberOfCharactersThatFitLabel) options:NSAttributedStringEnumerationReverse usingBlock:
         ^(NSDictionary *attributes, NSRange range, BOOL *stop) {
             [attributedTruncatedNewString setAttributes:attributes range:range];
         }];

        // Apply special attributes to the 'kExpandableLabelText'
        NSRange range = NSMakeRange(truncatedOriginalString.length + specialSuffix.length-kExpandableLabelText.length, kExpandableLabelText.length);
        [attributedTruncatedNewString addAttribute:NSFontAttributeName value:self.font range:range];
        [attributedTruncatedNewString addAttribute:NSForegroundColorAttributeName value:[UIColor blueColor] range:range];
        
        // Set the new string
        [self setAttributedText:attributedTruncatedNewString];
    }
}
{% endhighlight %}

#### When to truncate?
The project calls `truncate` in `layoutSubviews`. Whether this is the right place to call it can be arguable, but there is one **very important** thing to keep in mind: the calculations are done based off the `UILabel` bounds, so make sure they are correct before calling the truncate function!

#### Should we be truncating?
As stated above, truncation is decided by the labels `numberOfLines` property. If the value is `0`, it means that we do not want to truncate the label. If the value `>0`, we must calculate whether the `text` of the `UILabel` fits the `numberOfLines`. First we calculate the `actualHeightOfText`: the height of the `UILabel` if we were to display <u>every</u> line of text. Then we calculate the `desiredHeightOfText`: the height of the `UILabel` with respect to the value of `numberOfLines`. If `desiredHeightOfText < actualHeightOfText` we want to truncate!

{% highlight objective-c %}
- (BOOL)shouldTruncate {

    if (self.numberOfLines == 0) {
        return NO;
    }
    
    // Calculate the height of the text with no truncation
    CGSize sizeOfText = [self.text boundingRectWithSize:CGSizeMake(self.bounds.size.width, CGFLOAT_MAX)
                                                options:NSStringDrawingUsesLineFragmentOrigin
                                             attributes:[NSDictionary dictionaryWithObject:self.font forKey:NSFontAttributeName] context:nil].size;
    CGFloat actualHeightOfText = sizeOfText.height;
    
    // Calculate the height of the text bound to 'self.numberOfLines'
    CGFloat desiredHeightOfText = [self textRectForBounds:self.bounds limitedToNumberOfLines:self.numberOfLines].size.height;
    
    return desiredHeightOfText < actualHeightOfText;
}
{% endhighlight %}

#### How many characters fit a rectangle? 
The most important function in the whole project! Apple's **Core Text** library provides `CTFramesetterSuggestFrameSizeWithConstraints` function. We can pass a `CFRange` to it called the `fitRange` (*"On return, contains the range of the string that actually fit in the constrained size."*). By passing in the <u>desired</u> rectangle of the `UILabel` and by taking the `length` from the `fitRange`, we get the number of characters that fit our desired label!

{% highlight objective-c %}
- (NSInteger)numberOfCharactersThatFitLabel {
    
    // Create an 'CTFramesetterRef' from an attributed string
    CTFontRef fontRef = CTFontCreateWithName((CFStringRef)self.font.fontName, self.font.pointSize, NULL);
    NSDictionary *attributes = [NSDictionary dictionaryWithObject:(__bridge id)fontRef forKey:(id)kCTFontAttributeName];
    CFRelease(fontRef);
    NSAttributedString *attributedString  = [[NSAttributedString alloc] initWithString:self.text attributes:attributes];
    CTFramesetterRef frameSetterRef = CTFramesetterCreateWithAttributedString((CFAttributedStringRef)attributedString);

    // Get a suggested character count that would fit the attributed string
    CFRange characterFitRange;
    CTFramesetterSuggestFrameSizeWithConstraints(frameSetterRef, CFRangeMake(0,0), NULL, CGSizeMake(self.bounds.size.width, self.numberOfLines*self.font.lineHeight), &characterFitRange);
    CFRelease(frameSetterRef);
    return (NSInteger)characterFitRange.length;
}
{% endhighlight %}


## Conclusion

And that's it! Keep in mind that this functionality can be applied to many different UX cases. For example, Facebook's *"... Continue Reading"* button.