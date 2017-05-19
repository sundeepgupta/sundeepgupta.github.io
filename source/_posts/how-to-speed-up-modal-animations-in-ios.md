---
title: How to speed up modal animations in iOS
tags:
  - ios
id: 104
categories:
  - Uncategorized
date: 2015-01-28 21:26:20
---

Apple doesn't provide an easy way to speed up the animations for presenting modals. In this post, I describe a straightforward way to do this using the `UIViewControllerAnimatedTransitioning` and `UIViewControllerAnimatedTransitioningDelegate` protocols.

<!--more-->

## Problem

Recently, for an iOS project I was working on, I needed to speed up the modal animation and alter its duration curve to match a different custom animation somewhere else in the app. Unfortunately, the built-in modal segue or `presentViewController:animated:completion:` doesn't provide any opportunity to modify the animation.

My first thought was to get my custom animation to match Apple's built-in modal animation behaviour. But how? It's not published anywhere. I could brute force match it by trial and error, but then app would have a dependency on Apple. If Apple were to change their stock animation in the future, I'd run into the same problem.

## Solution

It was clear that I needed to build my own modal animation so I could have full control without any dependencies. The solution uses the iOS 7+ custom animation protocols: `UIViewControllerAnimatedTransitioning` and `UIViewControllerAnimatedTransitioningDelegate`. There's plenty of literature about how they work. If you're interested in digging in, here is a good point: [objc.io - View Controller Transitions](http://www.objc.io/issue-5/view-controller-transitions.html). At the bottom of that article, there are more links, including reference to the WWDC video.

### Example

From a birds eye, here is how things work:

*   From the presenting view controller, initialize and configure the animator.
*   Configure the modal and present/dismiss it with animation set to `YES`. If you don't turn the animation on, the animator will not be used.*   Provide the animator to the `UIViewControllerTransitioningDelegate` delegate methods.
*   In the animator, configure and animate your views.

#### Code

Here is some code that illustrates how to do this. Note that I've left out the unrelated bits.

##### Presenting View Controller

    // PresentingViewController.m

    #import "ModalAnimator.h"

    @interface PresentingViewController () &lt;UIViewControllerTransitioningDelegate&gt;
    @property (strong, nonatomic) ModalAnimator *modalAnimator;
    @end

    @implementation PresentingViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        self.modalAnimator = [ModalAnimator new];
    }

    // Storyboard "Present Modally" seque with "Animates" turned on.
    - (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
        UIViewController *viewController = segue.destinationViewController;    
        viewController.modalPresentationStyle = UIModalPresentationCustom;
        viewController.transitioningDelegate = self;
    }

    #pragma mark - UIViewControllerTransitioningDelegate methods
    - (id&lt;UIViewControllerAnimatedTransitioning&gt;)animationControllerForPresentedController:(UIViewController *)presented presentingController:(UIViewController *)presenting sourceController:(UIViewController *)source {
        [self.modalAnimator shouldPresent];
        return self.modalAnimator;
    }

    - (id&lt;UIViewControllerAnimatedTransitioning&gt;)animationControllerForDismissedController:(UIViewController *)dismissed {
        [self.modalAnimator shouldDismiss];
        return self.modalAnimator;
    }

    @end
    `</pre>

    Here you can see the general flow. In this example, a storyboard segue is used to present the modal. Here is how the segue is configured in the storyboard:

    ![modal segue settings](http://i.imgur.com/Gohw9gX.png)

    Alternatively, it could have been presented in code like so:

    <pre>`viewController.modalPresentationStyle = UIModalPresentationCustom;
    viewController.transitioningDelegate = self;
    [self presentViewController:viewController animated:YES completion:nil];
    `</pre>

    ##### Custom Animator

    <pre>`// ModalAnimator.h

    @interface ModalAnimator : NSObject &lt;UIViewControllerAnimatedTransitioning&gt;
    - (void)shouldPresent;
    - (void)shouldDismiss;
    @end
    `</pre>

    The public interface is very simple. Only a couple of configuration setters to let the animator know if we should be using the present or the dismiss animation. As seen above in the `PresentingViewController.m` code, these get called in the `UIViewControllerTransitioningDelegate` methods.

    <pre>`// ModalAnimator.m

    @interface ModalAnimator ()
    @property (nonatomic) BOOL isPresenting;
    @end

    @implementation ModalAnimator

    - (instancetype)init {
        self = [super init];
        if (self) {
            self.isPresenting = YES;
        }
        return self;
    }

    - (void)shouldPresent {
        self.isPresenting = YES;
    }

    - (void)shouldDismiss {
        self.isPresenting = NO;
    }

    #pragma mark - UIViewControllerAnimatedTransitioning methods
    - (NSTimeInterval)transitionDuration:(id&lt;UIViewControllerContextTransitioning&gt;)transitionContext {
        return 0.2;
    }

    - (void)animateTransition:(id&lt;UIViewControllerContextTransitioning&gt;)transitionContext {
        UIViewController *fromVc = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
        UIViewController *toVc = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
        UIView *fromView = fromVc.view;
        UIView *toView = toVc.view;
        UIView *containerView = transitionContext.containerView;
        NSTimeInterval duration = [self transitionDuration:transitionContext];

        if (self.isPresenting) {
            UIViewController *topFromVc = [self topViewControllerForViewController:fromVc];
            [topFromVc viewWillDisappear:YES];
            fromView.userInteractionEnabled = NO;
            [containerView addSubview:toView];

            // Setup animating the view from the bottom of the screen. 
            CGRect originalFrame = toView.frame;
            CGRect hiddenFrame = CGRectMake(originalFrame.origin.x, originalFrame.size.height, originalFrame.size.width, originalFrame.size.height);
            toView.frame = hiddenFrame;

            [UIView animateWithDuration:duration animations: ^{
                toView.frame = originalFrame;
            } completion: ^(BOOL finished) {
                [transitionContext completeTransition:![transitionContext transitionWasCancelled]];
                [topFromVc viewDidDisappear:YES];
            }];
        } else {
            UIViewController *topToVc = [self topViewControllerForViewController:toVc];
            [topToVc viewWillAppear:YES];

            CGRect hiddenFrame = CGRectMake(fromView.frame.origin.x, fromView.frame.size.height, fromView.frame.size.width, fromView.frame.size.height);

            [UIView animateWithDuration:duration animations: ^{
                fromView.frame = hiddenFrame;
            } completion: ^(BOOL finished) {
                [fromView removeFromSuperview];
                toView.userInteractionEnabled = YES;
                [transitionContext completeTransition:![transitionContext transitionWasCancelled]];
                [topToVc viewDidAppear:YES];
            }];
        }
    }

    #pragma mark - Private methods
    - (UIViewController *)topViewControllerForViewController:(UIViewController *)viewController {
        if ([viewController isKindOfClass:[UINavigationController class]]) {
            UINavigationController *navigationController = (UINavigationController *)viewController;
            return navigationController.topViewController;
        } else {
            return viewController;
        }
    }

    @end

The animator essentially implements two delegate methods which describe the animation and its duration. A couple of things to take note of:

1.  To preserve the way the default modal presentation works, manually call view lifecycle methods.

2.  The private method `topViewControllerForViewController:` is here to ensure that the view life cycle methods are called on the correct view controller.

## Wrap-Up

I ended up using this modal animator in a few places in my app, and now it looks and feels consistent. If this has helped you or have different or better ways to speed up modal animations in iOS, feel free to comment.

Happy exploring!