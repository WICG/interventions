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

## Draft Spec

[Draft spec](https://cdn.rawgit.com/WICG/interventions/master/scroll-anchoring/spec.html)

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

    - If E is `position: fixed`, or if E is `position: absolute` and E's
      containing block is an ancestor of the scrollable area, or if E has
      `overflow-anchor: none` (see Exclusion API), skip over E and its
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

## SANACLAP

Initial testing revealed that scroll anchoring often performed undesired scroll
adjustments in response to changes in the computed style of the anchor node or
one of its ancestors.  For example, a map panning interface might update a CSS
transform in response to mouse movements, or a sticky-header implementation
might adjust padding on the body element.

The SANACLAP principle ("**s**uppress if **a**nchor **n**ode **a**ncestor
**c**hanged a **l**ayout-**a**ffecting **p**roperty") aims to suppress these
undesired scroll anchoring adjustments.

Under the SANACLAP principle, if the anchor node, or any ancestor of the anchor
node up to and including the scrollable element (or document), has any of a
certain set of CSS properties modified (the "layout-affecting properties"), any
scroll anchoring adjustment that would have resulted from that modification is
suppressed.

The exact set of properties that are considered layout-affecting for the purpose
of scroll anchoring is subject to change, but should at minimum include:

* offset properties (left, top, right, bottom)
* margin properties
* padding properties
* border-width properties
* width, height, min-width, min-height, max-width, max-height
* position, display, float, clear, overflow, transform

## Exclusion / Opt Out API

Scroll anchoring aims to be the default mode of behavior when launched, so that
users benefit from it even on legacy content, but we also want to give
developers control over the feature if and when they need it.

In particular, developers should be able to use CSS

* to disable scroll anchoring in part or all of a webpage (opt out), or
* to exclude portions of the DOM from the anchor node selection algorithm
  (exclusion).

We propose a CSS property `overflow-anchor` whose values have the following
meaning for some element E:

* `overflow-anchor: visible` declares that the DOM subtree rooted at E is
  eligible to participate in the anchor node selection algorithm for any
  scrollable area created by E or an ancestor of E.

* `overflow-anchor: none` declares that the DOM subtree rooted at E should be
  skipped by the anchor node selection algorithm for any scrollable area created
  by E or an ancestor of E.

* `overflow-anchor: auto` invokes the user agent's default behavior.  With the
  launch of scroll anchoring this will be equivalent to `visible`, but is
  subject to future modification.

Note the following implications of the above semantics:

* If `overflow-anchor: none` is applied to a descendant D of a scroller S, then
  S will skip over D when selecting an anchor node, but can still perform scroll
  anchoring adjustments based on anchor nodes outside of D.

* If `overflow-anchor: none` is applied to a scroller S, then S will not perform
  any scroll anchoring adjustments.

* If `overflow-anchor: none` is applied to the root element (`html`), the
  document-level scrollable area will not perform any scroll anchoring
  adjustments.

The `overflow-anchor` property is also proposed (with different values) for
[CSS Sticky Scrollbars](http://tabatkins.github.io/specs/css-sticky-scrollbars/).
This feature is distinct from scroll anchoring, but shares the notion of
adjusting scroll position in response to content changes.

The `overflow-anchor` property is not inherited (but the effect of skipping a
subtree in the anchor node selection algorithm bears a resemblance to property
inheritance).

## Open Issues

#### Definition of Layout-Affecting Property

We need some definition of the set of properties that are considered
layout-affecting for SANACLAP purposes.

The full set of CSS properties that can
affect the position of an element is complex and messy to enumerate.
([Here](https://docs.google.com/spreadsheets/d/1iyg8udD-QNwv9sUja7vDBJI6MdKmFwczfvwwW2BAEzg/edit)
is an attempt to do this based on the Blink layout invalidation code.)

The current implementation treats all of these as layout-affecting for SANACLAP
purposes, but the web compatibility issues that motivated SANACLAP require
suppression for only a small subset of these properties.

In particular, the properties that relate directly to positioning and the CSS
box model are more important than the properties that relate to text rendering
or obscure layout features.

On the other hand, a principled approach of "anything that can affect a
descendant's computed position" has more conceptual simplicity, if this can be
feasibly incorporated into a spec.

#### Separating Opt Out and Exclusion APIs

The `overflow-anchor` property is in a way doing double-duty as an opt out for
scrollable elements and an exclusion mechanism for subtrees within a scrollable
element's content.

Another option would be to use two separate CSS properties for this.

#### Opt In for Simpler Anchoring

SANACLAP adds significant complexity to the behavior of scroll anchoring, and
additional heuristics may still be needed for web compatibility.

There may be value in letting web developers opt in to a simpler and more
aggressive version of scroll anchoring that does not include compatibility
heuristics.

This could be done through another value for the `overflow-anchor` property.

#### Anchoring to Absolute-Positioned Elements

The current anchor node selection algorithm has no special treatment for
absolute-positioned elements unless their containing block is outside the
scroller.  (An absolute-positioned element whose containing block is outside
the scroller is not part of the scrolling content, so it is clearly futile to
anchor to such an element.)

There is an open question of whether to skip absolute-positioned elements
unconditionally.

The argument in favor is that an absolute-positioned element is not affected by
its in-flow siblings.  If static-positioned element A is followed by
absolute-positioned element B, changes in the size of A do not move B, so
anchoring to B isn't useful.

Counter-arguments:

* An absolute-positioned element in a relative-positioned container can be
  affected by in-flow siblings of the container.

* An absolute-positioned element may have in-flow descendants that benefit from
  scroll anchoring.  In particular, many websites have a high-level
  absolute-positioned wrapper around all of the meaningful content.

* There are no known web compatibility issues that are solved by skipping
  absolute-positioned elements.  (Prior to SANACLAP this was not the case, but
  SANACLAP solved those issues in a more generic way and was needed for other
  reasons.)

* Developer predictability and interoperability are best served by minimizing
  "special cases".  Skipping absolute-positioned elements would be a surprising
  quirk that contributes (albeit incrementally) to the complexity of the
  feature.

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

The SANACLAP principle eliminates many, but not all, of the web compatibility
issues that arise from scroll event handler feedback loops.  Solving the
remaining cases is an in-progress discussion.

## Implementation Status

A prototype of scroll anchoring is behind a flag in Google Chrome and can be
enabled from the "about:flags" page.  The implementation is tracked in
[Issue 558575](http://crbug.com/558575) and its dependencies.

Mozilla has discussed a similar feature in
[Bug 43114](https://bugzilla.mozilla.org/show_bug.cgi?id=43114).
