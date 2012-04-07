---
layout: post
title: Core Data with a Single Shared UIManagedDocument
excerpt: UIManagedDocument is a great way to set up a Core Data stack for your application. With application delegates taking a smaller role in newer applications, sharing the document's managed object context can be tricky.
---

UIManagedDocument is a great way to set up a Core Data stack for your application. It's also the best way to enable iCloud support for your Core Data applications. For applications with a single view controller as the entry point, setting up the document and passing it on to subsequent view controllers works great. But if your application has multiple entry points, or a single entry point you cannot modify (such as a UITabBarController), it can be a little less straightforward.

While re-writing an existing app to use many of the new features in iOS 5, I ran into this very issue. The application used a tab bar controller as the entry point and my previous experience with letting the first view controller set up the managed document was not going to work. How could I provide access to the document, while insuring it was in the correct state, to all the view controllers in the app? I guess I could have set it up in the application delegate and then grabbed a reference to the tab bar controller through the window and then used `respondsToSelector:` to figure out which controllers had a document property and...

Blech.

One thing I really like about writing apps for iOS 5 is how little the application delegate is responsible for now. Previously it had become a dumping ground for setting up nearly everything for an application while handling application events and pretty much everything else that didn't fit into a view controller. Using it for setting up my managed document seemed like a step backward. Why not create a new class? A global singleton whose only job is to handle and provide a convenient interface to the managed document?

	#import <Foundation/Foundation.h>
	
	typedef void (^OnDocumentReady) (UIManagedDocument *document);
	
	@interface MYDocumentHandler : NSObject
	
	@property (strong, nonatomic) UIManagedDocument *document;

	+ (MYDocumentHandler *)sharedDocumentHandler;
	- (void)performWithDocument:(OnDocumentReady)onDocumentReady;
	
	@end
	
There. Just one instance method. Define a block that takes a UIManagedDocument as it's only argument and feel secure that it will only be executed once the document has been successfully loaded (if necessary).

Looking inside:

	#import "MYDocumentHandler.h"
	
	@interface MYDocumentHandler ()
	- (void)objectsDidChange:(NSNotification *)notification;
	- (void)contextDidSave:(NSNotification *)notification;
	@end;
	
	@implementation MYDocumentHandler
	
	@synthesize document = _document;
	
	static MYDocumentHandler *_sharedInstance;
	
	+ (MYDocumentHandler *)sharedDocumentHandler
	{
	    static dispatch_once_t once;
	    dispatch_once(&once, ^{
	        _sharedInstance = [[self alloc] init];
	    });
	    
	    return _sharedInstance;
	}
	
	- (id)init
	{
	    self = [super init];
	    if (self) {
	        NSURL *url = [[[NSFileManager defaultManager] URLsForDirectory:NSLibraryDirectory inDomains:NSUserDomainMask] lastObject];
	        url = [url URLByAppendingPathComponent:@"MyDocument.md"];
	        
	        self.document = [[UIManagedDocument alloc] initWithFileURL:url];
	        
	        // Set our document up for automatic migrations
	        NSDictionary *options = [NSDictionary dictionaryWithObjectsAndKeys:
	                                 [NSNumber numberWithBool:YES], NSMigratePersistentStoresAutomaticallyOption,
	                                 [NSNumber numberWithBool:YES], NSInferMappingModelAutomaticallyOption, nil];
	        self.document.persistentStoreOptions = options;
	        
	        // Register for notifications
	        [[NSNotificationCenter defaultCenter] addObserver:self
	                                                 selector:@selector(objectsDidChange:)
	                                                     name:NSManagedObjectContextObjectsDidChangeNotification
	                                                   object:self.document.managedObjectContext];
	        
	        [[NSNotificationCenter defaultCenter] addObserver:self
	                                                 selector:@selector(contextDidSave:)
	                                                     name:NSManagedObjectContextDidSaveNotification
	                                                   object:self.document.managedObjectContext];
	    }
	    return self;
	}
	
	- (void)performWithDocument:(OnDocumentReady)onDocumentReady
	{    
	    void (^OnDocumentDidLoad)(BOOL) = ^(BOOL success) {
	        onDocumentReady(self.document);
	    };
	    
	    if (![[NSFileManager defaultManager] fileExistsAtPath:[self.document.fileURL path]]) {
	        [self.document saveToURL:self.document.fileURL
	           forSaveOperation:UIDocumentSaveForCreating
	          completionHandler:OnDocumentDidLoad];
	    } else if (self.document.documentState == UIDocumentStateClosed) {
	        [self.document openWithCompletionHandler:OnDocumentDidLoad];
	    } else if (self.document.documentState == UIDocumentStateNormal) {
	        OnDocumentDidLoad(YES);
	    }
	}
	
	- (void)objectsDidChange:(NSNotification *)notification
	{
	#ifdef DEBUG
	    NSLog(@"NSManagedObjects did change.");
	#endif
	}
	
	- (void)contextDidSave:(NSNotification *)notification
	{
	#ifdef DEBUG
	    NSLog(@"NSManagedContext did save.");
	#endif
	}
	
	@end

Pretty simple. First we setup our singleton using `dispatch_once`. Then on initialization we define our document, set up the persistent store options and register for a couple notifications (optional, of course). When the document is needed (when `performWithDocument:` is called) we look to see if the document exists, or is closed, creating it or opening it as required. Once we've successfully loaded our document we execute the block we've received, passing in our now-ready document. Now we have a nice way of accessing our document:

	#import "MYDocumentHandler.h"
	
	@implementation MyViewController
	
	- (void)viewWillAppear:(BOOL)animated
	{
	    [super viewWillAppear:animated];
	    
	    if (!self.document) {
	        [[MYDocumentHandler sharedDocumentHandler] performWithDocument:^(UIManagedDocument *document) {
	            self.document = document;
	            // Do stuff with the document, set up a fetched results controller, whatever.
	        }];
	    }
	}
	
	@end
	
You can find me on [Twitter](http://twitter.com/jdriscoll) (@jdriscoll).








