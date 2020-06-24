---
title: 'Front-end Setup'
---

We're going to be using the Next.js blog starter as the base of our project. By default, it pulls all its data from the filesystem, but we'll want to make some changes to have it pull data from Strapi's GraphQL API.

We want to _keep Strapi running_ throughout this entire process. So, open a new terminal and create a new project alongside Strapi using the following command:

```bash
yarn create next-app -e blog-starter tina-strapi-blog
```

Navigate into the new directory and start up the website's dev server.

```bash
cd tina-strapi-blog
yarn dev
```

Navigate to [http://localhost:3000](http://localhost:3000) and see what we'll be working with.

The blog is made up of two main pieces: an index page, and the blog post pages. Our goal will be to have the data on both of these pages come from Strapi, instead of from the filesystem.

## Setup Environment Variable

Our front-end has to know how to reach Strapi, so we'll set up an environment variable that will allow us to configure this. First, install `dotenv`

```bash
yarn add dotenv
```

Then in the root of the project create a file called `.env` and fill in the Strapi URL.

**.env**

```.env
STRAPI_URL=http://localhost:1337
```

Also at the root of the project create a new file called `next.config.js` and populate it with

**next.config.js**

```js
require('dotenv').config()

module.exports = {
  env: {
    STRAPI_URL: process.env.STRAPI_URL,
  },
}
```

Whenver we start up our front-end project, this URL will be loaded into our environment and we'll have access to Strapi's location wherever we end up needing it.

## Add Tina

Let's start getting Tina in place to let us use our editing experience a bit down the road. First, install Tina and the `styled-component` peer dependency.

```bash
yarn add tinacms styled-components
```

### Add the Provider

Go into `pages/_app.js`, we're going to wrap what's currently being returned here with our Tina and Strapi providers.

Delete what the `MyApp` function is returning and setup up the CMS object to use Strapi.

```js
import "../styles/index.css";

import {
  StrapiMediaStore,
  TinaStrapiClient,
} from "react-tinacms-strapi-bm-test"; // @TODO
import { TinaCMS } from "tinacms";

export default function MyApp({ Component, pageProps }) {
  const cms = new TinaCMS({
    sidebar: { hidden: true },
    apis: {
      strapi: new TinaStrapiClient(process.env.STRAPI_URL),
    },
    media: {
      store: new StrapiMediaStore(process.env.STRAPI_URL),
    },
  });
  return (
    /* We'll fill this out soon */
  );
}
```

We're instantiating the CMS object, giving it access to a Strapi client, and also giving it access to a Strapi media store. This will let us communicate with Strapi's APIS and help Tina know how to upload media to the Strapi server.

We want to return the page content, wrapped in providers so that every page has access to the CMS and Strapi client..

```js
import {
  StrapiMediaStore,
  StrapiProvider,
  TinaStrapiClient,
} from 'react-tinacms-strapi-bm-test' @TODO
import { TinaCMS, TinaProvider } from 'tinacms'

// ...

return (
  <TinaProvider cms={cms}>
    <StrapiProvider
      editMode={false}
      enterEditMode={() => {}}
      exitEditMode={() => {}}
    >
      <Component {...pageProps} />
    </StrapiProvider>
  </TinaProvider>
)
```

We're fudging some values for `editMode`, `enterEditMode`, and `exitEditMode` so we can focus on pulling data from Strapi. We'll come back to this when we're ready to get editing working.

After doing this, you should be able to refresh the browser and notice that _nothing_ has changed! But under-the-hood, we've gotten Tina set up and are ready to start using it.