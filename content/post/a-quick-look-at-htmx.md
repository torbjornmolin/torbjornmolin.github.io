+++
title = "A quick look at HTMX"
description = "SPA's are going out of style"
date = 2022-10-16T09:13:50Z
author = "Torbjörn Molin"
+++

## The times they are a-changing

There are trends in everything and tech stacks, programming languages and frameworks are certainly no exception, quite the opposite. Recently one of the technologies that has fallen out of favor the people that like to tell the internet what's hot is SPA applications and client side rendered pages in general.

Thoughtworks added ["SPA by default"](https://www.thoughtworks.com/radar/techniques/spa-by-default) to their hold list in march of 2022 on the grounds that teams have stoped contemplating that going with a SPA is even a choice in the first place. They also list many of the complexities that come with SPA applications, again noting that the complexity might be warranted but needs to be taken on as an active decision.

One alternative that seems to come up a lot lately is [htmx](https://htmx.org), a small Javascript library that enables HTML elements to dynamically update their content from a server resource in response to events. There is one [case study](https://htmx.org/essays/a-real-world-react-to-htmx-port/) available on their site that shows how replacing React with HTMX reduced total code base size by 67% and page load times by 50-60%.

Being mainly a backend developer who also often has to do frontend (I guess that makes me full stack?), writing less Javascript sounds like an appealing proposal so I thought I'd do a quick small project just to see what HTMX is like.

## What to build

I'd like to get this done in an afternoon, so it will have to be something small. I'll build a calculator that helps the user figure out how to distribute income in a typical Swedish freelancer one person limited company. The input will be hourly rate and perhaps expected number of days billed per year. Just so that the demo keeps some server side state, the user is able to save the result of the calculation and the ten last saved calculations are displayed on the page.

I'll build this as a ASP.NET Core web api that also servers a static page that is the application.

## Let's get started

To get started, we create a new web api project and add a folder for static pages to it.

```bash
mkdir htmx_test
cd htmx_test
dotnet new webapi
mkdir wwwroot
```

The wwwroot folder is the default content folder for an ASP.NET project, but no static content is served from it by default in a web api project, so we need to enable it. To do so, we simply add a line to the default `Program.cs`

```csharp
app.UseStaticFiles();
```

We'll also create a basic index.html and add it to `wwwroot`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />

    <title>Freelance calculator</title>
    <meta name="description" content="Freelance calculator." />
    <meta name="author" content="Torbjörn Molin" />
  </head>
  <body>
    <header>Freelance calculator</header>
    <div>
      <h1>Calculator</h1>
      <p>This is the calculator</p>
    </div>
    <div hx-get="/calculator/history" hx-trigger="every 2s">Initial</div>
  </body>
</html>
```

Now when we run the project with `dotnet run` and open `http://localhost:<port>/index.html` we should see our page.
![static site](/htmx-start-webpage.png)
Now that we've set up the project we can start having a go at creating our small app with htmx. We start by adding htmx to our site

```html
<script src="https://unpkg.com/htmx.org@1.8.2"></script>
```

You can also download `htmx.min.js` and keep it in `wwwroot` and reference that, if you like.

Now that we've added htmx, let's create some dynamic content. We start by renaming the default `WeatherController` to `CalculatorController`. We also delete most of the boilerplate code and add a method that will respond to the route `<baseurl>/calculator/history`:

```csharp
using Microsoft.AspNetCore.Mvc;

namespace htmx_test.Controllers;

[ApiController]
[Route("[controller]")]
public class CalculatorController : ControllerBase
{
    private readonly ILogger<CalculatorController> _logger;

    public CalculatorController(ILogger<CalculatorController> logger)
    {
        _logger = logger;
    }

    [HttpGet("history")]
    public string GetHistory()
    {
        return $"history, {DateTime.Now.Second}";
    }
}
```

We then change our history div in `index.html` to

```html
<div hx-get="/calculator/history" hx-trigger="every 2s"></div>
```

The page now polls our endpoint every two seconds and replaces the content of our div with whatever html is returned.
![dynamic history](/htmx-dynamic-history.png)
Pretty cool! But not very useful yet. Let's add a get method to `CalculatorController` that for an input of hourly rate and number of vacation days returns how much is left after expenses are paid.

```csharp
    [HttpGet(Name = "Calculator")]

    public string GetCalculator(int rate, int vacationDays)
    {
        var calculatorResult = new FreelanceCalculator().Calculate(rate, vacationDays);
        var sb = new StringBuilder();
        sb.Append($"<p>Total salary cost: {calculatorResult.SalaryExpense}</p>");
        sb.Append($"<p>Pension cost {calculatorResult.Pension}</p>");
        sb.Append($"<p>Remaining: {calculatorResult.Remaining}</p>");

        return sb.ToString();
    }
```

The `Calculate` method of `FreelanceCalculator` is not that important here, but it returns a `FreelanceCalculatorResult` with three integers that are used by `GetCalculator`.

We also make some updates to `index.html` to use the new endpoint:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />

    <title>Freelance calculator</title>
    <meta name="description" content="Freelance calculator." />
    <meta name="author" content="Torbjörn Molin" />
    <script src="htmx.min.js"></script>
  </head>
  <body>
    <header>Freelance calculator</header>
    <div>
      <form hx-get="/Calculator" hx-target="#result">
        <input type="number" name="rate" value="850" />
        <input type="number" name="vacationDays" value="30" />
        <button type="submit">Calculate</button>
      </form>
      <div id="result"></div>
    </div>
    <div hx-get="/calculator/history" hx-trigger="every 2s">Initial</div>
  </body>
</html>
```

The top div gets a form with two number inputs. `hx-get` tells htmx which endpoint to use and `hx-target` specifies that the result should be loaded in the div with id `result`.

We now have an actual interactive page:

![a working calculator](/htmx-first-calculator.png)

Let's make the history section display something useful as well. We'll do it very quick and dirty and non-thread safe and just keep a static `Queue<FreelanceCalculatorResult>` in the controller, add the result of the calculations to that every time and remove entries if we have more than ten.

In `CalculatorController` we add

```csharp
private static readonly Queue<FreelanceCalculatorResult> freelanceCalculatorResults = new Queue<FreelanceCalculatorResult>();
```

`GetHistory()` gets changed to

```csharp
    [HttpGet("history")]
    public string GetHistory()
    {
        if (freelanceCalculatorResults.Count == 0)
            return "No history yet";

        var sb = new StringBuilder();
        sb.Append("<ul>");
        foreach (var result in freelanceCalculatorResults.AsEnumerable().Reverse())
        {
            sb.Append($"<li>Rate: {result.Rate} Vacation days: {result.VacationDays} Amount left: {result.Remaining}</li>");
        }
        sb.Append("</ul>");
        return sb.ToString();
    }

```

and in `GetCalculator` we add

```csharp
        freelanceCalculatorResults.Enqueue(calculatorResult);
        if (freelanceCalculatorResults.Count > 10)
            freelanceCalculatorResults.Dequeue();

```

![calculator with history](/htmx-calculator-with-history.png)

Finally, we probably should have the history div load when the page loads and whenever we submit a new calculation instead of polling every two seconds. To accomplish that, assign an id to the button. Like `id="submit-button"` for example, and change the `hx-trigger` of the history div to `hx-trigger="load, click from:#submit-button"`. This tells htmx to listen to the `load` event and also to the `click` event from the html-element with id `submit-button` and voilà, we have the behavior that we're after.

The full html that we end up with looks something like this:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />

    <title>Freelance calculator</title>
    <meta name="description" content="Freelance calculator." />
    <meta name="author" content="Torbjörn Molin" />
    <script src="htmx.min.js"></script>
  </head>
  <body>
    <header>Freelance calculator</header>
    <div>
      <form hx-get="/Calculator" hx-target="#result">
        <input type="number" name="rate" value="850" />
        <input type="number" name="vacationDays" value="30" />
        <button id="submit-button" type="submit">Calculate</button>
      </form>
      <div id="result"></div>
    </div>
    History
    <div
      hx-get="/calculator/history"
      hx-trigger="load, click from:#submit-button"
    ></div>
  </body>
</html>
```

## Final thoughts

Even though this was a simple hello world type example, I'm kind of impressed with how easy it was. The features that htmx provide really do go a long way with very little effort and since the concepts introduced are so few and simple I didn't have to read a lot of documentation to figure out what to do. Around ten minutes of reading was probably all I had to do.

I guess that some sort of template engine to handle the HTML snippets in the backend would be nice to have for a real world project, but if you have that htmx will probably give many projects most if not all of what is needed for very little effort compared to the big Javascript frameworks.

If you want to look at the very hacky code, it's available on [Github](https://github.com/torbjornmolin/htmx_test)
