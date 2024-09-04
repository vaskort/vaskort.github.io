---
layout: post
title: A trap focus function you need for your modals
---

As Accessibility gets more and more important there are some functions that you would find useful in every front end project you work on. One of them is the trap focus function. The function which allows the focus to happen only to the children of a container, in our example to our modal.

## TL;DR

- [Full example on CodeSandbox (React)](https://codesandbox.io/s/github/vaskort/trap-focus)
- [Full example on GitHub (React)](https://github.com/vaskort/trap-focus)
- [Trap focus function on GitHub](https://github.com/vaskort/trap-focus/blob/master/src/Modal/Modal.js#L7-L56)
- [Plain JS/HTML example on Codepen](https://codepen.io/vaskort/pen/LYpwjoj)

## Intro

This post got inspired from W3's example where they are [demoing the trap focus functionality](https://www.w3.org/TR/wai-aria-practices/examples/dialog-modal/dialog.html), it's pretty much their code just a bit simplified and explained.

What this function does is that it just checks if the focused item is a child of the container you have defined initially, and if not, it forces the focus back to the first or the last child of the container.

From my experience this logic works better in cases where our user uses a Voice Over hotkey to jump to a different area of the page. For instance if he clicks the "Jump to the next heading" hotkey with this logic you allow him to go but then on the next tab you force the focus back to the modal.

## How the function works

At first we define our function that takes two params, the first one is the element that would be the container element, our modal. The second one is the element which had focus previously and is the one that we will focus back when the modal is closed.

Then we create an array with all the focusable elements of our modal.

```javascript
const trapFocus = ((element, prevFocusableElement = document.activeElement) => {
    const focusableEls = Array.from(
      element.current.querySelectorAll(
        'a[href]:not([disabled]), button:not([disabled]), textarea:not([disabled]), input[type="text"]:not([disabled]), input[type="radio"]:not([disabled]), input[type="checkbox"]:not([disabled]), select:not([disabled])'
      )
    );
```

Next, we "grab" the first and the last item from the array of the focusable elements as we will need them later and we declare a `currentFocus` variable to store the item that has focus. And after the variable declaration we focus on the first element of the modal and update the `currentFocus` variable.

```javascript
const firstFocusableEl = focusableEls[0];
const lastFocusableEl = focusableEls[focusableEls.length - 1];
let currentFocus = null;

firstFocusableEl.focus();
currentFocus = firstFocusableEl;
```

Inside this function we declare the handleFocus function. It includes the main functionality of our trap focus function, follow the comments as it's pretty self-explanatory.

```javascript
const handleFocus = e => {
  e.preventDefault();
  /* if the focused element "lives" in your modal container
   then do nothing and update the currentFocus var */
  if (focusableEls.includes(e.target)) {
    currentFocus = e.target;
    /* else you're out of the modal container. */
  } else {
    /* If previously the focused element was
     the first element then focus the last element */
    if (currentFocus === firstFocusableEl) {
      lastFocusableEl.focus();
    } else {
      /* Else the previously focused element was the last element
       so just focus the first one */
      firstFocusableEl.focus();
    }
    // update the current focus var
    currentFocus = document.activeElement;
  }
};
```

We attach the `handleFocus` function to the document whenever a focus event is fired. Notice that you will not be able to listen to this event on `document` if you don't set the `useCapture` parameter to `true`.

```javascript
document.addEventListener("focus", handleFocus, true);
```

Finally we return a function the will help us clear the `focus` event from the `document` but also focus to the element that had the focus before the modal was open. Usually a button that opens the modal.

```javascript
return {
  onClose: () => {
    document.removeEventListener("focus", handleFocus, true);
    prevFocusableElement.focus();
  }
};
```

## A small caveat with this approach

There is one scenario where this function will not work as expected. But don't worry there's a small trick to fix it.

If your modal is the last html element in your DOM and you keep pressing the tab key, eventually you will manage to focus the address bar of your browser and get out of your modal. That is because we can only get events on elements that exist in the `document` and not outside of it. Hence the `handleFocus` function will not run. And no, as of May 2020, there's no way to find out if a user has focus on the address bar (thank god because imagine all the "Please don't leave us" messages that you would get from websites if it was possible).

### The solution

In a real-world project if you have your Modal html between your header menu and footer you will most likely not have this problem, assuming your header and footer have some focusable elements.
If that's not the case and/or you want to be 100% sure that it should work you can add two empty divs with `tabindex=0` before and after your Modal markup like so:

```html
<div tabindex="0"><div>
<div id="modal">...</div>
<div tabindex="0"><div>
```

## Conclusion

I hope this post would help you towards improving the accessibility of your page. Accessibility can be hard to nail down correctly especially if you start testing in Voice Over tools as it can be daunting at first but you quickly get the grasp of them. Good luck!

## Useful links

- [Full example on CodeSandbox (React)](https://codesandbox.io/s/github/vaskort/trap-focus)
- [Full example on GitHub (React)](https://github.com/vaskort/trap-focus)
- [Trap focus function on GitHub](https://github.com/vaskort/trap-focus/blob/master/src/Modal/Modal.js#L7-L56)
- [Plain JS/HTML example on Codepen](https://codepen.io/vaskort/pen/LYpwjoj)
- [W3.org - Modal Example](https://www.w3.org/TR/wai-aria-practices/examples/dialog-modal/dialog.html)
- [Giving focus to buttons diagram on MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button#Clicking_and_focus)