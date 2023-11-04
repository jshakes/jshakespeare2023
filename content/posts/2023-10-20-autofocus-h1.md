---
date: 2023-11-04 12:20:52+00:00
title: Making route changes accessible in React with an autofocusing h1
description:
featured_image:
featured_image_caption:
featured_image_alt:
slug: accessible-route-change-react-router-autofocus-heading
---

Many routing packages for React don’t include any sort of focus management out of the box (including the most popular, React Router). This means that after a route change the browser's focus will linger on the most recently focused element, even if that element is no longer in the DOM. This can be confusing for users who are navigating using a screen reader and/or their keyboard.

If you’re starting a new React project that requires routing, you may want to consider using [Reach Router](https://reach.tech/router/), which gives you focus management for free. From their [docs](https://reach.tech/router/accessibility):

> Reach Router provides out-of-the-box focus management so your apps are significantly more accessible without you breaking a sweat.
> 
> When the location changes, the top-most part of your application that changed is identified and focus is moved to it.

However, if you’re stuck with some other routing package then you will have to roll your own focus management. There are many ways to do this, but there is some [consensus](https://www.gatsbyjs.com/blog/2019-07-11-user-testing-accessible-client-routing/) that the best way to handle SPA route changes is to move focus to the new page’s heading, in other words the `h1` tag.

When we move focus to the h1 we do two things to help users:

1. We move focus to the top of the new page's content, ideally after any navigation or page chrome that may be redundant. We ensure that focus is not left lingering on some possibly irrelevant or likely non-existent DOM element from the previous page.
2. We prompt screen readers to announce the new page's title

By default, `h1` is not a focusable element, but we can make any HTML element focusable using the `tabindex` attribute (in React, we must write this as `tabIndex`). 

Then, to focus the element, we just need to add a one-line `useEffect` hook that focuses the `h1` on mount using a ref. 

Altogether, the component looks like this in TypeScript:

```tsx
import React, { useEffect, useRef } from 'react';

/**
 * A wrapper for h1 that focuses on mount. Useful as a page heading to
 * reset focus after a route change. See
 * https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Client-side_JavaScript_frameworks/React_accessibility
 */
export default function FocusablePageHeading(props: {
  children: React.ReactNode;
}) {
  const el = useRef<HTMLHeadingElement>(null);
  useEffect(() => el.current?.focus(), []);

  return (
    <h1 tabIndex={-1} ref={el}>
      {props.children}
    </h1>
  );
}
```

And that’s it! Note that we don’t need to do anything fancy to make the h1 unfocusable after it is blurred. That’s because although we are give the element a `tabindex` attribute to make it focusable programmatically, because its value is `-1`  the user will be unable to re-focus it with the Tab key once it is blurred.

## Further reading

- [What we learned from user testing of accessible client-side routing techniques with Fable Tech Labs](https://www.gatsbyjs.com/blog/2019-07-11-user-testing-accessible-client-routing/) — gatsbyjs.org
- [Accessibility in React](https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Client-side_JavaScript_frameworks/React_accessibility) — developer.mozilla.org