# Glossary definition

We should create a centralized glossary of terminology, so we aren't duplicating content across the site. A centralized glossary also helps keep internal and external product language consistent as a company grows. Additionally, it can be used to integrate documentation into a Dashboard.

## Format

In my mind, the glossary would be a JSON file with entries that look like the following:

```
{
  "route": {
    "title": "Route",
    "definition": "A technique that modern web applications use to allow you to assign URLs to functions easily. In Flask, routes are mapped to Python functions. Imagine you’re visiting the Deepgram website and want to go to the Why Deepgram? section at deepgram.com/why-deepgram. Our servers know to serve you that specific page because a web application running on the server knows that when you go to our website and navigate to the Why Deepgram? section, it should handle the request, figure out what to do, and send a page back. The logic behind deciding which page to send back is performed by a route. Navigating to deepgram.com/why-deepgram uses the /why-deepgram route, navigating to deepgram.com/pricing uses the /pricing route, and so on.",
    "short": "A technique that modern web applications use to allow you to assign URLs to functions easily. In Flask, routes are mapped to Python functions."
    "associated-page": "/routes"
  }
  ```
  
  ## Components
  
  For each key, the following components should exist:
  
  | Name | Description | 
  |-|-|
  | `title` | Display name of the term being defined. Shows at the top of the definition entry. | 
  | `definition` | Long definition for the term being defined. Used to embed definitions in beginner topics, and also appears at the top of search results. Used in the main glossary, which lists all terms. |
  | `short` | Short definition for the term being defined. Appears in tooltips in intermediate/advanced topics. |
  | `associated-page` | URL to an internal page with further information, if one exists. If not empty, adds the line "To learn more, see", then looks up the title of the page provided in the URL and provides a link using that Title and the URL. For this example, it would look like: **To learn more, see [Web Application Routes](/routes)**. | 
  
