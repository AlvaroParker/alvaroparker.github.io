+++
title = "How to Self-Host a Next.js Application with Cloudflare"
date = "2025-05-10T18:14:38.000Z"

[taxonomies]
tags = ["DevOps", "Next.js", "Cloudflare", "Server"]
+++

# Self-Hosting a Next.js Application

A small team of devs and I had the task of developing a small e-learning platform that needed to include the following features:

- Create courses
- Create professors
- Pay for courses
- Add modules to a course
- Add videos and files (resources) to modules within a course
- Allow users to log in
- Provide an admin dashboard to manage the website

Using something like Moodle would have been overkill and didn’t include all the features the client wanted, so that was out of the question. Hence, we decided to develop our own platform using Next.js and Turborepo. We split the platform into three applications: `services` for the API endpoints, `admin` for the administrator dashboard, and `web` for the public view of the platform.

## The Stack

To host and stream our videos, we first considered using YouTube for simplicity. However, since we couldn’t protect the URLs from unauthorized access, we ruled it out. Instead, we chose Cloudflare Stream, which offers restricted video access using signed URLs. This allowed us to generate signed URLs valid for a limited time and prevent unauthorized downloads.

To store files and dynamic content like profile pictures, we chose to use S3-compatible object storage. We didn’t want to store files on the same server hosting the websites, especially since we weren’t yet sure whether we would self-host. To decouple file storage from the app, we opted for object storage. During development, we used Adobe’s S3 mock implementation in Docker, so we didn’t need to set up an AWS account yet.

For the database, we chose PostgreSQL with Prisma, which is one of the most widely used ORMs with Next.js.

## Deploying on a VPS

After development, we reached the final stage: deploying the website for production. We had a few deployment options but ended up self-hosting due to internal reasons and client requirements. My responsibility was to deploy the site and "make sure it was fast."

