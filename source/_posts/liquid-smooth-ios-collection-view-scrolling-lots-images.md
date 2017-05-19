---
title: Liquid smooth iOS collection view scrolling with lots of images
tags:
  - ios
  - performance
id: 177
categories:
  - Uncategorized
date: 2016-01-10 21:53:30
---

An implementation of how I achieved smooth scrolling with lots of images that needed dynamic sizing and tinting.

<!--more-->

## tl;dr

Skip to the **[Solution](#solution)** section for the implementation which includes all the code examples.

## Problem

On a recent project, I had an issue with choppy scrolling in a couple of collection views which held a grid of images. It wasn't horrible, but noticeable enough, especially on slower phones.

### Original implementation

The app has a domain object called a "cue". Each cue had an image associated with it and we wanted to render cues in the "library". Because the cues were static, they were placed in the asset catalog.

    // CCLibraryViewController
    - (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath {
        CCCueCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:NSStringFromClass([CCCueCell class]) forIndexPath:indexPath];
        CCCue *cue = self.cues[indexPath.item];
        [cell configureWithImage:[UIImage imageNamed:cue.name]];
        return cell;
    }

    // CCCueCell
    - (void)configureWithImage:(UIImage *)image {
        self.imageView.image = image;
        [CCHelper tintImageView:self.imageView withColor:kGreyColor];
    }

    // CCHelper
    + (void)tintImageView:(UIImageView *)imageView withColor:(UIColor *)color {
        UIImage *tintableImage = [imageView.image imageWithRenderingMode:UIImageRenderingModeAlwaysTemplate];
        imageView.image = tintableImage;
        imageView.tintColor = color;
    }
    `</pre>

    As you can see, everything is pretty straightforward here.

    ### Sidebar:

    I've learned since, that you can actually set your images' rendering mode directly in the asset catalog GUI:

    ![Rendering mode for asset catalog images](http://content.screencast.com/users/SundeepG/folders/Do%20not%20delete/media/ce8c5e04-7847-4b32-8ba2-ba22c45cc1dd/00000154.png)

    ## Diagnosis

    It turns out there were 3 implementation details that contributed to the performance issue.

1.  Tinting
2.  `UIImage imageNamed:`
3.  Asset Catalog

    So, basically the entire implementation :)

    ### Tinting

    I won't get into why tinting was required, but there was a work around by providing natively sets of natively tinted images. It certainly helped, but there was still a bit of scroll lag.

    ### `UIImage imageNamed:`

    If you do a search for [UIImage imageNamed performance](https://duckduckgo.com/?q=uiimage%20imagenamed%20performance), you'll get a lot of hits discussing problems with how Apple implements this. The key thing for our particular problem was that when an image instantiated this way is rendered by setting on a UIImageView, and because this must be done on the main (UI) queue, it causes a performance lag. It does this because the image must be decompressed at render time.

    ### Asset Catalog

    Based on the issues with `UIImage imageNamed:`, using the Asset Catalog was no longer an option. This wasn't really causing anything actually, but I prefer to use the Asset Catalog when I can.

    ## Solution

    The goal was to get perfectly liquid smooth scrolling every time, even on the first scroll. To achieve this, I implemented a caching strategy that I pieced together from my research on the issue. In a nutshell:

1.  Place the images in the bundle, not the Asset Catalog.
2.  When the app loads, start caching asynchronously.
3.  Instantiate images using `UIImage imageWithContentsOfFile:` decompressing them and tint it as needed.
4.  Wrap them in NSPurgeableData instances.
5.  Use the images from the cache if available, otherwise create them and and cache them.

    Let's go through all of this with actual code examples.

    ### The cache

    I used a simple NSCache object to provide automatic memory managed caching. Much preferred over NSDictionary. You can [read more about NSCache here](http://nshipster.com/nscache/).

    <pre>`- (void)cacheSystemCueImages {
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0), ^{
            for (CCSystemCue *cue in self.systemCues) {
                NSString *key = [cue thumbnailCacheKey];
                if ([self.systemCueImageCache objectForKey:key]) {
                    continue;
                }

                CCDiscardableImage *discardable = [CCHelper decompressedImageForPath:[cue imagePath] size:[cue thumbnailSize] tintable:[cue tintable]];
                [self.systemCueImageCache setObject:discardable forKey:key];
            }
        });
    }
    `</pre>

    Here you can see the caching done on a background queue so it doesn't block the UI. Each image is stored along with a unique key, used to retrieve it later. I'll get into what a CCDiscardableImage in a bit.

    ### Instantiating the images

    We need to use `UIImage imageWithContentsOfFile:` to avoid the pitfalls of `UIImage imageNamed:`. Namely, don't let the OS do the caching for us. We want to fully control how we cache our images, specifically, already decompressed and tinted, so when we're on the main queue, it's only a matter of rendering a pre-processed image.

    <pre>`+ (CCDiscardableImage *)decompressedImageForPath:(NSString *)path size:(CGSize)size tintable:(BOOL)tintable {
        UIImage *image = [UIImage imageWithContentsOfFile:path];
        return [self decompressedImage:image forSize:[self screenScaledSize:size] tintable:tintable];
    }

    + (CGSize)screenScaledSize:(CGSize)size {
        CGFloat scale = [UIScreen mainScreen].scale;
        return CGSizeMake(size.width*scale, size.height*scale);
    }

    + (CCDiscardableImage *)decompressedImage:(UIImage *)image forSize:(CGSize)size tintable:(BOOL)tintable {
        UIGraphicsBeginImageContext(size);
        [image drawInRect:CGRectMake(0, 0, size.width, size.height)];
        image = UIGraphicsGetImageFromCurrentImageContext();
        UIGraphicsEndImageContext();

        if (tintable) {
            image = [image imageWithRenderingMode:UIImageRenderingModeAlwaysTemplate];
        }

        return [[CCDiscardableImage alloc] initWithImage:image];
    }
    `</pre>

    For us to store a fully decompressed image in the cache, which is what will help our performance, we need to provide the size at which it will be rendered at which is done by rendering it to a UIGraphicsImageContext and snapping an image from that.

    ### CCDiscardableImage: NSPurgeableData

    NSPurgeableData from the horse's mouth:

    > You should use the NSPurgeableData class when you have objects with bytes that can be discarded when no longer needed.

    You might be wondering why we'd need to do this since NSCache already manages itself. Well, it does, but in my testing, it doesn't behave optimally. If we used NSCache directly with UIImages, the cache would fill up. But if I backgrounded the app, and then foregrounded it again, the cache would be empty and have fill up again. To avoid this, I wrapped the images in NSPurgeableData:

    <pre>`#import &lt;UIKit/UIKit.h&gt;

    @interface CCDiscardableImage : NSPurgeableData
    @property (readonly, strong, nonatomic) UIImage *image;
    - (instancetype)initWithImage:(UIImage *)image;
    @end
    #import "CCDiscardableImage.h"

    @interface CCDiscardableImage()
    @property (strong, nonatomic) UIImage *image;
    @end

    @implementation CCDiscardableImage
    - (instancetype)initWithImage:(UIImage *)image {
        if (self = [super init]) {
            _image = image;
        }
        return self;
    }
    @end
    `</pre>

    ### Using the image

    I try to get the image from the cache first. If it's not there, I go through the same process as before to create it.

    <pre>`- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath {
        NSString *reuseIdentifier = NSStringFromClass([CCCueCell class]);
        CCCueCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:reuseIdentifier forIndexPath:indexPath];
        CCSystemCue *cue = self.cues[indexPath.item];

        NSString *key = [cue thumbnailCacheKey];
        BOOL isSelected = [self isSelectedIndexPath:indexPath];
        CCDiscardableImage *cachedDiscardable = [self.cueImageCache objectForKey:key];
        if (cachedDiscardable) {
            [cell configureWithImage:cachedDiscardable.image isSelected:isSelected];
            return cell;
        }

        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0), ^{
            CCDiscardableImage *newDiscardable = [CCHelper decompressedImageForPath:[cue imagePath] size:[cue thumbnailSize] tintable:[cue tintable]];
            [self.cueImageCache setObject:newDiscardable forKey:key];

            dispatch_async(dispatch_get_main_queue(), ^{
                if ([self isVisibleIndexPath:indexPath]) {
                    [cell configureWithImage:newDiscardable.image isSelected:isSelected];
                }
            });
        });

        return cell;
    }

    - (void)configureWithImage:(UIImage *)image isSelected:(BOOL)isSelected {
        self.imageView.image = image;
        isSelected ? [self addBorder] : [self removeBorder];
    }

If we need to create the image, we do that in the background again to not lock up the UI. But remember, we need to actually _render_ the image on the main queue, otherwise, the rendering will be delayed. Also, the we really only need to render the image if the cell is still visible on the screen. If the user is scrolling super fast, the cell might be hidden at this point in time, in which case, no need to render it. A simple additional optimization.

## Conclusion

The resulting implementation worked out well. There was zero scroll lag because all the heavy lifting was now done in the background. And this was because we were able to cache fully decompressed images by not using `UIImage imageNamed:`. The worse case scenario was if the user was very speedy by opening the library view and immediately scrolling down fast. This would result in some cells appearing blank for a fraction of a second. But the scrolling would remain liquid smooth. I hope this post or perhaps one bit of it was helpful for you.

Happy exploring!