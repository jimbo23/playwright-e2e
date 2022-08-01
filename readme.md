Playwright vs Cypress

**Common**:
1. Both are framework-agnostic. Works well with web built with React/Vue/Angular
2. Easy to use
3. Can be integrated into CI/CD
4. Both are designed to be e2e testing framework
4. Both support headless / non-headless

**Comparison**:
 <table>
  <tr>
    <th></th>
    <th>Playwright</th>
    <th>Cypress</th>
    <th>Who wins</th>
  </tr>

  <tr>
    <td>Language</td>
    <td>js, ts, py, C#, ...</td>
    <td>Limited to js only</td>
    <td>Playwright</td>
  </tr>

  <tr>
    <td>Browser</td>
    <td>Chromium, Firefox, Webkit</td>
    <td>Chrome, Firefox, Edge, i.e. does not support safari</td>
    <td>Playwright</td>
  </tr>
  
  <tr>
    <td>Tooling</td>
    <td>Has a debugging tool</td>
    <td>Has a dedicated GUI for debugging</td>
    <td>Cypress</td>
  </tr>

 <tr>
    <td>Performance</td>
    <td>-</td>
    <td>CI usually takes longer due to the cypress build</td>
    <td>Playwright</td>
  </tr>

  <tr>
    <td>Learning curve</td>
    <td>Easy</td>
    <td>Easier, due to the GUI</td>
    <td>Cypress</td>
  </tr>

  <tr>
    <td>Support Codegen</td>
    <td>Yes, across different languages as well</td>
    <td>Yes, uses a scenario recorder, not as flexible imo</td>
    <td>Playwright</td>
  </tr>

  <tr>
    <td>Community & Popularity</td>
    <td>smaller, rather new</td>
    <td>larger, has been around for a while</td>
    <td>Cypress</td>
  </tr>

  <tr>
    <td>Test parallelisation</td>
    <td>Yes, free</td>
    <td>Yes, paid service</td>
    <td>Playwright</td>
  </tr>

  <tr>
    <td>Debugging</td>
    <td>Step over is possible</td>
    <td>Gotta set breakpoints manually</td>
    <td>Playwright</td>
  </tr>
</table> 

Personal take: **Playwright**


```
This really depends on your use case and your organisation.

First you need to understand their architecture:

Playwright use the Chrome DevTools Protocol (CDP) which allows automation directly in the browser and is supported by all the modern ones. This means, it doesn't run in the execution loop of the browser and as such needs an external process (Node, ...) to drive it. It's a bit like Selenium, which by default uses its own protocol+implementation to drive the browser (the webdrivers). Note that from the version 4,Selenium is now able to use the CDP protocol and its features.

Cypress is an Electron App (Native JS app on desktop) which embeds the browser window in it. The library and your code is directly injected in the execution loop of the browser and needs to modify its default behaviour to achieve this. It uses its own implementation for this purpose although a smoother integration with CDP is in progress, but it's not exposed via its API. Node is also used but the main part of the control is delegated to the custom injected library (in the same process, I guess). PW (as Puppeteer) delegates this control to the CDP implementation of the browser which is standardised and maintained by the browser providers.

What are some consequences coming from these differences ?

    Cypress is limited to Javascript and transpiled languages as it runs in the browser. Playwright is language independant and gets by default JS/TS, Python, C# and Java support. But you can easily use it with other frameworks/languages like RobotFramework for instance.

    Cypress needs some adaptation for each type of browser and currently Safari is not supported. Playwright is supported on all the modern browsers and even gets experimental support for native mobile testing.

    CP doesn't support tabs. PW does.

    Support of Iframes is possible with CP but has sometimes an erratic behaviour and is harder to use than with PW in our experience.

    As it runs directly in the browser, you can setup some shortcuts in your test/fixtures for more efficiency with CP. It means that you can also implement Component/Unit tests with it and it's a promising feature. PW is focused on E2E/System tests and you need another framework to handle the Unit/Component testing part (Jest, Karma/Jasmine ...). You have the choice to use a single framework or 2 different with CP. On the opposite your tests are more close to the real end user's experience with PW as with CP which modifies the browser's behaviour.

    Although the integration with some high level (Acceptance/BDD) frameworks is possible, it's not as easy and more limited than with PW. For instance, when we integrate CP with CucumberJS the test runner is provided by CP and the structure of your scenarios and your implemented steps is more constraining. PW not only offers an integration with CucumberJS, RobotFramework, Gauge, but CodeceptJS provides also an official and excellent support for it and CP is not on their roadmap.

    On the CI aspect. If you don't master the building agents configuration, you probably need a containerisation solution (Docker) to deploy it. We just need npm install for PW.

    Another point is about test parallelisation. PW does provide it out of the box for free. CP doesn't. You'll have to subscribe to their SaaS offer to run your tests in parallel, or find some workarounds which are not perfect.

    Remote browsing concern: Playwright is compliant with an internal/self hosted Selenium grid (https://playwright.dev/docs/selenium-grid). Both are supported on SaaS solutions (BrowserStack, ...)

    On the UI/API aspect, they are relatively close. Both provide a runner, API mocking and fixtures, advanced locators, .... PW uses a classical pattern to manage asynchronicity (async/await) and CP has chosen a kind of promise answer (dot notation, which is in definitive not totally async) to simplify the code with autocompletion. To structure and reuse your code, CP also has a different approach with custom commands to extend the cy keyword which is somewhat similar to the user centric approach of TestingLibrary or Codecept (the I prefix).
```

```

DEBUGGING:

Cypress forces you to use either .pause() or .debug() on a selected element. Using .debug() Cypress will freeze the execution and print on chrome console the results of your selector. Then you can hover over the underlying HTML node and it will be highlighted in chrome, to verify if your selector works properly.

Thing is you can see ONLY that value and you don't have access to any 'state' created by test code. Also adding breakpoints to the test or "debugger" statements, doesn't help much because all code is executed at start and Cypress handles it asynchronously later. You'd have to put the debugger statement inside a "resolve" function to make sure async code has finished when you stop, but STILL you can see the value only for this line of code, and not inspect any other state.

You could also try using something like Promisify which turns Cypress own async functionality into Promises. Then you enter debugger statements after an await and can inspect variables that have been awaited. Still has problems though because it's a third party library, and Cypress wasn't originally designed to work with Promises and async/await. I tried it and faced a lot of problems while attempting to debug, so I switched to just using Cypress's .debug() function.

Playwright on the other hand, has perfect integration with Visual Studio Code and works with promises and async/await. You can debug it through VSCode, stop anywhere you want, and evaluate any test state you want. It even highlights the DOM element on chromium when you put VSCode's cursor on top of a selector expression (works with multi-dot statements too!!)

Playwright also features it's own specific inspector, where u can run a test step by step while watching on chromium and evaluate selectors live while ur frozen on a statement.

UI TEST CREATION:

With this inspector you can also "write" a test by recording your actions on UI, without even having to write code, maybe only correct some selectors for when using frameworks that create dynamic element ids (so on your second run the id might have changed and the selector doesn't work anymore).

I didn't find this feature on Cypress while I was checking it out and I found it pretty nice on Playwright.

So debugging and UI test creation are a big plus for Playwright.

All the devs voted for Playwright against Cypress in the end, so that says something ;)```
