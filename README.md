# LFRIS Marketing Docs

Production build status: [![Netlify Status](https://api.netlify.com/api/v1/badges/a37de724-defa-4a86-a2cc-3267dc619447/deploy-status)](https://app.netlify.com/sites/lfrism-doc/deploys)

This is the repo for Liferay Marketing Team's documentation site: https://mktg-docs.liferay.com/

## Getting Started

1. `git clone https://github.com/liferay/lfris-marketing-docs.gits`
2. `npm i`
3. Create `.env.development` and `.env.production` files, ask a webteam member for the necessary environment variables
4. Start development server `npm run dev` - go to `http://localhost:8000`
5. Start a production server `npm run build` then `npm run serve` and go to `http://localhost:9000`

For additional information: [Gatsby Documentation](https://www.gatsbyjs.org/docs/)

## Implementation Details

### Gatsby

This site is built with [gatsby](https://www.gatsbyjs.org/), a react-based static site generator. Please use their to [tutorial](https://www.gatsbyjs.org/tutorial/) to familiarize yourself.

We initialized this site with [this starter](https://github.com/diegonvs/gatsby-boilerplate), but the site evolved to better fit our use case and no longer follows most of what is provided in that starter.

Instead, we've based our architecture (atomic design) on mktg-docs.liferay.com's sister site: [liferay.design](https://liferay.design/). [Repo Link](https://github.com/liferay-design/liferay.design/).

### Netlify

We host our site on static-site host, [Netlify](http://netlify.com/). Webteam has login credentials. We manage deployment and authentication through netlify.

### Auth

#### Logging In

All pages except for the homepage are gated. Only users signed in with `@liferay.com` email addresses have access. Users can login through netlify by using the sign up tab, but it is recommended (and easier) to sign in using the `continue with google` button since `@liferay.com` emails are google accounts anyways.

Netlify Identity is unavailable locally so we have gating turned off locally.

#### Netlify Identity

We use [Netlify Identity](https://docs.netlify.com/visitor-access/identity/) as our authentication service.

[`Auth`](https://github.com/liferay/lfris-marketing-docs/blob/master/src/components/organisms/Auth/index.js) is a wrapper component that renders content if the user is logged in or renders a login screen if they aren't. It uses `react-netlify-identity-widget` to get users' login information.

Documentation for `react-netlify-identity`: https://github.com/sw-yx/react-netlify-identity

Documentation for `react-netlify-identity-widget`: https://github.com/sw-yx/react-netlify-identity-widget

Code Demo Video: https://www.youtube.com/watch?v=vrSoLMmQ46k

Gatsby Demo Repo: https://github.com/jlengstorf/gatsby-netlify-identity-functions

#### How to Ungate/Gate a page

By default https://mktg-docs.liferay.com/search, https://mktg-docs.liferay.com/deploy, and all pages generated by Google Docs are gated.

To ungate pages generated by Google Docs:

1. In Google Drive, select the document you wish to ungate
2. Click the 'i' icon
3. Fill the description with the following JSON Object:

```
{
  "needsAuth": false,
}
```

### Data

Since we are using Gatsby, Graphql manages our data/ is our source of data.

#### Google Docs

Most of the purpose of building this site out has been to provide marketers with an easy way of writing documentation. Since they largely use Google Docs, we've decided to use Google Docs as our main source of data.

We use [gatsby-source-google-docs](https://www.gatsbyjs.org/packages/gatsby-source-google-docs/?=gatsby-source-google-docs) to parse and convert Google Docs' data to markdown.

#### Metadata

Users can add additional data to their files by adding a JSON object to the description field. [Documentation](https://www.gatsbyjs.org/packages/gatsby-source-google-docs/#add-extra-data). None of these are required, but if users want to take the extra step, they can use these fields.

Here is an example of what to place into the description:

```
{
	"path": "/web/general/events",
	"title": "General Events",
	"description": "This is the document for general insights into how to set up events.",
	"needsAuth": "false"
}
```

Here is a table for the available fields and what they are:

| Field       | Description                                                              | Type                                |
| ----------- | ------------------------------------------------------------------------ | ----------------------------------- |
| path        | sets the desired url path for the document                               | string ex: /test/test/              |
| title       | overrides the default title                                              | string                              |
| Date        | overrides the default date                                               | string                              |
| description | sets the description for the doc (appears in search results and indeces) | string                              |
| author      | author name                                                              | string                              |
| needsAuth   | sets if the doc will be gated                                            | boolean (don't use quotation marks) |

##### Additional Notes

If you want to set up your own documentation site, follow the set up instructions found in the above link and use the relevant environment variable names (found in `gatsby-config.js`)

If you want to contribute to open source, help improve the plugin, or debug problems, it's been useful to look through the google docs api documentation: https://developers.google.com/docs/api/reference/rest

Sidenote: `gatsby-source-google-docs` sources data from google drive and google docs (hence the need for both api), transforms that into JSON, uses [json2md](https://www.npmjs.com/package/json2md) to convert the json into markdown so that the document's data can be available through `allMarkdownRemark`

#### Markdown Files

Pages can also be created through markdown files in the `content` folder. The fields listed above can be used in frontmatter of your markdown files.

We currently have `gatsby-source-filesystem` turned off, so uncomment that plugin in `gatsby-config.js` to turn this functionality back on.

### Theme

The `/theme` is based off the `foundations` theme-contributor in lfris-www (liferay.com). We are extending clay with it, so bootstrap is available as well.

More specific styles are done through CSS Modules and `styles.module.scss` files.

### Search

#### Elastic Lunr Search

We elastic's lunr search [plugin](https://www.gatsbyjs.org/packages/@gatsby-contrib/gatsby-plugin-elasticlunr-search/) to make our search indexes available in graphql.

[`<Search>`](https://github.com/liferay/lfris-marketing-docs/tree/master/src/components/molecules/Search) manages all things related to search. It has two views, one for use in the navigation and on for use on https://mktg-docs.liferay.com/search.

`<SearchForm>` is for user input and `<SearchResults>` displays the results. `index.js` manages the state for both.

We should re-factor to use algolia docSearch at some point.

### Misc plugins and packages

[`gatsby-image`](https://www.gatsbyjs.org/packages/gatsby-image/) and [`gatsby-plugin-sharp`](https://www.gatsbyjs.org/packages/gatsby-plugin-sharp/?=gatsby-plugin-sharp) to serve dynamic media.

[`gatsby-plugin-layout`](https://www.gatsbyjs.org/packages/gatsby-plugin-layout/) to have gatsby v1 layout functionality. This allows for page transitions.

[`gatsby-plugin-zopfli`](https://www.gatsbyjs.org/packages/gatsby-plugin-zopfli/?=zopfli) a google compression plugin that serves gzip versions of assets

[`gatsby-remark-default-html-attrs`](https://www.gatsbyjs.org/packages/gatsby-remark-default-html-attrs/?=gatsby-remark-default-h) to add classe to markdown tags on build (gives `h1` from google docs classes)

[`gatsby-source-filesystem`](https://www.gatsbyjs.org/packages/gatsby-source-filesystem/?=gatsby-source-filesystem) not in use currently

[`gatsby-source-google-docs`](https://www.gatsbyjs.org/packages/gatsby-source-google-docs/?=gatsby-source-google-docs) Source data from google docs and google drive

[`gatsby-transformer-remark`](https://www.gatsbyjs.org/packages/gatsby-transformer-remark/?=gatsby-transformer-remark) Parses markdown files to html

[`html-react-parser`](https://www.npmjs.com/package/html-react-parser) used on recommendation of `gatsby-source-google-docs`. Parses html and replaces it with react. We replace h2 tags with something that will generate our sidemenu, and we use it to replace normal images with Gatsby Images.

[`lodash.startcase`](https://lodash.com/docs/4.17.15#startCase) Single lodash method to convert kebab case path names to startcase for display in our sidebar.

[`react-scroll`](https://www.npmjs.com/package/react-scroll) Scroll animation for the sidemenu in article views.

[`sanitize-html`](https://www.npmjs.com/package/sanitize-html) Sanitizes the html genrated by google docs to protect from XSS attacks and stray html written in google docs. We allow all attributes and added a couple of tags that shouldn't be sanitized.

[`gatsby-plugin-resolve-src`](https://www.gatsbyjs.org/packages/gatsby-plugin-resolve-src/) Makes it easier to import components. We can just write `components/atoms` instead of `../components/atoms`.
