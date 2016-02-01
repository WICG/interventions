# interventions

## Objective

The purpose of this github is to create a place to collaborate between browser vendors on various intervention experiments.

## What are interventions?

Over the course of its existence, the web has accumulated a multitude of standards and common patterns. While most of these are beneficial for both developers and users, if abused (intentionally or otherwise) web APIs can sometimes be a detriment to user experience.

An **intervention** is when a user agent decides to deviate slightly from a standardized behavior in order to provide a greatly enhanced user experience. Because of its nature, it must be done sparingly and with extreme prudence.

An important part of every intervention is closing the feedback loop and educating developers about the new behavior, so that they can respond appropriately.

## Interventions vs. Other Web Platform Changes
An intervention is a specific type of change to the web platform.

An intervention:

1. Breaks long-standing web behavior in as minimal a way as possible.
2. Directly benefits the user in a substantial way.
3. Likely has an opt-out. Will err on the side of giving opt outs, but some rare cases may not need them.
4. Possibly only happens on some pages/content/loads/tbd if it can't be deployed to all web content in a backwards compatible way.

## Proposing an intervention

This github exists to allow collaborative brainstorming, and to share knowledge about what does and does not work. As such, we encourage everyone to propose interventions that could have big UX impact.

To propose a new intervention, just create a new github issue. Be sure to include exactly how and when the intervention will occur, and why it is an obvious user benefit.

## Discussion of interventions

There are currently no plans to have in person meetings about this and discussion should happen via github issues. If you have questions/concerns that can't be addressed via github issues email intervention-dev@chromium.org.
