---
title: "Integrating the KICK Categories API"
date: 2025-05-28 08:42:00 -500
categories: [StreamYard, APIs, Destinations]
tags: [production, new core feature, my journey, docu]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/KICK/kick.png"
---

In an effort to increase the product's value and widen our customer base, StreamYard has recently implemented a bunch of destinations. As we saw last time, I was in charge of building a generalizable infrastructure for destinations via RTMP under the hood. However, others like KICK or Brightcove, do have API access, and thus need direct implementation. 

This was a cool task for a number of reasons; firstly, because it is a classic full-stack developer task. I got to both add the backend with the necessary authentication and request to KICK, expose the endpoint, add a hook to handle fetching, and add the UI. But also because I got to what "real developers" do - AI won't help with an API this recent, even in public, so I had to dig through the official KICK docu to implement it (and basing a lot of code off of what we already had for the rest of the KICK integration, shoutout to my amazing colleagues for that). Also because I IDed my related tickets [KICKCAT] lol.

![Desktop View](/assets/img/StreamYard/KICK/new-dist-kick.png){: .normal}


## Backend

Other than adding the schemas and necessary throw Boom errors, the bulk of the task was handling the actual authenticated requests. As you can see in the [KICK dev docu](https://docs.kick.com/), KICK works with authenticated API GET requests by bearer token authentication in the `Authorization` header. This is part of its OAuth 2.0 flow to authorize users. Then we need to make the request with the necessary parameters, in this case `q` for the search query, which is required. Here's how this might work:


```typescript
export type FetchCategoriesParams = {
	/**
	 * Search query to filter categories
	 */
	searchQuery: string;
	/**
	 * The access token for authentication
	 */
	token: string;
};

/**
 * Fetch categories, can be updated by passing a search query
 *
 * @see {@link https://docs.kick.com/apis/categories}
 *
 * @param payload - {@link FetchCategoriesParams}
 *
 * @returns Promise that resolves to categories information
 */
export const fetchCategories = (
	{ searchQuery, token }: FetchCategoriesParams,
) => {
	const additionalConfig = { maxAttempts: 3, timeout: 6000, ...extraConfig };
	return helpers
		.requestWithErrorTracking({
			methodName: 'getCategories',
			config: {
				uri: `${API_BASE_URL}/public/v1/categories`,
				method: HTTP_METHODS.GET,
				headers: {
					Accept: '*/*',
					Authorization: `Bearer ${token}`,
				},
				qs: {
					q: searchQuery,
				},
			},
		})
		.then(_.camelCaseKeys)
		.then(result => {
			return fetchCategoriesSchema.parse(result.data);
		})
		.catch(err => {
			throw err;
		});
};

export default fetchCategories;
```


For the search query, we added a minimum limit of 3 characters to avoid unnecessary requests or results being too long. All requests should be wrapped in an auth wrapper, which checks if the token is expired, and if it is it refreshes it and then does the reuest.

```typescript
interface KickApiError extends Error {
	statusCode?: number;
}

const kickCategoriesRoute = (server: ExternalServer) =>
	server.route({
		method: 'GET',
		path: '...', // our route
		options: {
			auth: ...,
			description: 'Fetch Kick categories',
			pre: [
				{
					method: fetchDestinationFromParams,
					assign: 'destination' as const,
				},
				{ method: ensureDestinationExists },
			],
			validate: {
				query: Joi.object({
					searchQuery: Joi.string().min(3).required(),
				}).required(),
			},
			response: {
				schema: fetchCategoriesSchema,
			},
		},
		handler: async req => {
			const { destination } = req.pre;

			if (destination.platform !== 'kick') {
				throw Boom.badRequest('Destination is not a Kick destination');
			}

			const kickClient = kick.kickClient(destination);

			try {
				const response = await kickClient.fetchCategories({
					searchQuery: req.query.searchQuery,
				});
				const categories = response.map(category => ({
					id: category.id,
					name: category.name,
					thumbnail: category.thumbnail,
				}));

				return categories;
			} catch (err: unknown) {
				if (err instanceof Error) {
					const kickError = err as KickApiError;
					throw Boom.boomify(kickError, {
						statusCode: kickError.statusCode || 500,
					});
				}
				throw Boom.badGateway('Failed to fetch Kick categories');
			}
		},
	});

module.exports = kickCategoriesRoute;
export default kickCategoriesRoute;
```

And of course, we need to make sure we update the routes for creating and updating destinations, so that we can change the category before, during and after the stream.


## Frontend

Then in the frontend, the hardest part was making sure users couldn't send API requests to the categories endpoint if their KICK token was expired. This can be done in 2 main ways. The first would be having a poller that, say, every 24 hours checks token validity and shows an alert if expired (even though we can refresh tokens for requests, a user only has access to their account for 30 days unless they manually re-authenticate), similar to a cron job. Or the second option, we can allow users to make requests, but on failed request with error code 401 (failed to authenticate), show the alert. The best approach may also be to implement both (if you have the resources to run such a cron job). We did receive a massive surge of 401 errors to our endpoint, so I later implemented handling for this.

Other than that, it was just adding the UI to allow users to choose the category!

![Desktop View](/assets/img/StreamYard/KICK/select-kick-cat.png){: .normal}

<iframe class="embed-video" loading="lazy" src="/assets/img/StreamYard/KICK/kick-cat-fetching.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>


## Conclusion

This was a super fun task because I learned how to dig deep into official docu to integrate an API, and a full functionality from start to finish. I am a happy kiddo.