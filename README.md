<center>
  <img src="./assets/logo.svg" height="200">
</center>

# Wombat
### Flat file, headless CMS for building configurable API
Simple open-source serverless Content Management Framework (headless-CMS) to build powerful API effortlessly.

## Features
- **Simple** - Designed to replace Wordpress in building simple company / product / event pages
- **Fast** - Serverless oriented, so performance is the key
- **Lightweight** - Just 3 dependencies, no database or other system-wide requirements
- **Two types of data** - Unlike Contentful or Strapi, offers not only items collection, but also single entities like pages or configs
- **Full internationalization** - Every piece content is assigned to one language, so you have complete freedom in customizing it
- **No strict schema** - Do you need each item within a content type to do be different and have a different structure for each language? No problem, sky is the limit!
- **No Admin Panel** - You are not limited by the UI or poor UX, just you and bunch of JSON and Markdown files
- **Front-end agnostic** - Simple REST API can be used by any application

## Setup
1. Create content following [Content structure](#content-structure) description.
2. Create `/public` directory to host static assets.
3. (Optional) Create `conifg.json` file.
4. Chose serverless platform to host your functions, for example Lambda function on [ZEIT Now](https://zeit.co/now)

## Serverless deployment
This guide is for [ZEIT Now](https://zeit.co/docs/v2/deployments/official-builders/node-js-now-node/), but setting up any other serverless env looks simillar.

1. In your project with `now.json` create an `api/` directory
2. Inside run `yarn init` and then `yarn add @snowdog/wombat`
3. In `api/package.json` add automatic DB building and option to run development server
   ```
    "scripts": {
        "postinstall": "wombat build",
        "dev": "wombat start"
    }
   ```
3. Define new builds and routes in `now.json`
   ```
    {
        "builds": [
            { "src": "api/*.js", "use": "@now/node" }
        ],
        "routes": [
            { "src": "/api/entity/(?<name>[^/]*)", "dest": "/api/entity.js?name=$name" },
            { "src": "/api/collection/(?<name>[^/]*)", "dest": "/api/collection.js?name=$name" }
        ]
    }
   ```
4. Copy [collection.js](./examples/now/collection.js) and [entity.js](./examples/now/entity.js) from examples to `/api` directory
5. Deploy your app via `now`

## Config options
* `defaultLang` (default `en`) - Fallback language when request is send without `lang` query param.
* `port` (default `3000`, only for development purpouses) Port used by the web server to listen for requests.

## Content structure
All content is kept in `/content` directory.
Inside you need to define supported languages. For example, we want to have content just in English, so it will be `/content/en`.

Wombat supports two data types:
### Collections
Designed to store sets of similar data, like blog posts, authors, job offers etc.
- Collections are stored in `collection` directory inside each language.
- Each collection and item of collection needs to be added as a directory.
- Every property of an item collection needs to be a separate JSON or Markdown file.

### Entities
Created to keep a single object like a landing page content or global configuration.
- Entities are stored in `entity` directory inside each language.
- Each entity needs to be added as a directory.
- Every property of entity needs to be a separate JSON or Markdown file.
- You can define the relation between entity and collection.

### How to define a relation between entity and collection
Create new prop JSON file with relation config:
```js
{
  "collectionName": "collectionName", // required
  "filter": {
    "items": ["award", "about", "partner"], // (optional) Array of collection items IDs. Items order is preserved.
    "sortBy": "title", // (optional) prop name used for sorting
    "sort": "asc", // (optional) `desc` is default - `sortBy` required to use it
    "limit": 2 // (optional) - `sortBy` required to use it
  }
}
```

### Example content tree
```
content
└── en
    ├── collection
    |   └── blog
    |       └── why-wombat-poop-is-cube
    |       |   ├── content.md
    |       |   ├── featured-image.json
    |       |   └── title.json
    |       └── things-you-dont-know-about-wombats
    |           ├── content.md
    |           ├── featured-image.json
    |           ├── form-config.json
    |           └── title.json
    └── entity
        └── home
            ├── blog-posts.json
            ├── about.md
            └── hero-banner.json
```
## API
### `/entity/{name}`
Retrieve an entity item.

**Example:**
To get `home` send request to `/entity/home`.

**URL params:**
- `lang` - Return content in given lang.

### `/collection/{name}`
Retrieve the whole collection as array.

**Example:**
To get `blog` send request to `/collection/blog`

**URL params:**
- `lang` - Return content in given lang.
- `items` - List of coma separated IDs of collection items. Items order is preserved.
- `sortBy` - Prop name used for sorting.
- `sort` - Can be set to `asc`. `sortBy` required to use it.
- `limit` - Limit numer or returned collection items. `sortBy` required to use it.