Considering the many blog posts explaining why self-hosting a Next.js application might not be ideal (see the [Open Next project](https://opennext.js.org/) and [ThePrimeagen’s interview with the lead dev](https://www.youtube.com/watch?v=E-w0R-leDMc)), I took on the challenge of making it work as well as possible—even though Next.js is not typically deployed in this fashion.

My first step was deploying the Turborepo project behind an NGINX reverse proxy. I created one Docker image to run all three apps. Each app exposes an internal port in a `docker-compose.yml` file, which is then accessed by a container running NGINX. The NGINX container exposes ports `80` and `443` to the internet:

```
                    +------------------------+
                    |     Public Internet    |
                    +-----------+------------+
                                |
                         (Public IP: 80/443)
                                |
                     +----------v-----------+
                     |     NGINX Proxy      |
                     |  (Docker container)  |
                     | Ports: 80, 443 (host)|
                     +----------+-----------+
                                |
       +------------------------+-------------------------+
       |                        |                         |
   / --> port 3000         /admin --> port 4000     /api --> port 5000
       |                        |                         |
+------v-----+          +-------v-----+           +-------v------+
|  Next.js   |          |  Next.js    |           |  Next.js     |
|   App: web |          | App: admin  |           | App: service |
+------|-----+          +------|------+           +-------|------+
       |                       |                          |
       +-----------------------+--------------------------+
                               |
                         Internal Access to:
                         +-------------------+
                         |   PostgreSQL DB   |
                         | (Docker port 5432)|
                         +-------------------+
                                  ^
                                  |
                    +-------------+-------------+
                    |   TurboRepo Docker Image   |
                    | (Runs all 3 Next.js apps)  |
                    +-------------+-------------+
                                  |
                    +-------------v-------------+
                    |     Docker Compose File    |
                    | (Defines 3 containers:     |
                    |  nginx, turbo, postgres)   |
                    +----------------------------+
```

This setup ran on the VPS with only ports `80` and `443` exposed to the internet. The PostgreSQL database was accessible only at the Docker network level. NGINX routed requests to `/`, `/admin/*`, or `/api/*` to the appropriate Next.js app.

With S3 deployed on AWS and API keys configured via `.env` files, the deployment was _ready_ for production. We just needed to configure the `A` record on Cloudflare DNS for domain access. But we could do better—by leveraging one of Cloudflare’s key features: caching.

## Caching on the Edge

Let’s first define edge caching: it involves storing frequently accessed content on servers closer to the end user, such as edge servers. For example, if users are in Brazil and the VPS is in Chile, edge caching allows requests to be served faster from nearby servers.

```

                            [ User Requests ]
                                  |
                                  v
                       +-----------------------+
                       |   Global Load Balancer|
                       +-----------------------+
                          /         |         \
                         /          |          \
                        v           v           v
                +------------+ +------------+ +------------+
                | Edge Cache | | Edge Cache | | Edge Cache |
                |  Server 1  | |  Server 2  | |  Server 3  |
                +------------+ +------------+ +------------+
                  |   |   |        |   |   |       |   |   |
                  |   |   |        |   |   |       |   |   |
      [img, css, js]  |   |   [img, css, js]  |   [img, css, js]
                      |   |                  |
                      |   v                  v
                   (MISS or Expired)     (MISS or Expired)
                      |                      |
                      v                      v
                    +-----------------------------+
                    |       VPS (Origin Server)   |
                    |   Runs website (HTML, API)  |
                    +-----------------------------+
```

We configured Cloudflare cache rules to target:

- `/_next/static/*`
- `/images/*`
- `/fonts/*`
- `/_next/image*`

Requests to these paths ignore the `Cache-Control` headers sent by Next.js and apply a one-year TTL on both the edge and the browser. This way, follow-up requests are faster—either from the browser cache or Cloudflare’s edge cache.

### Migrating to R2

Everything worked fine, and deployment was a success. But I overlooked an important issue: the S3 images for courses and profile pictures weren’t being cached, slowing down page loads. We could’ve solved this using Amazon CloudFront, but mixing providers didn’t feel ideal. Since the app was "production-ready" but not yet launched, we decided to migrate to Cloudflare R2.

R2 offers free-tier egress and lets us cache public assets using Workers.

Here's how it works: we had publicly available files named like `public-<uuid>`. We set up a Cloudflare Worker accessible via `<WORKER_URL>/public-<UUID>`, which serves R2 objects and caches them at the edge. The Worker looks like this:

```js
export default {
 async fetch(request, env) {
  const key = some_fn(); // some function to get the key

  if (!key.startsWith('public-')) {
   return new Response('Forbidden', { status: 403 });
  }

  ... // Some internal logic
  // Fetch from R2:
  const object = await env.MY_BUCKET.get(key);
  if (!object) {
   return new Response('Object Not Found', { status: 404 });
  }

  const headers = new Headers();
  object.writeHttpMetadata(headers);
  headers.set('etag', object.httpEtag);
  headers.set('Cache-Control', 'public, max-age=31536000');

  response = new Response(object.body, { headers });

  // Store in edge cache for future requests
  await cache.put(request, response.clone());
  return response;
 }
}
```

Now, when users visit the landing page and request profile pictures or course thumbnails, the assets are cached at the edge, reducing load times and improving responsiveness.

## The TLDR on our caching strategy

To optimize performance and reduce latency, we implemented multiple layers of caching using Cloudflare. Static assets such as JavaScript, CSS, fonts, and images served by the Next.js application were cached at the edge by configuring page rules for paths like `/_next/static/*`, `/images/*`, `/fonts/*`, and `/_next/image*`, with a one-year TTL applied both at the edge and in the browser. Additionally, we leveraged Cloudflare R2 for storing public files and used a Worker to serve these assets via a `/public-<UUID>` URL pattern. The Worker not only enforced access control but also applied long-term caching headers and stored the responses in the edge cache. Together, these strategies significantly improved load times and responsiveness across geographically distributed users.

## Final Notes

While self-hosting a Next.js project isn’t always the best approach, sometimes it’s necessary. If you go that route, make sure you take full advantage of your provider's caching capabilities to improve user experience. You shouldn’t just throw your Next.js app into a Docker container and call it a day—unless that’s all your case really needs.
