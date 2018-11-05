# Intersection Observer V2 Explained

## Background: IntersectionObserver V1

[IntersectionObserver](https://w3c.github.io/IntersectionObserver/) is an API that enables push notifications whenever the state of the geometric intersection between an Element and a selected ancestor changes. Among the primary use cases for IntersectionObserver are measuring ad impressions; lazy-loading content in the "below the fold" portion on a page; and implementing infinite-scrolling lists.

## What's Missing from V1?

V1 of the API can tell you when an Element is scrolled into the window's viewport, but it doesn't tell you whether the Element is covered by any other page content ("occluded"); or whether the Element's visual display has been modified by any visual effects (transform, opacity, filter, etc.). For an Element in the top-level document, that information can be determined by analyzing the DOM in javascript, or via elementFromPoint(). But for an Element inside an iframe, the information is unobtainable by any means.

## What's in V2?

V2 of the API adds the ability to track the "visibility" of a target Element. By setting an option in the IntersectionObserver constructor, push notifications will contain a new boolean field named isVisible. A "true" value for isVisible is a strong guarantee from the implementation that the target Element is completely unoccluded by other content and has no visual effects that would alter or distort its display on screen. A "false" value means that the implementation cannot make that guarantee.

An important detail of the spec is that the implementation is permitted to report false negatives (i.e., setting isVisible to "false" even when the target Element is completely visible and unmodified), but it may *never* report a false positive. For performance or other reasons, an implementation may use an algorithm which is not 100% accurate, as long as all inaccuracies are false negatives.

## Why V2?

The primary purpose of V2 is to be a powerful weapon against some of the most pervasive forms of fraud on the web -- in particular, click-jacking and other UI redress attacks. Commonly, a malicious site will use a variety of visual design techniques to manipulate users into unintentionally clicking into a third-party iframe (e.g., an ad, or a "like" button, or a payment widget). When the iframe processes the click event, it has no way to determine that its content was not faithfully displayed on the screen. Using IntersectionObserverV2, code running inside the iframe can get a strong guarantee from the implementation that its content was completely visible and unmodified for some minimum length of time before the click.

## Advantages of V2 for Fraud Prevention

There are two key aspects that make IntersectionObserver V2 a powerful tool for fraud prevention:

* Because the IntersectionObserver code can run in the context of the iframe, it does not rely on the embedding context -- which may be malicious -- for anything. The iframe only needs to trust the browser implementation. V2 puts all of the power into the hands of the iframe.
    
* Most current strategies for combatting UI redress attacks consist of crawling the web and scanning the rendered content of pages. Typically, this requires lots of computing power and sophisticated analyses. At best, such techniques will identify and black-list fraudulent sites, but only after they have been live on the web for some time, and likely defrauded some number of users. By contrast, IntersectionObserver V2 makes it possible to detect and **stop fraud before it occurs**, with code running entirely on the client.
