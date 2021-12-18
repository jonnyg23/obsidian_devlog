---
tags: 📥/📓/🟧
aliases:
  - 
cssclass:
type: learning-journal
status: 🟧
---

## Metadata
- `Title:` [[! 2021-12-18 Web Hosting & GraphQL]]
- `Type:` [[!]]
- `Tags:`
- `Formation Date:` [[2021-12-18]]

---

## [[Web Hosting]] Providers & [[Full Stack Deployment]]

>Note: I was researching to find out what areas of my full stack website would cost me money & what web hosting I would need for a [[Postgres]] database server. This is what I found in my 2 hours of digging…

### Front End Hosting

#### Netlify ✅

- [[Netlify]] seemed to be the winner in this category as it is so quick to deploy a React app.
	- It even gives you the ability to have a CI/CD pipeline with Github!


### Back End & Database Hosting

#### Reliable Cheapest Options

##### Hostinger ✅

> The perk about Hostinger is that SSL, Domain is free with purchase. Also, you get unlimited bandwidth & unlimited databases in the cloud hosting plan.

- [**VPS Hosting**](https://www.hostinger.com/vps-hosting)
- [**Cloud Hosting**](https://www.hostinger.com/cloud-hosting)


##### Cloudways ✅

> The perk for Cloudways is that you can pick your "Pay-as-You-Go" Plan from different Data Centers such as [[DigitalOcean]], [[Linode]], [[Vultr]], [[AWS]], & [[Google Cloud]].



#### VPS Hosting

> Use VPS (Virtual Private Server) for small project that you don't believe you will need scalability very much. The price is normally lower than Cloud server hosting.

##### Vultr

##### OVHCloud


#### Cloud Hosting

> Use Cloud Hosting if you need the ability to scale very quickly.

> **Warning**: Cloud Hosting, if performed incorrectly, could rack up a quick bill very fast if there is any endless recursive calls in the web app!

##### AWS RDS & ECS

- AWS (Amazon Web Services)
- [RDS (Relational Database Service)](https://aws.amazon.com/rds/?nc2=h_ql_prod_fs_rds) - Set up, operate, and scale a relational database in the cloud with just a few clicks
	- PostgreSQL is included as a database engine.
- [ECS (Elastic Container Service)](https://aws.amazon.com/ecs/) - Run highly secure, reliable, and scalable containers.

##### Heroku

- This seems to be the more costly option, possibly a little less than AWS services.

##### Google Cloud


### User/Site Image Cloud Storage & Cache+Resize Images

#### Cache + Resizing Images

##### IMGIX
- This website [IMGIX](https://imgix.com/pricing) doesn't give you space to store your image files, but it can really help with image caching & resizing images in real time to whatever dimension you need.
	- IMGIX should be seen as a pre-processing layer before the image is sent to the cloud space below.
	- **Free Plan has Infinite Transformations!**

#### Image Cloud Storage

##### Google Cloud Storage

- **Research this, but I believe you will be able to send your images to the cloud storage & you can query this image with an image ID appended to the cloud URL.**


## GraphQL

> [What is GraphQL? Use Cases, Applications & Databases](https://fauna.com/blog/what-is-graphql-use-cases-applications-and-databases)

### [[GraphQL]] vs. [[REST API]]

|                        | **REST**                                                                                                                                     | **GraphQL**                                                                                                                                                                                                                                                                                                       |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Data Fetching**      | With REST you have to access multiple server resource endpoints, requiring multiple round trips between the client and the server.           | With GraphQL, data access is handled through a single endpoint that defines all the data requirements for the API call. This single-shot design is quite beneficial in mobile apps, where multiple round trips are frowned upon in low bandwidth scenarios.                                                       |
| **Web Caching**        | Caching is part of the HTTP specification which REST APIs can benefit from.                                                                  | GraphQL does not follow the HTTP specification for caching. Instead, it uses a single endpoint, so it is up to the developer to ensure that caching is appropriately implemented for non-mutable queries. <br> <br>Several third-party GraphQL libraries such as FlacheQL, and Relay exist to solve this problem. |
| **File Uploading**     | REST supports file uploading to different computers over the internet                                                                        | Does not support file uploading                                                                                                                                                                                                                                                                                   |
| **Technology Ramp Up** | If you’re familiar with the HTTP and client-server architecture, you can easily understand the REST API and start using it in your projects. | The GraphQL SDL has a bit of a learning curve. If your engineering team hasn't worked with GraphQL before, they should plan to spend time and resources learning it and ramping up.                                                                                                                               | 


### When to use GraphQL?

- **Reducing app bandwidth usage** - Usually mobile applications deal with slow networks and reduced bandwidth limitations all the time. Optimize this with GraphQL, as multiple network requests can be added to a single query, thus reducing the total number of app requests.
- **Complex & Composite Systems** - Abstraction reduces complexity. Doesn't overwhelm the developer.
- **Microservice based architectures** - Effective request routing, parallel request execution, and graceful failure handling. Even as microservices grow in your environment, your app only points to a single endpoint, and GraphQL manages to route the requests to the appropriate microservices.
- **When you need rapid prototyping** - MVP (Minimum Viable Product) & quick to prototype new apps. GraphQL has features that can help accelerate development so that your front-end teams can get going without depending on your backend teams.

### Drawbacks of GraphQL

- **The "N+1" GraphQL problem** - Once you get over the basics of GraphQL, you'll encounter the well-known "N+1" problem. GraphQL executes a separate resolver function for every field in the query request, whereas there is one resolver per endpoint in REST. For example, for the below query, GraphQL will fetch the authors first, and then for each author, fetch the address.

```js
query
{
    authors {
        name
        address {
            country
        }
    }
}
```

This results in N+1 round trips to the database. New GraphQL libraries like [Dataloader](https://github.com/graphql/dataloader) are now available, which can batch consecutive requests and make a single data request under the hood.

- **Your app is too simple** - In cases where your app only leverages a few fields and that too in a standardized way, the added complexity of GraphQL would be overkill, and it may be more efficient to use a REST architecture.
- **Your use-case needs to support file uploads** - Sooner or later, you will realize that uploading files with GraphQL is a significant pain point. Unlike REST, file uploads are not part of the official GraphQL spec yet. However, GraphQL enthusiasts have built several third-party libraries that are geared towards solving this problem.
- **vCaching Issues** - Caching delivers snappier app experiences for users accessing the same resources over and over again. Because GraphQL is transport agnostic— not making use of the HTTP semantics makes caching a challenge.
- **The learning curve is steep** - GraphQL is a compelling query language for APIs with many benefits, but its higher complexity and steeper learning curve may not suit every application and team. With GraphQL, your engineering team may have to spend additional hours getting accustomed to the SDL before gaining their productive edge.





### Examples

- [Buidling a GraphQL server with [[FastAPI]]](https://blog.logrocket.com/building-a-graphql-server-with-fastapi/)




🔗 Links to this page:

