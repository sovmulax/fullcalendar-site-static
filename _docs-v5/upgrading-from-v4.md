---
title: V5-beta Release Notes and Upgrade Guide
layout: text
---

<style>

  #toc { flex-basis: 30% }
  #toc ul { margin-left: 0; list-style-position: inside  }

  .text-content table {
    table-layout: fixed;
    line-height: 1.5em; /* TODO: put outside? */
  }

  .text-content th,
  .text-content td {
    padding: 0 12px;
  }

  .text-content td:first-child {
    width: 30%; /* first column */
  }

  .text-content table p {
    margin: 1em 0 !important; /* previous rule killed top margin for first child */
  }

  .text-content p code,
  .text-content li code {
    white-space: nowrap;
  }

  .text-content .diff-list {
    margin: 0 0 0 4px;
    padding: 0;
    list-style: none;
  }

  .text-content .diff-list li {
    text-indent: -1em;
    margin-left: 1em;
  }

  .diff-removed,
  .diff-added,
  .diff-unchanged {
    font-family: Monaco, Consolas, "Lucida Console", monospace;
    font-size: 14px;
  }

  .diff-removed:before,
  .diff-added:before {
    display: none; /* should be activated with inline */
    margin-right: 6px;
    font-size: 12px;
    opacity: 0.5;
    position: relative;
    top: -1px;
    /* text-decoration: none !important; want to override hover underline. not working */
  }

  .diff-list li.diff-added:before,
  .diff-list li.diff-removed:before,
  .diff-list li > .diff-added:first-child:before,
  .diff-list li > .diff-removed:first-child:before,
  .diff-with-icon:before {
    display: inline;
  }

  .diff-removed {
    color: #ad0000 !important; /* overcome link color */
  }

  .diff-added {
    color: #006b00 !important; /* overcome link color */
  }

  .diff-removed:before {
    content: 'x';
  }

  .diff-added:before {
    content: '+';
  }

</style>

<div class='sidebar-layout'>
<div class='sidebar-layout__main' id='summary' markdown='1'>

## Summary of Changes

This guide outlines the changes between v4 and v5-beta.

**Major breaking changes:**

