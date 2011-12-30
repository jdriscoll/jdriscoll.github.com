---
layout: post
title: NSManagedDocument, Nested Contexts and Notifications
excerpt: NSManagedDocument is great. It wraps up an entire core data stack into a nice little package. But like most new APIs, there are a few little details that can trip you up if you're not careful.
---

NSManagedDocument is great. It wraps up an entire core data stack into a nice little package. But like most new APIs, there are a few little details that can trip you up if you're not careful.

NSManagedDocument uses a nested managedObjectContext. For the most part this is an implementation detail, and not something you need to worry about. When you need a context, grab the one from your document and everything will be fine (assuming single threaded code of course). But one case where it might trip you up, as it did me, is in code that relies on the NSManagedObjectContextObjectsDidChangeNotification that NSManagedObjectContext provides.

There are two things to keep in mind when using NSManagedObjectContextObjectsDidChangeNotification with NSManagedDocument. For starters, because it uses a nested context, if you do not specify a context when you subscribe to the notification, you will receive two notifications for every event. Once from the document's context, and once from the parent context when the changes are passed along. If you happen to be working with a tableview this can cause lots of fun little internal inconsistency exceptions. The second thing to remember is that NSManagedDocument loads asynchronously. If you call `NSNotificationCenter addObserver:selector:name:object:` in viewDidLoad, or any where else before your document is ready, and pass in your document's managedObjectContext as the object parameter, that value is going to be nil and you will still get two notifications for the price of one.

The solution is to register your observer within the block passed to either `NSManagedDocument saveToUrl:forSaveOperation:completionHandler` or `NSManagedDocument openWithCompletionHandler:`. This way you can be sure your document will return a valid context and your notification handler will only be called once for every event.

Here's a example:

	// Here we define the block that will be executed whenever
	// our document is ready to use:
	void (^OnDocumentDidLoad)(BOOL) = ^(BOOL success) {
        // Load some managed objects, reload your tableview, etc.
        [[NSNotificationCenter defaultCenter] addObserver:self
                                                 selector:@selector(objectsDidChange:)
                                                     name:NSManagedObjectContextObjectsDidChangeNotification
                                                   object:self.document.managedObjectContext];
    };
    
	// Now we check to see if our document exists, is closed, or is 
    // already open and ready to work with:	
    if (![[NSFileManager defaultManager] fileExistsAtPath:[self.document.fileURL path]]) {
        [self.document saveToURL:self.document.fileURL
                forSaveOperation:UIDocumentSaveForCreating
               completionHandler:OnDocumentDidLoad];
    } else if (self.document.documentState == UIDocumentStateClosed) {
        [self.document openWithCompletionHandler:OnDocumentDidLoad];
    } else if (self.document.documentState == UIDocumentStateNormal) {
        OnDocumentDidLoad(YES);
    }

Cheers!

*Updated Dec 30, 2011*

The best introduction to NSManagedDocument I've found so far is in lectures 13 and 14 from [Stanford's iPhone and iPad development course](http://itunes.apple.com/WebObjects/MZStore.woa/wa/viewPodcast?id=473757255). Available for free on iTunes U.
