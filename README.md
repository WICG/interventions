# Interventions

## Objective

The purpose of this github is to create a place to collaborate between browser vendors (and web developers) on various intervention experiments.

## What are interventions?

Over the course of its existence, the web has accumulated a multitude of standards and common patterns. While most of these are beneficial for both developers and users, if abused (intentionally or otherwise) web APIs can sometimes be a detriment to user experience.

An **intervention** is when a user agent decides to deviate slightly from a standardized behavior in order to provide a greatly enhanced user experience. Because of its nature, it must be done sparingly and with extreme prudence.

## Interventions vs. Other Web Platform Changes
An intervention is a specific type of change to the web platform.

An intervention:

1. Breaks long-standing web behavior in as minimal a way as possible.
2. Directly benefits the user in a substantial way.
3. Likely has an opt-out. Will err on the side of giving opt outs, but some rare cases may not need them.

## Proposing an intervention

This github exists to allow collaborative brainstorming, and to share knowledge about what does and does not work. As such, we encourage everyone to propose interventions that could have big UX impact.

To propose a new intervention, just create a new github issue. Be sure to include exactly how and when the intervention will occur, and why it is an obvious user benefit.

## Intervention design guidelines

Interventions that are well-designed to balance the concerns of the user with the concerns of the web developer tend to have the following properties:

1. *Predictable*: A developer can anticipate when their code will be impacted by an intervention, and what the resulting behavior will be.
2. *Avoidable*: Developers following documented best practices shouldn't be impacted by the intervention.
3. *Transparent*: A developer should be able to tell that an intervention has been hit.  For example, a console message should be generated in the developer tools with a link to details about the intervention and how to avoid it, and some sort of detection or reporting should be available in the wild.
4. *Justified by data*: As part of designing and shipping an intervention, the browser should collect statistics from the wild that attempt to quantify both the benefit of the intervention and the potential cost in order to help find a good tradeoff.  Once the intervention has shipped, the browser should continue to collect statistics on how often it is triggering and what benefits it's providing.

In order to design an intervention that meets these properties it's sometimes necessary to first standardize and ship new API surface area.  For example in order to [ignore touch listeners to improve scrolling](https://github.com/WICG/interventions/issues/18) we had to first define an API for [passive event listeners](https://github.com/WICG/EventListenerOptions/blob/gh-pages/explainer.md).

## Discussion of interventions

There are currently no plans to have in-person meetings about this and discussion should happen primarily via github [issues](https://github.com/WICG/interventions/issues).  Also, every intervention which ships in Chrome will have a [discussion thread on blink-dev](https://groups.google.com/a/chromium.org/forum/#!searchin/blink-dev/intervene%7Csort:date) where comments from the public are welcomed.  If you have other questions/concerns email intervention-dev@chromium.org.

For more on the history of the origins of this idea, see http://bit.ly/user-agent-intervention.  See also this recent [presentation at BlinkOn](https://docs.google.com/presentation/d/1yD5nmmzQGAbV6Zn3aiuEOAFccgbWjXomLCDFM4dYMF4/edit) ([video](https://www.youtube.com/watch?v=wQGa_6CRc9I))
