---
title: Hello World!
date: "2020-01-26"
description: "Day #1 of 100 Days of Gatsby"
---

This is day #1 of my journey through [100 Days Of Gatsby](https://www.gatsbyjs.org/blog/100days/). The challenge of the first day was to [build a blog](https://www.gatsbyjs.org/blog/100days/start-blog/), and this post is the first post of that blog. 

### Why

Before I get into the coding challenge part, a little bit about why I'm doing this. I have been working as a developer for a long time, but largely without a personal web site. Because I have been drawn to Gatsby recently as a tool for building sites, I figured this would be a good opportunity to finally make progress on a site while I learn more about Gatsby and its ecosystem. I am also looking forward to the challenge of writing these posts while learning, since teaching others (you the reader) helps an individual (me) to retain the learned information. And hopefully my writing will improve as I do this. It has been a while.

I should also note that this is not meant to be a tutorial! There are excellent [tutorials on the Gatsby site](https://www.gatsbyjs.org/tutorial/). My intention here is to document my process and to explain what's going on in each challenge.

### Getting Started

I started out by generating a site from the default Gatsby starter. Maybe this is cheating a little, but it seems like there's a bit of boilerplate to get started. Since the first challenge is to create a blog, I figured I could start with the default starter and build a blog from there. If we run `gatsby new` without the second argument we will get the default starter, so let's do that: `gatsby new webbywebsite`.

Now I have a working Gatsby site. Good! But I want a blog on the site. I'm not really sure what this site is going to turn into, but keeping in the spirit of a [minimum viable product](https://en.wikipedia.org/wiki/Minimum_viable_product), I will just list out my blog posts right on the index page. This seems good enough. But in order to display these posts I will of course need some content.

### Creating Content

One of the many nice things about Gatsby is the way it can pull in content from a variety of sources: a file system, a headless cms, even an Instagram feed. Our content can come from almost anywhere, as long as we have mechanisms to retreive it and parse it so that Gatsby can consume it. For this blog, we will source our content from markdown files that get checked into the website's code repository. That means we can write up a post simply by creating a markdown file. The file will consist of markdown, which gets rendered as html, and frontmatter, which is like metadata for the post. Once we have created the file for the post, it is available for Gatsby to read. How does that happen?

### Sourcing Content

Now that the content has been created, it is time for Gatsby to read it. We need to tell Gatsby where to look for the content. In this case, the content exists on our file system, as opposed to a local database or a remote service. In order to read from the file system, we will use the [gatsby-source-filesystem](https://www.gatsbyjs.org/packages/gatsby-source-filesystem/) plugin. We add it as a dev dependency (it was already added in the Gatsby default starter). Then we add it to the `plugins` array in `gatsby-config.js`. The entry looks like this:

```
{
  resolve: `gatsby-source-filesystem`,
  options: {
    path: `${__dirname}/content/blog`,
  },
},

```
This tells Gatsby to read files from `/content/blog`, which is where we store our blog post markdown files. But once it reads the files, how does it know what to do with them? How does it know to parse them as markdown files?

### Transforming Content

Just as Gatsby sources data with various __source plugins__, it parses that data with various transformer plugins. To illustrate, let's assume I am sourcing entirely from my file system. I will then use gatsby-source-filesystem to get that data. But those files might be json, xml, csv, etc. Each kind of file needs a different transformer to prepare that data for Gatsby's data layer. In our case, we are only using markdown files, so we will use the [gatsby-transformer-remark](https://www.gatsbyjs.org/packages/gatsby-transformer-remark/) plugin. Again, we add it as a dev dependency, and then add it to `gatsby-config.js`. Now, when Gatsby builds, it turns each markdown file into a graphql node, with properties like `html` and `frontmatter` that we can access in our queries.  

We can inspect and explore this data by using GraphiQL, which is a GraphQL ui that we can access in a browser after running `gatsby develop` and browsing http://localhost:8000/___graphql. If we look at the Explorer in GraphiQL, we see a few queries that gatsby-transformer-remark has added, such as `allMarkdownRemark`. Running this   query will show the html of our first post:

```
query MyQuery {
  allMarkdownRemark {
    edges {
      node {
        html
      }
    }
  }
}
```

This shows how the transformer plugins shape our raw data into nodes that we can then query from the client. But what if we want to further shape that data so that we have more properties available to us? For example, what if we want a slug for each post so that we can eventually navigate to it? 

### Managing the Data Layer

Gatsby's data is handled by GraphQL. When a Gatsby site builds, its data layer is also assembled, and we can manipulate it somewhat with the [Gatsby Node API](https://www.gatsbyjs.org/docs/node-apis). Simply be exporting any of these functions from `gatsby-node.js` in the root of our project, we can construct our data layer as we see fit.

So how would we add a slug to a blog post node? We can use the [onCreateNode](https://www.gatsbyjs.org/docs/node-apis/#onCreateNode) api for that. When a node is created, this function will be called, and we can add the slug to the node. We do this by using gatsby-source-filesystem's createFilePath function to construct the markdown file's path. That path gives us a string from which we can derive a unique substring for a slug, like `/hello-world/`. We use the [createNodeField](https://www.gatsbyjs.org/docs/actions/#createNodeField ) action to attach this slug field to the node, so that it is available in our queries on the node. 

Now that we have a slug on each node, we want pages represented by those slugs. I have a set the slug `/hello-world/` on the node because I want to be able to view that node, represented by a page in a browser as a blog post. We do thise by implementing the [createPages](https://www.gatsbyjs.org/docs/node-apis/#createPages) api, also in the `gatsby-node.js` file. We query for all the plog posts, and for each blog post, we create a page whose path is the slug. We can also create previous and next properties that can be used to navigate to the previous and next pages, if there are any. 

### Passing Data to Components

The last part I want to mention is fairly straightforward. In order to get the data from the data layer into the components, we again use GraphQL to get the data we need. On our index page that renders the list of posts, we want to query for that list of posts. We can do that by using allMarkdownRemark again, just as we did in GraphiQL, but this time in the index page itself with the `graphql` tag. You can [read the docs](https://www.gatsbyjs.org/docs/page-query/#how-does-the-graphql-tag-work) for more info on how that works, but we basically use this tag to get the data we need for the component it is written for. For our purposes, that's it in a nutshell, and completes the flow from content to rendered component.

### Last Thoughts

I hope that explains how the blog posts work on this site. This was the first challenge of [100 Days Of Gatsby](https://www.gatsbyjs.org/blog/100days/), and I thought it was a valuable experience to write this post. I certainly would not have retained some of the information around the data layer, which is brand new to me. On the other hand, rendering the components is fairly easy, as I work mostly with React in client side applications. 

Finally, I realize that this looks pretty much like a Gatsby starter site. That's intentional, as I want to change it incrementally and write a post for each major change. 

The source code for this site is available here: https://github.com/tsargent/webbywebsite.com. The first pull request shows the changes I made for this particular challenge: https://github.com/tsargent/webbywebsite.com/pull/1. There are a few unrelated changes there as well, as I wanted to remove some starter code that I don't particularly need at the moment.

Thanks for reading!
