---
title: JavaScript Refresh Page – How to Reload a Page in JS
date: 2024-08-07T14:38:23.052Z
author: Joel Olawanle
authorURL: https://www.freecodecamp.org/news/author/joel-olawanle/
originalURL: https://www.freecodecamp.org/news/javascript-refresh-page-how-to-reload-a-page-in-js/
translator: ""
reviewer: ""
---

JavaScript is a versatile programming language that allows developers to create dynamic and interactive web applications. One common task in web development is to refresh or reload a web page, either to update its content or to trigger certain actions.

<!-- more -->

In this article, we will explore different ways to refresh a page in JavaScript and understand the pros and cons of each approach.

## Why Refresh a Page in JavaScript?

Refreshing a web page can be useful in various scenarios. For example:

1.  **Content Update:** If the content on a web page is dynamic and changes frequently, you may need to refresh the page to display the latest data or information. This is commonly seen in news websites, stock market trackers, weather apps, and so on.
2.  **Form Submission:** After submitting a form on a web page, you may want to refresh the page to show a success message or reset the form for a new submission.
3.  **State Reset:** In some cases, you may want to reset the state of a web page or clear certain data to start fresh. Refreshing the page can help achieve this.

Now, let's explore different ways to refresh a page in JavaScript.

## Method 1: How to Refresh the Page Using `location.reload()`

The simplest way to refresh a page in JavaScript is to use the `location.reload()` method. This method reloads the current web page from the server, discarding the current content and loading the latest content.

```js
// Refresh the page
location.reload();
```

### Pros of Using location.reload()

-   It's straightforward and easy to use.
-   It reloads the entire page from the server, ensuring that you get the latest content.

### Cons of Using location.reload()

-   It discards the current content of the page, which can result in loss of user input or data.
-   It may cause a flickering effect as the page reloads, which can impact user experience.

## Method 2: How to Refresh the Page Using `location.replace()`

Another way to refresh a page in JavaScript is to use the `location.replace()` method. This method replaces the current URL of the web page with a new URL, effectively reloading the page with the new content.

When you try this in your console, you will notice it displays your current URL:

```js
console.log(location.href)
```

This means, when you use the `location.replace()` method to replace the current URL of the web page with a new URL (same URL), your page will refresh.

```js
// Refresh the page by replacing the URL with itself
location.replace(location.href);
```

### Pros of Using location.replace()

-   It's a quick way to reload the page with new content.
-   It preserves the current content of the page and replaces only the URL, avoiding loss of user input or data.

### Cons of Using location.replace()

-   It replaces the entire URL of the page, which may result in the loss of the current browsing history.
-   It may not work in some scenarios, such as when the web page was opened in a new window or tab.

## Method 3: How to Refresh the Page Using `location.reload(true)`

The `location.reload()` method also accepts a boolean parameter `forceGet` which, when set to `true`, forces the web page to reload from the server, bypassing the cache.

This can be useful when you want to ensure that you get the latest content from the server, even if the page is cached.

```js
// Refresh the page and bypass the cache
location.reload(true);
```

### Pros of Using location.reload(true)

-   It ensures that you get the latest content from the server, even if the page is cached.

### Cons of Using location.reload(true)

-   It discards the current content of the page, which can result in loss of user input or data.
-   It may cause a flickering effect as the page reloads, which can impact user experience.

## Method 4: How to Refresh the Page Using `location.href`

Another way to refresh a page in JavaScript is to use the `location.href` property to set the URL of the web page to itself. This effectively reloads the page with the new URL, triggering a page refresh.

```js
// Refresh the page by setting the URL to itself
location.href = location.href;
```

### Pros of Using location.href

-   It's a simple and effective way to refresh the page.
-   It preserves the current content of the page and only updates the URL, avoiding loss of user input or data.

### Cons of Using location.href

-   It replaces the entire URL of the page, which may result in the loss of the current browsing history.
-   It may not work in some scenarios, such as when the web page was opened in a new window or tab.

## Method 5: How to Refresh the Page Using `location.reload()` with a Delay

If you want to add a delay before refreshing the page, you can use the `setTimeout()` function in combination with the `location.reload()` method.

This allows you to specify a time interval in milliseconds before the page is reloaded, giving you control over the timing of the refresh.

```js
// Refresh the page after a delay of 3 seconds
setTimeout(function(){
    location.reload();
}, 3000); // 3000 milliseconds = 3 seconds
```

### Pros of Using location.reload() with a Delay

-   It allows you to control the timing of the page refresh by adding a delay.
-   It provides flexibility in scenarios where you want to refresh the page after a certain event or action.

### Cons of Using location.reload() with a Delay

-   It may cause a delay in the page refresh, which can impact user experience.
-   It may not work as expected in scenarios with unstable or slow network connections.

## Wrapping Up

In this article, you have learned the different ways to refresh a page in JavaScript. Each method has its pros and cons, which should make it easier for you to choose the best method for your web development project.

When using any of these methods to refresh a page, it's important to consider the impact on user experience, data loss, and browsing history.

I hope this article helps you understand how to reload a web page in JavaScript and choose the appropriate method for your specific use case.

Happy coding!

Embark on a journey of learning! [Browse 200+ expert articles on web development][1]. Check out [my blog][2] for more captivating content from me!

---

![Joel Olawanle](https://www.freecodecamp.org/news/content/images/size/w60/2022/06/1654890413623.jpg)

Frontend Developer & Technical Writer

---

If you read this far, thank the author to show them you care. Say Thanks

Learn to code for free. freeCodeCamp's open source curriculum has helped more than 40,000 people get jobs as developers. [Get started][3]

[1]: https://joelolawanle.com/contents
[2]: https://joelolawanle.com/posts
[3]: https://www.freecodecamp.org/learn/