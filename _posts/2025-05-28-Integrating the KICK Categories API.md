---
title: "Integrating the KICK Categories API"
date: 2025-05-28 08:42:00 -500
categories: [StreamYard, APIs, Destinations]
tags: [production, new core feature, my journey, docu]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/StreamYard/KICK/kick.png"
---

In an effort to increase the product's value and widen our customer base, StreamYard has recently implemented a bunch of destinations. As we saw last time, I was in charge of building a generalizable infrastructure for destinations via RTMP under the hood. However, others like KICK or Brightcove, do have API access, and thus need direct implementation. 

This was a cool task for a number of reasons; firstly, because it is a classic full-stack developer task. I got to both add the backend with the necessary authentication and request to KICK, expose the endpoint, add a hook to handle fetching, and add the UI. But also because I got do to what "real developers" do - AI won't help with an API this recent, even in public, so I had to dig through the official KICK docu to implement it (and basing a lot of code off of what we already had for the rest of the KICK integration, shoutout to my amazing colleagues for that). Also because I IDed my related tickets [KICKCAT] lol.

![Desktop View](/assets/img/StreamYard/KICK/new-dist-kick.png){: .normal}


## Backend

Other than adding the schemas and necessary throw Boom errors, the bulk of the task was handling the actual authenticated requests. As you can see in the [KICK dev docu](https://docs.kick.com/), KICK works with authenticated API GET requests by bearer token authentication in the `Authorization` header. This is part of its OAuth 2.0 flow to authorize users. Then we need to make the request with the necessary parameters, in this case `q` for the search query, which is required. Here's how this might work:


```typescript
export type FetchCategoriesParams = {
  searchQuery: string;
  token: string;
};

export const fetchCategories = async ({ searchQuery, token }: FetchCategoriesParams) => {
  const response = await fetch(`${API_BASE_URL}/categories?q=${searchQuery}`, {
    method: 'GET',
    headers: {
      'Accept': '*/*',
      'Authorization': `Bearer ${token}`,
    },
  });

  if (!response.ok) {
    throw new Error(`Failed to fetch categories: ${response.status}`);
  }

  const result = await response.json();
  return categoriesSchema.parse(result.data);
};
```


For the search query, we added a minimum limit of 3 characters to avoid unnecessary requests or results being too long. All requests should be wrapped in an auth wrapper, which checks if the token is expired, and if it is it refreshes it and then does the request.

```typescript
interface PlatformApiError extends Error {
  statusCode?: number;
}

const categoriesRoute = (server: Server) =>
  server.route({
    method: 'GET',
    path: '/api/destinations/{id}/categories',
    options: {
      auth: 'session',
      description: 'Fetch categories for a destination',
      pre: [
        { method: fetchDestinationFromParams, assign: 'destination' },
        { method: ensureDestinationExists },
      ],
      validate: {
        query: Joi.object({
          searchQuery: Joi.string().min(3).required(),
        }).required(),
      },
      response: {
        schema: categoriesResponseSchema,
      },
    },
    handler: async (req) => {
      const { destination } = req.pre;

      if (destination.platform !== 'target-platform') {
        throw Boom.badRequest('Destination platform mismatch');
      }

      const client = createPlatformClient(destination);

      try {
        const response = await client.fetchCategories({
          searchQuery: req.query.searchQuery,
        });

        return response.map(category => ({
          id: category.id,
          name: category.name,
          thumbnail: category.thumbnail,
        }));
      } catch (err: unknown) {
        if (err instanceof Error) {
          const apiError = err as PlatformApiError;
          throw Boom.boomify(apiError, {
            statusCode: apiError.statusCode || 500,
          });
        }
        throw Boom.badGateway('Failed to fetch categories');
      }
    },
  });

export default categoriesRoute;
```

And of course, we need to make sure we update the routes for creating and updating destinations, so that we can change the category before, during and after the stream.


## Frontend

Then in the frontend, the hardest part was making sure users couldn't send API requests to the categories endpoint if their KICK token was expired. This can be done in 2 main ways. The first would be having a poller that, say, every 24 hours checks token validity and shows an alert if expired (even though we can refresh tokens for requests, a user only has access to their account for 30 days unless they manually re-authenticate), similar to a cron job. Or the second option, we can allow users to make requests, but on failed request with error code 401 (failed to authenticate), show the alert. The best approach may also be to implement both (if you have the resources to run such a cron job). We did receive a massive surge of 401 errors to our endpoint, so I later implemented handling for this.

To make calls to our endpoint and fetch the categories, I created a custom hook, something like:

```typescript
export const useFetchCategories = (id: string) => {
  const [categories, setCategories] = useState<Category[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const fetchCategories = useCallback(
    async (params: { searchQuery: string }) => {
      try {
        setIsLoading(true);
        setError(null);

        const response = await api.getCategories(id, params.searchQuery);
        const result = await response.json();

        setCategories(result);
      } catch (err) {
        setError('Failed to fetch categories');
      } finally {
        setIsLoading(false);
      }
    },
    [id],
  );

  return {
    categories,
    isLoading,
    error,
    refetch: fetchCategories,
  };
};
```

Another issue I faced was that for other destinations with categories, like Twitch, we handle categories in terms of the id, as they are fixed and hard-coded (because they never change). Meanwhile, Kick has hundreds of categories that are continuously updated, so it is important we verify past selections of id (the only data we store) matches the intended name. Since the time constraint of the deadline made it impossible for me to update the full infrastructure to handle both the ids and names, I added a bit of a workaround - setting the id and name in the local storage. This is only to populate the `SelectCategory` once we try to edit a stream that already had a selected category. This ensures consistency even if the categories change and the id no longer matches your category name.

```typescript
useEffect(() => {
  if (props.value !== '') {
    // Check localStorage for previously selected category
    const cachedCategories = JSON.parse(
      localStorage.getItem('cached-categories') || '{}'
    );
    const existingSelection = cachedCategories[id];

    if (existingSelection && String(existingSelection.id) === String(props.value)) {
      setSelectedCategory({
        label: existingSelection.name,
        value: existingSelection.id,
      });
      return;
    }
  }
  setSelectedCategory(null);
}, [props.value, categories, id]);
```


Other than that, it was just adding the UI to allow users to choose the category!

![Desktop View](/assets/img/StreamYard/KICK/select-kick-cat.png){: .normal}

<iframe class="embed-video" loading="lazy" src="/assets/img/StreamYard/KICK/kick-cat-fetching.mp4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>


## Conclusion

This was a super fun task because I learned how to dig deep into official docu to integrate an API, and a full functionality from start to finish. I am a happy kiddo.