- the CSS and DOM structure has been refactored. any custom CSS you've written will break ([see below](#css-and-dom-structure))
- no more manually importing fullcalendar stylesheets. however, your build system will need adjustment ([see below](#css-importing))
- many options have been replaced, especially ones related to custom content injection. If an option name has the word `text`, `html`, or `render` in it, it has probably been replaced by something else.
- the Vue and Angular connectors have different APIs for accepting options

**Major new features:**

- rendering optimizations. we now internally use a virtual DOM ([see below](#virtual-dom))
- ability to inject custom content almost anywhere in the calendar
- horizontal scrolling for vertical resource view ([#3022](https://github.com/fullcalendar/fullcalendar/issues/3022)). also applies to all daygrid/timegrid views. accomplished by setting [dayMinWidth](dayMinWidth). Requires a premium plugin.
- expanding the height of timegrid slots ([#265](https://github.com/fullcalendar/fullcalendar/issues/265))
- expanding the height of resource rows in timeline view ([#4897](https://github.com/fullcalendar/fullcalendar/issues/4897))
- pre-built bundles that require minimal configuration and no build system ([see below](#pre-built-bundles))
- the React and Vue connectors accept custom rendering functions/templates

**Things that are NOT YET IMPLEMENTED but will be soon**:

- sticky headers ([#3473](https://github.com/fullcalendar/fullcalendar/issues/3473))
- printer-friendly timeline view ([#4813](https://github.com/fullcalendar/fullcalendar/issues/4813))
- TypeScript definitions for new options are in a state of disarray. Until they are fixed, the options definitions have been relaxed to accept almost any input.
- a pure-React distro without leveraging Preact (potentially)

<br />

**Want the full docs** in a non-changelog format? [View the docs]({{ site.baseurl }}/docs/v5)

**Found a bug?** [Report it on the issue tracker]({{ site.baseurl }}/reporting-bugs)

<!-- **Have a comment?** [Comment on the blog post](#) -->

</div>
<div class='sidebar-layout__sidebar' id='toc' markdown='1'>

## Table of Contents

- [Getting the Beta](#getting-the-beta)
- [Virtual DOM](#virtual-dom)
- [CSS and DOM Structure](#css-and-dom-structure)
- [CSS Importing](#css-importing)
- [Pre-built Bundles](#pre-built-bundles)
- [View Rendering](#view-rendering)
- [Date Rendering](#date-rendering)
- [Event Rendering](#event-rendering)
- [Resource Rendering](#resource-rendering)
- [List View Rendering](#list-view-rendering)
- [Now Indicator Rendering](#now-indicator-rendering)
- [Calendar Sizing](#calendar-sizing)
- [Custom JS Views](#custom-js-views)
- [Other Misc Changes](#other-misc-changes)
- [Upgrading from v3](#upgrading-from-v3)
- [React Connector](#react-connector)
- [Vue Connector](#vue-connector)
- [Angular Connector](#angular-connector)

</div>
</div>


## Getting the Beta

You can install the packages via npm using the `@5.0.0-beta` postfix:

```sh
npm install --save @fullcalendar/core@5.0.0-beta @fullcalendar/daygrid@5.0.0-beta
```

Or, you can use one of the new pre-built bundles ([see below](#pre-built-bundles))


## Virtual DOM

FullCalendar now internally uses a miniature virtual-DOM library called [Preact](https://preactjs.com/). Aside from making the codebase more maintainable, it makes FullCalendar more performant to end-users. DOM manipulations and page reflows are kept to a minimum. This is especially true for rerendering events. FullCalendar no longer needs to rerender ALL events when just one event changes. It rerenders only what it needs to ([#3003](https://github.com/fullcalendar/fullcalendar/issues/3003)).

Just because we use a virtual DOM doesn't mean we no longer think about performance. We still care about limiting the amount of rerender execution, even though it performs fewer real DOM operations. This will continue to be a priority as we further develop the beta.

How does this affect FullCalendar's API? It doesn't really. From any of the content injection options like `eventContent` you are able to construct and return a virtual DOM node. [Learn more in this article](content-injection#vdom). Aside from that, you won't need to think about the virtual DOM.


## CSS and DOM Structure

Firstly, the DOM structure of the calendar has been simplified quite a bit. There is less nesting. Month view and daygrid view have particularly benefitted from this. Each row is represented by a single `<tr>` and events are rooted in individual `<td>` cells, whereas in v4 we had many tables-within-tables (addresses [#2853](https://github.com/fullcalendar/fullcalendar/issues/2853))

The CSS has been completely rewritten. Most importantly, the selectors are flatter. Instead of a selector like `.fc-event .fc-title` we now use something like `.fc-event-title`. This results in fewer selectors battling for precedence and makes styling easier to override.

As a result, if you've written custom styling, <strong>it will most likely need to be rewritten for v5</strong>, or at the very least you will need to swap out your classNames. To help you do this, we will likely release a className-upgrade document prior to the official release.

<strong>CSS variables</strong> are a new feature. They allow you to manipulate fullcalendar's CSS in a more surgical way than simply overriding existing rules. Some build-system magic is required for this. Read more on <a href='css-variables'>how to use CSS variables &raquo;</a>


## CSS Importing

In v4, it was your responsibility to import all of fullcalendar's stylesheets. You may have done this in one your project's SASS files. Or, if you had a build system that handled CSS, you may have done this from your JavaScript.

In v5, you no longer need to do this! The plugins will import their own stylesheets. So, you'll be able remove lines like these:

```js
import { Calendar } from '@fullcalendar/core'
import timeGridPlugin from '@fulllcalendar/timegrid'

// DON'T DO THIS ANYMORE! it will happen automatically
// import '@fullcalendar/core/main.css';
// import '@fullcalendar/daygrid/main.css'; // a dependency of timegrid
// import '@fullcalendar/timegrid/main.css';

let calendar = new Calendar(calendarEl, {
  plugins: [ timeGridPlugin ]
  // other options...
})
```

<strong>HOWEVER</strong>, you'll need a build system that is able to handle CSS. Some popular ones:

- for [Webpack](https://webpack.js.org/) you'll need [css-loader](https://webpack.js.org/loaders/css-loader/)
- for [Rollup](https://rollupjs.org/) you can use [rollup-plugin-postcss](https://github.com/egoist/rollup-plugin-postcss)
- for [Parcel](https://parceljs.org/) you won't need to do anything, it's supported by default

Configuring your build system to handle CSS is beyond the scope of this document. There are many examples out there.

<strong>If you don't use a build system</strong>, meaning you use manual <code>&lt;script&gt;</code> tags and browser globals, then please read the next section...


## Pre-built Bundles

What if you want to avoid using a build system? What if you prefer manual `<script>` tags and browser globals? This is why we are beginning to offer pre-built bundles of plugins ([#4566](https://github.com/fullcalendar/fullcalendar/issues/4566)). In fact, using the pre-built bundles will be the <strong>ONLY</strong> way to use manual `<script>` tags going forward. The individual plugins will no longer provide browser-runnable UMD files.

First, [get the bundle distro files on the Getting Started page &raquo;](getting-started#pre-built-bundles)

To use a bundle, do something like this:

```html
<link ref='fullcalendar/main.css' rel='stylesheet' />
<script src='fullcalendar/main.js'></script>
<script>
  document.addEventListener('DOMContentLoaded', function() {
    var calendarEl = document.getElementById('calendar')
    var calendar = new FullCalendar.Calendar(calendarEl, {
      // plugins: [ 'dayGrid' ] // DON'T DO THIS ANYMORE!
    })
    calendar.render()
  })
</script>
```

You'll still need to include the CSS file. You won't need to define the `plugins` array anymore.

For initializing [scheduler](premium), do something like this:

```html
<link ref='fullcalendar-scheduler/main.css' rel='stylesheet' />
<script src='fullcalendar-scheduler/main.js'></script><!-- only one JS file. don't include the other bundle -->
<script>
  document.addEventListener('DOMContentLoaded', function() {
    var calendarEl = document.getElementById('calendar')
    var calendar = new FullCalendar.Calendar(calendarEl, {
      // plugins: [ 'resourceTimeline' ] // DON'T DO THIS ANYMORE!
    })
    calendar.render()
  })
</script>
```

When using the scheduler bundle, you don't need to include both the standard bundle <strong>AND</strong> the scheduler bundle. The scheduler bundle already <strong>INCLUDES</strong> the standard plugins.

The `main.js` file does <strong>NOT</strong> include the following plugins:

- `google-calendar` - write a separate script tag for `<bundle-dir>/google-calendar.js`
- `rrule` - write separate script tag for `<bundle-dir>/rrule.js` and the [rrule JS file](https://www.jsdelivr.com/package/npm/rrule?path=dist%2Fes5)
- `luxon` - write separate script tag for `<bundle-dir>/luxon.js` and the [luxon JS file](https://www.jsdelivr.com/package/npm/luxon?path=build%2Fglobal)
- `moment` - write separate script tag for `<bundle-dir>/moment.js` and the [moment JS file](https://www.jsdelivr.com/package/npm/moment)
- `moment-timezone` - write separate script tag for `<bundle-dir>/moment-timezone.js` and the [moment-timezone JS file(s)](https://www.jsdelivr.com/package/npm/moment-timezone)


## Toolbar

When specifying [header](header) and [footer](footer), the `{ left, center, right }` properties are still available to you. However, the following properties have been added to better support RTL locales:

- `start` - if the calendar is left-to-right, it will appear on the left. if right-to-left, it will appear on the right.
- `end` - if the calendar is left-to-right, it will appear on the right. if right-to-left, it will appear on the left.


## View Rendering

<table>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>viewSkeletonRender</li>
        <li class='diff-removed'>viewSkeletonDestroy</li>
      </ul>
    </td>
    <td>
      <p>
        Use the <a href='view-render-hooks'>view render hooks</a> instead:
      </p>
      <ul class='diff-list'>
        <li>
          <span class='diff-added'>viewClassNames</span>
          - for adding classNames to the root view element
        </li>
        <li>
          <span class='diff-added'>viewDidMount</span>
          - simply renamed from <span class='diff-removed'>viewSkeletonRender</span>
        </li>
        <li>
          <span class='diff-added'>viewWillUnmount</span>
          - simply renamed from <span class='diff-removed'>viewSkeletonDestroy</span>
        </li>
      </ul>
    </td>
  </tr>
</table>


## Date Rendering

The header elements above the day cells in daygrid and timegrid views. Also, the title elements for each day in list view. For the timeline view, jump down to <a href='#slot-rendering'>slot rendering</a>.

<table>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>columnHeader</li>
      </ul>
    </td>
    <td>
      <ul class='diff-list'>
        <li>
          <a href='dayHeaders' class='diff-added'>dayHeaders</a>
          - simply renamed. Accepts <code>true</code> or <code>false</code> for enabling headers.
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>columnHeaderFormat</li>
      </ul>
    </td>
    <td>
      <ul class='diff-list'>
        <li>
          <a href='dayHeaderFormat' class='diff-added'>dayHeaderFormat</a>
          - simply renamed
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>columnHeaderText</li>
        <li class='diff-removed'>columnHeaderHtml</li>
      </ul>
    </td>
    <td>
      <p>
        Use the <a href='day-header-render-hooks'>day-header render hooks</a> instead:
      </p>
      <ul class='diff-list'>
        <li>
          <span class='diff-added'>dayHeaderClassNames</span>
        </li>
        <li>
          <span class='diff-added'>dayHeaderContent</span>
          - for emulating <span class='diff-removed'>columnHeaderText</span>, return a string.
          For emulating <span class='diff-removed'>columnHeaderHtml</span>, return an object like <code>{ html: '' }</code>.
          The received arguments are different.
        </li>
        <li>
          <span class='diff-added'>dayHeaderDidMount</span>
        </li>
        <li>
          <span class='diff-added'>dayHeaderWillUnmount</span>
        </li>
      </ul>
    </td>
  </tr>
</table>

The days cells in daygrid and timegrid views:

<table>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>dayRender</li>
      </ul>
    </td>
    <td>
      <p>
        Use the <a href='day-cell-render-hooks'>day-cell render hooks</a> instead:
      </p>
      <ul class='diff-list'>
        <li>
          <span class='diff-added'>dayCellClassNames</span>
          - for injecting classNames
        </li>
        <li>
          <span class='diff-added'>dayCellContent</span>
          - for if you injected DOM content via <span class='diff-removed'>dayRender</span>
        </li>
        <li>
          <span class='diff-added'>dayCellDidMount</span>
          - for if you needed the DOM element in <span class='diff-removed'>dayRender</span>
        </li>
        <li>
          <span class='diff-added'>dayCellWillUnmount</span>
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td></td>
    <td>
      <ul class='diff-list'>
        <li>
          <a href='dayMinWidth' class='diff-added'>dayMinWidth</a>
          - creates a horizontal scrollbar if day cells get any narrower. For daygrid and timegrid views only. Requires the <code>@fullcalendar/scrollgrid</code> premium plugin.
        </li>
      </ul>
    </td>
  </tr>
</table>

<p id='slot-rendering'>
  The horizontal time slots in timegrid view or the vertical datetime slots in timeline view. In timeline view, even though these slots might represent distinct days, they are still considered "slots" as opposed to "day cells".
</p>

<table>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>minTime</li>
      </ul>
    </td>
    <td>
      <ul class='diff-list'>
        <li>
          <a href='slotMinTime' class='diff-added'>slotMinTime</a>
          - simply renamed
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>maxTime</li>
      </ul>
    </td>
    <td>
      <ul class='diff-list'>
        <li>
          <a href='slotMaxTime' class='diff-added'>slotMaxTime</a>
          - simply renamed
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>slotWidth</li>
      </ul>
    </td>
    <td>
      <ul class='diff-list'>
        <li>
          <a href='slotMinWidth' class='diff-added'>slotMinWidth</a> - simply renamed. same exact behavior. only applies to timeline view
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td></td>
    <td>
      <p>
        The following <a href='slot-render-hooks'>slot render hooks</a> are now available:
      </p>
      <ul class='diff-list'>
        <li class='diff-added'>slotLabelClassNames</li>
        <li class='diff-added'>slotLabelContent</li>
        <li class='diff-added'>slotLabelDidMount</li>
        <li class='diff-added'>slotLabelWillUnmount</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td></td>
    <td>
      <p>
        You can now customize the long span of content next to the slot's date/time text. In timegrid view, this is the horizontal space that passes under all of the days. In timeline view, this is the vertical space that passes through the resources. Use the following <a href='slot-render-hooks'>slot render hooks</a>:
      </p>
      <ul class='diff-list'>
        <li class='diff-added'>slotLaneClassNames</li>
        <li class='diff-added'>slotLaneContent</li>
        <li class='diff-added'>slotLaneDidMount</li>
        <li class='diff-added'>slotLaneWillUnmount</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>timeline slot classNames</td>
    <td>
      <p>
        In timeline view, slots now have more descriptive classNames like
        <code>fc-slot-future</code>,
        <code>fc-slot-past</code>,
        <code>fc-slot-fri</code>,
        <code>fc-slot-sat</code>,
        <code>fc-slot-today</code>,
        etc.
      </p>
    </td>
  </tr>
</table>

Date rendering, in aggregate:

<table>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>datesRender</li>
      </ul>
    </td>
    <td>
      <p>
        No direct replacement. Instead, handle <em>individual</em> date cells via the
        <a href='day-header-render-hooks'>day header</a>,
        <a href='day-cell-render-hooks'>day cell</a>, or
        <a href='slot-render-hooks'>slot</a>
        render hooks.
      </p>
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>datesDestroy</li>
      </ul>
    </td>
    <td>
      <p>
        No direct replacement. Instead, handle <em>individual</em> date cells via
        <span class='diff-added'>dayHeaderWillUnmount</span>,
        <span class='diff-added'>dayCellWillUnmount</span>, or
        <span class='diff-added'>slotLabelWillUnmount</span>.
      </p>
    </td>
  </tr>
</table>

Week numbers:

<table>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>weekNumbersWithinDays</li>
      </ul>
    </td>
    <td>
      <p>
        This is now the default behavior. The <code>weekNumbersWithinDays:false</code> behavior has been retired.
        <a href='{{ site.baseurl }}/docs/weekNumbersWithinDays'>See the old docs</a> for an illustration of the difference.
      </p>
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>weekLabel</li>
      </ul>
    </td>
    <td>
      <ul class='diff-list'>
        <li>
          <a href='weekText' class='diff-added'>weekText</a>
          - simply renamed. the text that gets prefixed to the formatted week number
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td></td>
    <td>
      <p>
        The following <a href='week-number-render-hooks'>week-number render hooks</a> are now available:
      </p>
      <ul class='diff-list'>
        <li class='diff-added'>weekNumberClassNames</li>
        <li class='diff-added'>weekNumberContent</li>
        <li class='diff-added'>weekNumberDidMount</li>
        <li class='diff-added'>weekNumberWillUnmount</li>
      </ul>
    </td>
  </tr>
</table>

The area where the "all-day" text is displayed, both in timegrid view and list view:

<table>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>allDayText</li>
        <li class='diff-removed'>allDayHtml</li>
      </ul>
    </td>
    <td>
      <p>
        Use the <a href='all-day-render-hooks'>all-day render hooks</a> instead:
      </p>
      <ul class='diff-list'>
        <li>
          <span class='diff-added'>allDayClassNames</span>
        </li>
        <li>
          <span class='diff-added'>allDayContent</span>
          - for emulating <span class='diff-removed'>allDayText</span>, assign a string. For emulating <span class='diff-removed'>allDayHtml</span>, assign an object like <code>{ html: '' }</code>
        </li>
        <li>
          <span class='diff-added'>allDayDidMount</span>
        </li>
        <li>
          <span class='diff-added'>allDayWillUnmount</span>
        </li>
      </ul>
    </td>
  </tr>
</table>


## Event Rendering

<table>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>eventRender</li>
        <li class='diff-removed'>eventDestroy</li>
      </ul>
    </td>
    <td>
      <p>
        Use the new <a href='event-render-hooks'>event render hooks</a> instead:
      </p>
      <ul class='diff-list'>
        <li>
          <span class='diff-added'>eventClassNames</span>
          - for injecting classNames
        </li>
        <li>
          <span class='diff-added'>eventContent</span>
          - for if you injected DOM content via <span class='diff-removed'>eventRender</span>
        </li>
        <li>
          <span class='diff-added'>eventDidMount</span>
          - for if you needed the DOM element in <span class='diff-removed'>eventRender</span>
        </li>
        <li>
          <span class='diff-added'>eventWillUnmount</span>
          - like <span class='diff-removed'>eventDestroy</span>, but receives additional arguments
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>eventPositioned</li>
      </ul>
    </td>
    <td>
      <p>
        No direct replacement. You can use <span class='diff-added'>eventDidMount</span> to know when the element
        has been inserted into the DOM, but it's no longer possible to know when its position has stabilized.
      </p>
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>_eventsPositioned</li>
      </ul>
    </td>
    <td>
      <p>
        No direct replacement.
      </p>
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li>
          <span class='diff-removed'>Calendar::rerenderEvents</span> method
        </li>
      </ul>
    </td>
    <td>
      Call <a href='Calendar-render' class='diff-unchanged'>Calendar::render</a> after initialization to rerender everything.
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>timeGridEventMinHeight</li>
      </ul>
    </td>
    <td>
      <p>
        Set a <code>min-height</code> on your event elements via CSS. The computed min-height is considered when positioning events.
      </p>
    </td>
  </tr>
  <tr>
    <td>
      <p>background events</p>
    </td>
<td markdown='1'>

Background events now display their title (addresses [#2746](https://github.com/fullcalendar/fullcalendar/issues/2746)). They previously did not. To prevent this, don't assign a `title` to the <a href='event-parsing'>Event Object Input</a>. Alternatively, you can override the rendering like this:

```js
eventContent: function(arg) {
  if (arg.event.rendering.match(/background/)) { // handles inverse-background too
    return null
  }
}
```

</td>
  </tr>
</table>

When there are too many events to fit within a single day:

<table>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>eventLimit</li>
      </ul>
    </td>
    <td>
      <ul class='diff-list'>
        <li>
          <a href='dayMaxEventRows' class='diff-added'>dayMaxEventRows</a>
          - the max number of stacked event levels within a given day. This <em>includes</em> the +more link if present. This is the same behavior as v4's <span class='diff-removed'>eventLimit</span>. Just as in v4, setting it to <code>true</code> uses the cell's natural dimensions to limit events.
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td></td>
    <td>
      <ul class='diff-list'>
        <li>
          <a href='dayMaxEvents' class='diff-added'>dayMaxEvents</a>
          - the max number of events within a given day, <em>not</em> counting the +more link (addresses <a href='https://github.com/fullcalendar/fullcalendar/issues/3035'>#3035</a>)
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>eventLimitClick</li>
      </ul>
    </td>
    <td>
      <ul class='diff-list'>
        <li>
          <a href='moreLinkClick' class='diff-added'>moreLinkClick</a>
          - renamed from <span class='diff-removed'>eventLimitClick</span>. No longer receives the <code>moreEl</code> or <code>dayEl</code> properties. The <code>segs</code> property has been rename to <code>allSegs</code>.
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>eventLimitText</li>
      </ul>
    </td>
    <td>
      <p>
        Use the <a href='more-link-render-hooks'>more link render hooks</a> instead:
      </p>
      <ul class='diff-list'>
        <li>
          <span class='diff-added'>moreLinkClassNames</span>
        </li>
        <li>
          <span class='diff-added'>moreLinkContent</span>
          - for emulating <span class='diff-removed'>eventLimitText</span>, return a string. The received arguments are different.
        </li>
        <li>
          <span class='diff-added'>moreLinkDidMount</span>
        </li>
        <li>
          <span class='diff-added'>moreLinkWillUnmount</span>
        </li>
      </ul>
    </td>
  </tr>
</table>


## Resource Rendering

<table>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>resourceText</li>
        <li class='diff-removed'>resourceRender</li>
      </ul>
    </td>
    <td>
      <p>
        A resource "label" is anywhere the name of a resource is displayed. They exist in the header of vertical resource view and the side section of resource timeline view.
      </p>
      <p>
        Use the following <a href='resource-render-hooks'>resource render hooks</a> going forward:
      </p>
      <ul class='diff-list'>
        <li>
          <span class='diff-added'>resourceLabelClassNames</span>
          - for injecting classNames
        </li>
        <li>
          <span class='diff-added'>resourceLabelContent</span>
          - for if you injected DOM content via <span class='diff-removed'>resourceRender</span>. For emulating <span class='diff-removed'>resourceText</span>, return a string. The received arguments are different.
        </li>
        <li>
          <span class='diff-added'>resourceLabelDidMount</span>
          - for if you needed the DOM element in <span class='diff-removed'>resourceRender</span>
        </li>
        <li>
          <span class='diff-added'>resourceLabelWillUnmount</span>
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td></td>
    <td>
      <p>
        A resource "lane" is an element in resource-timeline view. It runs horizontally across the timeline slots for each resource.
      </p>
      <p>
        The following <a href='resource-render-hooks'>resource render hooks</a> are now available:
      </p>
      <ul class='diff-list'>
        <li>
          <span class='diff-added'>resourceLaneClassNames</span>
        </li>
        <li>
          <span class='diff-added'>resourceLaneContent</span>
        </li>
        <li>
          <span class='diff-added'>resourceLaneDidMount</span>
        </li>
        <li>
          <span class='diff-added'>resourceLaneWillUnmount</span>
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li>
          <span class='diff-removed'>Calendar::rerenderResources</span> method
        </li>
      </ul>
    </td>
    <td>
      <p>
        Call <a href='Calendar-render' class='diff-unchanged'>Calendar::render</a> after initialization to rerender everything.
      </p>
    </td>
  </tr>
</table>

When resources are grouped together in resource-timeline view:

<table>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>resourceGroupText</li>
      </ul>
    </td>
    <td>
      <p>
        A resource group "label" is where a group's name is displayed.
      </p>
      <p>
        Use the following <a href='resource-group-render-hooks'>resource group render hooks</a> going forward:
      </p>
      <ul class='diff-list'>
        <li>
          <span class='diff-added'>resourceGroupLabelClassNames</span>
        </li>
        <li>
          <span class='diff-added'>resourceGroupLabelContent</span>
          - you can emulate <span class='diff-removed'>resourceGroupText</span> by returning a string. The received arguments are different.
        </li>
        <li>
          <span class='diff-added'>resourceGroupLabelDidMount</span>
        </li>
        <li>
          <span class='diff-added'>resourceGroupLabelWillUnmount</span>
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td></td>
    <td>
      <p>
        A resource group "lane" is the horizontal area running along the time slots.
      </p>
      <p>
        The following <a href='resource-group-render-hooks'>resource group render hooks</a> are now available:
      </p>
      <ul class='diff-list'>
        <li class='diff-added'>resourceGroupLaneClassNames</li>
        <li class='diff-added'>resourceGroupLaneContent</li>
        <li class='diff-added'>resourceGroupLaneDidMount</li>
        <li class='diff-added'>resourceGroupLaneWillUnmount</li>
      </ul>
    </td>
  </tr>
</table>

The area on the side of resource-timeline view that contains resource names and data.

<table>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>resourceLabelText</li>
      </ul>
    </td>
    <td>
      <p>
        The "resource-area header" is above the resource data and displays the text "Resources" by default. It was previously called the "resource label", a term which is now being used to describe something else! When <a href='resourceAreaColumns' class='diff-added'>resourceAreaColumns</a> is activated, it will not be displayed.
      </p>
      <p>
        Use the <a href='resource-area-header-render-hooks'>resource-area header render hooks</a> going forward:
      </p>
      <ul class='diff-list'>
        <li>
          <span class='diff-added'>resourceAreaHeaderClassNames</span>
        </li>
        <li>
          <span class='diff-added'>resourceAreaHeaderContent</span>
          - for emulating <span class='diff-removed'>resourceLabelText</span>, assign a string
        </li>
        <li>
          <span class='diff-added'>resourceAreaHeaderDidMount</span>
        </li>
        <li>
          <span class='diff-added'>resourceAreaHeaderWillUnmount</span>
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>resourceColumns</li>
      </ul>
    </td>
    <td>
      <ul class='diff-list'>
        <li>
          <a href='resourceAreaColumns' class='diff-added'>resourceAreaColumns</a> - simply renamed
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>
      <p>
        Properties within <span class='diff-removed'>resourceColumns</span>:
      </p>
      <ul class='diff-list'>
        <li class='diff-removed'>labelText</li>
      </ul>
    </td>
    <td>
      <p>
        Renamed to these properties within <a href='resourceAreaColumns' class='diff-added'>resourceAreaColumns</a>:
      </p>
      <ul class='diff-list'>
        <li>
          <span class='diff-added'>headerClassNames</span>
        </li>
        <li>
          <span class='diff-added'>headerContent</span>
          - for emulating <span class='diff-removed'>labelText</span>, assign a string.
        </li>
        <li>
          <span class='diff-added'>headerDidMount</span>
        </li>
        <li>
          <span class='diff-added'>headerWillUnmount</span>
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>
      <p>
        Properties within <span class='diff-removed'>resourceColumns</span>:
      </p>
      <ul class='diff-list'>
        <li class='diff-removed'>text</li>
        <li class='diff-removed'>render</li>
      </ul>
    </td>
    <td>
      <p>
        Renamed to these properties within <a href='resourceAreaColumns' class='diff-added'>resourceAreaColumns</a>:
      </p>
      <ul class='diff-list'>
        <li>
          <span class='diff-added'>cellClassNames</span> - for injecting classNames
        </li>
        <li>
          <span class='diff-added'>cellContent</span>
          - for if you injected DOM content via <span class='diff-removed'>render</span>. For emulating <span class='diff-removed'>text</span>, return a string. The received arguments have changed.
        </li>
        <li>
          <span class='diff-added'>cellDidMount</span>
          - for if you needed the DOM element in <span class='diff-removed'>render</span>
        </li>
        <li>
          <span class='diff-added'>cellWillUnmount</span>
        </li>
      </ul>
    </td>
  </tr>
</table>


## List View Rendering

In list view, the "No events to display" message.

<table>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>listDayAltFormat</li>
      </ul>
    </td>
    <td>
      <ul class='diff-list'>
        <li>
          <a href='listDaySideFormat' class='diff-added'>listDaySideFormat</a>
          - simply renamed
        </li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>
      <ul class='diff-list'>
        <li class='diff-removed'>noEventsMessage</li>
      </ul>
    </td>
    <td>
      <p>
        The following <a href='no-events-render-hooks'>render hooks</a> are now available:
      </p>
      <ul class='diff-list'>
        <li>
          <span class='diff-added'>noEventsClassNames</span>
        </li>
        <li>
          <span class='diff-added'>noEventsContent</span>
          - for emulating <span class='diff-removed'>noEventsMessage</span>, assign a string
        </li>
        <li>
          <span class='diff-added'>noEventsDidMount</span>
        </li>
        <li>
          <span class='diff-added'>noEventsWillUnmount</span>
        </li>
      </ul>
    </td>
  </tr>
</table>


## Now Indicator Rendering

<table>
  <tr>
    <td></td>
    <td>
      <p>
        The following <a href='now-indicator-render-hooks'>render hooks</a> are now available for customizing the <a href='nowIndicator'>now indicator</a>:
      </p>
      <ul class='diff-list'>
        <li class='diff-added'>nowIndicatorClassNames</li>
        <li class='diff-added'>nowIndicatorContent</li>
        <li class='diff-added'>nowIndicatorDidMount</li>
        <li class='diff-added'>nowIndicatorWillUnmount</li>
      </ul>
    </td>
  </tr>
</table>


## Calendar Sizing

<table>
  <tr>
    <td>
      <p><a href='contentHeight' class='diff-unchanged'>contentHeight</a></p>
    </td>
    <td>
      <p>
        No longer accepts a function. Reassign imperatively via <a href='dynamic-options#setting'>setOption</a>.
      </p>
    </td>
  </tr>
  <tr>
    <td>
      <p><a href='height' class='diff-unchanged'>height</a></p>
    </td>
    <td>
      <p>
        No longer accepts a function. Reassign imperatively via <a href='dynamic-options#setting'>setOption</a>.
      </p>
      <p>
        No longer accepts the <code>'parent'</code> value. Instead, assign <code>'100%'</code> (addresses <a href='https://github.com/fullcalendar/fullcalendar/issues/4650'>#4650</a>). Any other valid css values are accepted as well.
      </p>
    </td>
  </tr>
</table>


## Custom JS Views

Custom views written as JavaScript classes will need to be refactored to work. Subclasses of `View` are no longer accepted. Instead, you [specify a plain configuration object](custom-view-with-js#config-object).

In the old way, you had different methods that were called when different pieces of data changed:

```js
class CustomView extends View { // a class. this is the OLD way

  renderSkeleton() {
    this.el.classList.add('custom-view')
    this.el.innerHTML =
      '<div class="view-title"></div>' +
      '<div class="view-events"></div>'
  }

  renderDates(dateProfile) {
    this.el.querySelector('.view-title').innerHTML = dateProfile.currentRange.start.toUTCString()
  }

  renderEvents(eventStore) {
    this.el.querySelector('.view-events').innerHTML = Object.keys(eventStore).length + ' events'
  }

}
```

Now, the `content` function gets called when ANY change occurs:

```js
const CustomViewConfig = { // a plain object. this is the NEW way

  classNames: [ 'custom-view' ],

  content: function(props) {
    let html =
      '<div class="view-title">' +
        props.dateProfile.currentRange.start.toUTCString() +
      '</div>' +
      '<div class="view-events">' +
        Object.keys(props.eventStore).length + ' events' +
      '</div>'

    return { html: html }
  }

}
```

You can return any of the available [content injection formats](content-injection) such as HTML, real DOM nodes, or virtual DOM nodes.

If you want to maintain state across calls to `content`, you are better off writing a Preact/React component instead. [More information &raquo;](custom-view-with-js#component)


## Other Misc Changes

- **feature:** you can force rerendering of anything on the calendar by calling the `Calendar::render` method again after initialization
- **fix:** `rerenderDelay` causes selectable and editable lag ([#4770](https://github.com/fullcalendar/fullcalendar/issues/4770))
- **fix:** CSP doesn't allow setting of inline CSS ([#4317](https://github.com/fullcalendar/fullcalendar/issues/4317))
- **fix:** when `eventSourceSuccess` callback throws error, looks like JSON parsing failed ([#4947](https://github.com/fullcalendar/fullcalendar/pull/4947))
- **fix:** always show more-link when supplying `0` ([#2978](https://github.com/fullcalendar/fullcalendar/issues/2978))


## Upgrading from V3

Many developers will be upgrading from <strong>v3</strong> instead of v4. We will likely release a separate guide for this process before the official v5 is released. In the meantime, here are some tips for upgrading from v3 -> v5 in lieu of a full guide:

1. Follow the [v3 -> v4 upgrade guide]({{ site.baseurl }}/docs/upgrading-from-v3) but ignore the following areas:
  - "Initialization" and anything related to `<script>` tags or stylesheets
  - anything related to content injection, such as options with the words `render`, `text`, or `html` in them
2. Learn how to install and initialize a v5 calendar from the [Getting Started article](getting-started).
2. Follow this v4 -> v5 upgrade guide afterwards.


## React Connector

When using the React connector, you can now return virtual DOM nodes to customize rendering ([react#12](https://github.com/fullcalendar/fullcalendar-react/issues/12)):

```jsx
import FullCalendar from '@fullcalendar/react'
import dayGridPlugin from '@fullcalendar/daygrid'

export const DemoApp(props) => (
  <div className='wrapper'>
    <FullCalendar
      plugins={[dayGridPlugin]}
      eventContent={(arg) => (
        <div class='custom-event-content'>
          <b>{arg.timeText}</b>
          <i>{arg.event.title}</i>
        </div>
      )}
    />
  </div>
)
```


## Vue Connector

Previouly, when using the Vue connector, you specified each option as its own attribute in your template:

```vue
<!-- the old way -->
<FullCalendar
  :weekends='false'
  @dateClick='handleDateClick'
/>
```

Now, you specify them as a single root options object:

```vue
<!-- the new way -->
<FullCalendar
  :options='calendarOptions'
/>
```

Of course you'll need to create this object somewhere:

```js
const AppComponent = {
  data: function() {
    return {
      calendarOptions: {
        weekends: false,
        dateClick: this.handleDateClick.bind(this) // binding is important!
      }
    }
  },
  methods: {
    handleDateClick: function(arg) {
      alert('clicked on ' + arg.dateStr)
    }
  }
}
```

This results in less duplication between your Vue component's JS and template ([vue#47](https://github.com/fullcalendar/fullcalendar-vue/issues/47)). It also thankfully blurs the distinction between props and "handlers" for which you'd need to use `v-on` or `@`. All properties are treated equally, resuling in a simpler API.

You now have the ability to customize rendering with the use of [Vue scoped slots](https://vuejs.org/v2/guide/components-slots.html#Scoped-Slots) (addresses [vue#14](https://github.com/fullcalendar/fullcalendar-vue/issues/14)):

```vue
{% raw %}<FullCalendar :options='calendarOptions'>
  <template v-slot:eventContent='arg'>
    <b>{{ arg.timeText }}</b>
    <i>{{ arg.event.title }}</i>
  </template>
</FullCalendar>{% endraw %}
```

This is possible with any of the `*Content` options in the API.


## Angular Connector

Previouly, when using the Angular connector, you specified each option as its own attribute in your template:

```ng
<!-- the old way -->
<full-calendar
  [weekends]="false"
  (dateClick)="handleDateClick"
></full-calendar>
```

Now, you specify them as a single root options object:

```ng
<!-- the new way -->
<full-calendar
  [options]="calendarOptions"
></full-calendar>
```

Of course you'll need to create this object somewhere:

```js
class AppComponent {

  calendarOptions = {
    weekends: false,
    dateClick: this.handleDateClick.bind(this) // binding is important!
  }

  handleDateClick(arg) {
    alert('clicked on ' + arg.dateStr)
  }

}
```

This results in less duplication between your Angular component's JS and template. It also thankfully blurs the distinction between props and "handlers" for which you'd need to use `(parentheses)` instead of `[brackets]`. All properties are treated equally, resuling in a simpler API.