# Scroll Anchoring

Scroll anchoring is a proposed
[intervention](https://github.com/WICG/interventions)
that adjusts the scroll position to reduce visible content "jumps".

## Overview

Today, users of the web are often distracted by content moving around due to
changes that occur outside the viewport.  Examples include script inserting an
iframe containing an ad, or non-sized images loading on a slow network.

Historically the browser's default behavior has been to preserve the absolute
scroll position when such changes occur.  This means that to avoid shifting
content, the webpage must take pains to reserve space on the page for anything
that will load later.  In practice, few websites do this consistently.

Scroll anchoring aims to minimize surprising content shifts by scrolling to
follow the movement of visible page elements.

## Design

Scroll anchoring works by selecting a DOM node (the *anchor node*) whose
movement is used to determine the adjustment to the scroll position.

When the anchor node moves, the browser computes its previous location L0 and
its new location L1 in the coordinate space of the scrolling content.  It then
reads the current scroll position S0 and computes a new scroll position S1:

  S1 = S0 + (L1 - L0)

By setting the scroll position to S1 at the same time that the anchor node moves
to L1, we preserve the location of the anchor node relative to the viewport, and
no "jump" is visible.

The anchor node may be either a text node or an element.

Conceptually, a new anchor node is computed whenever the scroll position
changes.  (As a performance optimization, the implementation may wait until the
anchor node is needed before computing it.)

## Anchor Node Selection

We aim to select an anchor node that is deep in the DOM and close to the top
edge of the viewport.  This maximizes the amount of offscreen content whose
changes we will successfully compensate for, while avoiding undesired scrolling
when important content loads within the viewport (for example, upon clicking a
"Read More" link).

The current algorithm to select an anchor node for a scrollable area (either a
document or an element with scrollable overflow) is as follows:

* Walk the DOM within the scrollable area.

* For each element E encountered by the walk,

    - If E is position: fixed, or if E is position: absolute and E's containing
      block is an ancestor of the scrollable area, skip over E and its
      descendents.

    - Otherwise, compare its bounds to the scrollable area's visible region.

        - If E is fully visible, terminate the walk and use E as the anchor
          node.

        - If E is fully clipped (not in the viewport), skip over E and its
          descendents.

        - If E is partially visible, mark it as a "candidate" and descend into
          its children.  If the walk reaches the end of E without finding
          another anchor node, use E as the anchor node.  (We prefer deeper
          nodes in this case to avoid the failure mode of content being inserted
          inside the anchor node but outside the viewport.)

## Opt Out API

Scroll anchoring aims to be the default mode of behavior when launched, so that
users benefit from it even on legacy content.  To allow web developers to
disable scroll anchoring in part or all of a webpage we propose a CSS property
`overflow-anchor` which applies to scrollable elements and supports the
following values:

* `overflow-anchor: visible` causes a scrollable area to follow the content
  within its viewport, using the scroll anchoring algorithm described in this
  proposal.

* `overflow-anchor: none` causes a scrollable area to maintain its absolute
  scroll position when content changes.  This was the default behavior prior to
  the introduction of scroll anchoring.

* `overflow-anchor: auto` invokes the user agent's default behavior for the
  scrollable area.  With the launch of scroll anchoring this will be equivalent
  to `viewport`, but is subject to future modification.

The behavior of the document-level scrollable area is based on the computed
`overflow-anchor` style of the viewport-defining element (`html` or `body`).

The `overflow-anchor` property is also proposed (with different values) for
[CSS Sticky Scrollbars](http://tabatkins.github.io/specs/css-sticky-scrollbars/).
This feature is distinct from scroll anchoring, but shares the notion of
adjusting scroll position in response to content changes.

The `overflow-anchor` property is not inherited.

## Open Issues

#### History Scroll Restoration

Browsing history entries are stored with (absolute) scroll offsets, which are
restored during back/forward navigation.  Typically the offset is recorded after
everything on the page has finished loading, but the restoration is done before
the load completes (since a jump when the load completes would be jarring).

This creates a problem when scroll anchoring makes adjustments during page load,
since we may end up at a different offset by the time the load completes.

In the current architecture we have no way to perform correct scroll restoration
and also avoid visible jumps during page load.

One way to solve this is for the browsing history entry to store something
analogous to a CSS selector for the scroll anchor node, instead of storing an
absolute scroll offset.

#### Scroll Event Handlers

Many pages on the web make changes to the DOM in response to changes in the
scroll position.  For example, they change the layout of the page below a
certain scroll threshold, or they adjust the position of an element so that it
"sticks to" the viewport.

If the DOM changes made by a scroll event handler alter the position of the
anchor node, we easily encounter feedback loops in which scroll anchoring and
the scroll event handler trigger each other ad infinitum.  This is an
undesirable user experience.

## Implementation Status

A prototype of scroll anchoring is behind a flag in Google Chrome and can be
enabled from the "about:flags" page.  The implementation is tracked in
[Issue 558575](http://crbug.com/558575) and its dependencies.

Mozilla has discussed a similar feature in
[Bug 43114](https://bugzilla.mozilla.org/show_bug.cgi?id=43114).
