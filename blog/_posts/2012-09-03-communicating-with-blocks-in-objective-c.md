---
layout: post
published: true
title: Communicating with Blocks in Objective-C
excerpt: I love blocks because blocks make Objective-C much more expressive. They can also reduce the amount of code you need to write, which reduces the amount of code you need to maintain and debug. Any developer who has ever worked in a higher level language like Ruby, Python or Javascript should feel right at home using blocks. Once they get past the awkward syntax at least.
---

Let’s talk about [blocks](http://developer.apple.com/library/ios/#featuredarticles/Short_Practical_Guide_Blocks/_index.html).

I love blocks because blocks make Objective-C much more expressive. They can also reduce the amount of code you need to write, which reduces the amount of code you need to maintain and debug. Any developer who has ever worked in a higher level language like Ruby, Python or Javascript should feel right at home using blocks. Once they get past the awkward syntax at least.

One area where I’ve personally seen a lot of benefit from using blocks is in setting up communication between separate systems. This could be communication between an app and a web service or between separate concerns within a single app. Communication with blocks helps maintain a high level of encapsulation while keeping your code readable and concise.

Traditionally, we might set up communication between two objects using a [delegate](http://developer.apple.com/library/ios/#documentation/General/Conceptual/CocoaEncyclopedia/DelegatesandDataSources/DelegatesandDataSources.html). Delegates are used througout Cocoa and Cocoa Touch to great effect and they’re virtually impossible to avoid. In iOS, the application delegate is one of the first objects we interact with. Delegates work well when two objects need to have an ongoing or broad relationship and the UITableView delegate is a great example of this.

In other cases we might just want to be able to pass a little information around between objects without them necessarily knowing that much about each other. While we can get by defining a delegate protocol, implementing the delegate methods on our object and assigning the delegate to a property, using blocks we can accomplish the same thing while avoiding formal protocols and keeping the related code closer together.

Let’s say we want to build an app that fetches and displays stock quotes. Because we’re good little programmers we’ve decided to separate, or encapsuate, the logic that actually communicates with whatever web service we’re using and provides a common interface or API for the rest of our app to use.

If you’re new to this stuff (or if you use PHP) you might wonder why we would do this. Well for one, in the future we might want, or need, to switch the service that provides our stock quotes. If we wrote directly to the original service’s API in our app and the new service is not compatible with the original one we would need to update our application in any number of different places to make this change. But with the service API safely abstracted away behind an internal API of our own devising we would need only to modify one part of the app. This is called [Separation of Concerns](http://en.wikipedia.org/wiki/Separation_of_concerns), and it’s a wonderful thing.

Let’s assume we’ve got our Xcode project set up. You can even download [the complete project from Github](http://github.com/jdriscoll/ads-sample-stock-price-fetcher) if you want. Why don’t we first define the class that will handle all communication with the stock API we’ll be using. This class will provide the internal API that our application will interact with.

_Note: For this example I’m using the undocumented but public Yahoo! stock quote API the details of which are beyond this post but available elsewhere on the internet._

{% highlight objc %}
//  SPFPriceFetcher.h

#import <Foundation/Foundation.h>

typedef void (^SPFQuoteRequestCompleteBlock) (BOOL wasSuccessful, NSDecimalNumber *price);

@interface SPFPriceFetcher : NSObject

- (void)requestQuoteForSymbol:(NSString *)symbol
                 withCallback:(SPFQuoteRequestCompleteBlock)callback;

@end
{% endhighlight %}

Very simple. Just one method that takes a stock symbol and a callback block. As you can see here I've used a typedef to define the callback. In many ways this is like defining an informal protocol. It's not necessary and you can define the block inline with the method but in this case I think it aids in readability and it can be helpful when defining a callback as we'll see later.

Here's the implementation of our PriceFetcher class.

{% highlight objc %}
//  SPFPriceFetcher.m

#import "SPFPriceFetcher.h"
#import "JCDHTTPConnection.h"

// Yahoo stock quote API
// Example: http://download.finance.yahoo.com/d/quotes.csv?s=GOOG&f=l1
#define kYahooStockQuoteAPIURL @"http://download.finance.yahoo.com/d/quotes.csv"
#define kYahooStockQuoteAPIFormatString @"l1"

@implementation SPFPriceFetcher

- (void)requestQuoteForSymbol:(NSString *)symbol withCallback:(SPFQuoteRequestCompleteBlock)callback
{
    NSURL *url = [NSURL URLWithString:[NSString stringWithFormat:@"%@?s=%@&f=%@",
                                       kYahooStockQuoteAPIURL,
                                       symbol,
                                       kYahooStockQuoteAPIFormatString]];

    NSURLRequest *request = [NSURLRequest requestWithURL:url];

    JCDHTTPConnection *connection = [[JCDHTTPConnection alloc] initWithRequest:request];
    [connection executeRequestOnSuccess:
     ^(NSHTTPURLResponse *response, NSString *bodyString) {
         if (response.statusCode == 200) {
             NSDecimalNumber *price = [NSDecimalNumber decimalNumberWithString:bodyString];
             callback(YES, price);
         } else {
             callback(NO, nil);
         }
     } failure:^(NSHTTPURLResponse *response, NSString *bodyString, NSError *error) {
         callback(NO, nil);
     } didSendData:nil];
}

@end
{% endhighlight %}

You may notice that I'm keeping this block love-in going strong here by using my [JCDHTTPConnection](http://adevelopingstory.com/blog/2011/11/jcdhttpconnection.html) class which itself provides a block-based API for making HTTP requests using NSURLConnection. Calling blocks from within other blocks can be very powerful and allows you to chain callbacks which is also neat. I'm not going to go into all of this but basically we're just making a simple GET request to the API url and passing in the stock symbol as a parameter. When our request completes we check to make sure it was successful and execute the callback block we've been passed with a BOOL indicating whether we able to fetch the price and, if so, the price we fetched.

With our API proxy in place we can now look at how our application code might use this class. Our app's only view controller (SPFViewController) has a single action that's connected to the "Get Price" button in our UI. This action defines a callback block and then uses an instance of the SPFPriceFetcher class (that we lazily instantiate) to make the actual request. The callback that we pass in only needs to check if we were successful and update the UI accordingly.

{% highlight objc %}
- (IBAction)getPrice:(id)sender {
    SPFQuoteRequestCompleteBlock callback = ^(BOOL wasSuccessful, NSDecimalNumber *price) {
        if (wasSuccessful) {
            self.priceLabel.text = [NSString stringWithFormat:@"Latest price: $%@", [price stringValue]];
        } else {
            self.priceLabel.text = @"Unable to fetch price. Try again.";
        }
    };

    [self.quoter requestQuoteForSymbol:self.stockSymbolTextField.text
                          withCallback:callback];
}
{% endhighlight %}

With our application structured in this way we can easily change the internal workings of our PriceFetcher class without having to modify any other code. And because we've used blocks to implement our API we have the flexiblity to define our callbacks inline, ahead of time, or even pass them around or store them in instance variables.

If you haven't already you can [download the complete project from Github](http://github.com/jdriscoll/ads-sample-stock-price-fetcher).

I can only imagine we'll be seeing more Apple APIs that use blocks in the future. Delegate-based APIs like NSURLConnection are clumsy and begging for a more modern interface. As an exercise, try rewriting our PriceFetcher class using NSURLConnection without JCDHTTPConnection and see how much code it takes (sure you can cheat by using stringWithContentsOfURL but what if we need to make a POST request?). Blocks are a great addition to the Objective C language and they can help us all write better, and less, code.
