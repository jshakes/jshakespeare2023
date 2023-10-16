---
date: 2023-10-16 12:20:52+00:00
title: How to test an error boundary component with React Testing Library
description:
featured_image:
featured_image_caption:
featured_image_alt:
slug: react-error-boundary-testing-rtl
---

[[Skip straight to the code](#full-working-example)]

In a React application, an error boundary catches JavaScript errors thrown during the rendering of other components and optionally displays some fallback UI, typically an apologetic message and next steps to try. It may have other functional responsibilities like passing details of the error along to a monitoring service such as Sentry or New Relic. Without an error boundary, uncaught errors in a React application result in the ‘white screen of death’ whereby the application removes itself from the DOM entirely and the user is left in an empty void of nothingness.

If your application already includes an error boundary component: congratulations on accepting your innate human fallibility! Now let’s finish the job and make sure it actually works by writing a unit test.

In this post I won’t include the code for the error boundary component itself. The [React docs include a decent working example](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary) and there are plenty of third party packages like [react-error-boundary](https://github.com/bvaughn/react-error-boundary) or [@sentry/react](https://www.npmjs.com/package/@sentry/react)’s `ErrorBoundary` component. Good unit tests are agnostic to implementation details. All I’m assuming below is that your error boundary component can be passed children via `props.children` and that when an error is caught, it will display some fallback copy instead of those children. I’m British so have chosen the word ‘sorry’. (Other apologies are available but please don’t use [annoying cutesy language](https://alexwlchan.net/2022/no-cute/).)

## Setup

We need to do a couple of things before we can test the error boundary itself:

* Temporarily suppress `console.log` errors so that our test runner process doesn’t get its output spammed with error messages or worse, aborted
* Define a React component that will throw an error when rendered

```jsx
import React from 'react';
import MyErrorBoundary from './MyErrorBoundary';
import { render, screen } from '@testing-library/react';

describe('MyErrorBoundary', () => {
  // A very buggy component
  const ThrowError = () => {
    throw new Error('Test');
  };

  // Temporarily suppress console errors so we don't clog the logs
  const realError = console.error;
  beforeEach(() => {
    console.error = jest.fn();
  });
  afterEach(() => {
    console.error = realError;
  });
});
```

(Yep, that really is a valid React component!)

## Assertions

At its core an error boundary component only has two jobs: display its children when everything is fine, and display a fallback when those children throw an error.

In RTL we can test this behavior with the following assertions:

```jsx
it('renders children when everything is fine', async () => {
  render(
    <MyErrorBoundary>
      <p>Everything is fine</p>
    </MyErrorBoundary>
  );
  expect(screen.getByText(/Everything is fine/i)).toBeInTheDocument();
});

it('shows an apologetic error message when an unhandled exception is thrown', () => {
  render(
    <MyErrorBoundary>
      <ThrowError />
      <p>Everything is fine</p>
    </MyErrorBoundary>
  );

  expect(screen.queryByText(/Everything is fine/i)).not.toBeInTheDocument();
  expect(screen.getByText(/sorry/i)).toBeInTheDocument();
});
```

If your error boundary has other functionality you want to test, for example displaying a contact form, you could add assertions for that too.

## Full working example

```jsx
import React from 'react';
import MyErrorBoundary from './MyErrorBoundary';
import { render, screen } from '@testing-library/react';

describe('MyErrorBoundary', () => {
  // A very buggy component
  const ThrowError = () => {
    throw new Error('Test');
  };

  // Temporarily suppress console errors so we don't clog the logs
  const realError = console.error;
  beforeEach(() => {
    console.error = jest.fn();
  });
  afterEach(() => {
    console.error = realError;
  });

  it('renders children when everything is fine', async () => {
    render(
      <MyErrorBoundary>
        <p>Everything is fine</p>
      </MyErrorBoundary>
    );
    expect(screen.getByText(/Everything is fine/i)).toBeInTheDocument();
  });

  it('shows an apologetic error message when an unhandled exception is thrown', () => {
    render(
      <MyErrorBoundary>
        <ThrowError />
        <p>Everything is fine</p>
      </MyErrorBoundary>
    );

    expect(screen.queryByText(/Everything is fine/i)).not.toBeInTheDocument();
    expect(screen.getByText(/sorry/i)).toBeInTheDocument();
  });
});
```
