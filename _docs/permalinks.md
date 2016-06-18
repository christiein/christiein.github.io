---
layout: docs
title: Permalinks
permalink: /docs/permalinks/
---

Jekyll supports a flexible way to build your site’s URLs. You can specify the
permalinks for your site through the [Configuration](../configuration/) or in
the [YAML Front Matter](../frontmatter/) for each post. You’re free to choose
one of the built-in styles to create your links or craft your own. The default
style is `date`.

Permalinks are constructed by creating a template URL where dynamic elements
are represented by colon-prefixed keywords. For example, the default `date`
permalink is defined according to the format `/:categories/:year/:month/:day/:title.html`.

## Template variables

<div class="mobile-side-scroller">
<table>
  <thead>
    <tr>
      <th>Variable</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <p><code>year</code></p>
      </td>
      <td>
        <p>Year from the Post’s filename</p>
      </td>
    </tr>
    <tr>
      <td>
        <p><code>month</code></p>
      </td>
      <td>
        <p>Month from the Post’s filename</p>
      </td>
    </tr>
    <tr>
      <td>
        <p><code>i_month</code></p>
      </td>
      <td>
        <p>Month from the Post’s filename without leading zeros.</p>
      </td>
    </tr>
    <tr>
      <td>
        <p><code>day</code></p>
      </td>
      <td>
        <p>Day from the Post’s filename</p>
      </td>
    </tr>
    <tr>
      <td>
        <p><code>i_day</code></p>
      </td>
      <td>
        <p>Day from the Post’s filename without leading zeros.</p>
      </td>
    </tr>
    <tr>
      <td>
        <p><code>short_year</code></p>
      </td>
      <td>
        <p>Year from the Post’s filename without the century.</p>
      </td>
    </tr>
    <tr>
      <td>
        <p><code>hour</code></p>
      </td>
      <td>
        <p>
          Hour of the day, 24-hour clock, zero-padded from the post’s <code>date</code> front matter. (00..23)
        </p>
      </td>
    </tr>
    <tr>
      <td>
        <p><code>minute</code></p>
      </td>
      <td>
        <p>
          Minute of the hour from the post’s <code>date</code> front matter. (00..59)
        </p>
      </td>
    </tr>
    <tr>
      <td>
        <p><code>second</code></p>
      </td>
      <td>
        <p>
          Second of the minute from the post’s <code>date</code> front matter. (00..59)
        </p>
      </td>
    </tr>
    <tr>
      <td>
        <p><code>title</code></p>
      </td>
      <td>
        <p>
            Title from the document’s filename. May be overridden via
            the document’s <code>slug</code> YAML front matter.
        </p>
      </td>
    </tr>
    <tr>
      <td>
        <p><code>slug</code></p>
      </td>
      <td>
        <p>
            Slugified title from the document’s filename ( any character
            except numbers and letters is replaced as hyphen ). May be
            overridden via the document’s <code>slug</code> YAML front matter.
        </p>
      </td>
    </tr>
    <tr>
      <td>
        <p><code>categories</code></p>
      </td>
      <td>
        <p>
          The specified categories for this Post. If a post has multiple
          categories, Jekyll will create a hierarchy (e.g. <code>/category1/category2</code>).
          Also Jekyll automatically parses out double slashes in the URLs,
          so if no categories are present, it will ignore this.
        </p>
      </td>
    </tr>
  </tbody>
</table>
</div>


