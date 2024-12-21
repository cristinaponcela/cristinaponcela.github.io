---
title: "My First Month at Bending Spoons as a Software Engineer"
date: 2024-12-16 08:42:00 -500
categories: [ Bending Spoons, Listing, Unit Testing, Typescript]
tags: [employment, RxJS, Redux, my journey, ESLint, Jest]
image: "https://raw.githubusercontent.com/cristinaponcela/cristinaponcela.github.io/refs/heads/main/assets/img/BendingSpoons.jpg"
---

Bending Spoons has been making a lot of noise lately, both for its profitable acquisitions and unusual nature. As of today, I have completed my first month working at this so-called Italian unicorn as a Software Engineer, and wanted to take a moment to break down how a company as big as Bending Spoons gets its engineers up to speed and ready to push code to products used by millions in weeks.

It has been _a ride_. Mainly because I wanted it to - they are super chill and flexible with hours, and as long as the work is done, you are allowed to work more or less when you want, and for as long or little as you need. But the main reason personally to apply in the first place was to learn from the best, as I learned how to code by myself and am aware I have gaps in my knowledge and a few coding bad habits. So I decided to take the most of it and *work*. I cannot stress this enough, the amount of learning I have done in the last 4 weeks far exceeds the experience I had before, and was faster than I thought plausible.

You start with around 6 weeks of training - they give you a guide to build a B2B product, in my case a website that sells courses. This is beautifully done, as it allows you to get at least some understanding of all the aspects that go into building and deploying a product. We believe in something widely used in the company, the term [T-shaped Engineers](https://medium.com/@mailtodevens/t-shaped-engineers-the-future-of-tech-talent-according-to-gartner-a2a3b92490da), which essentially means having a broad knowledge, even if shallow in some areas, and then being a specialist in your domain. This allows you to be more versatile and ask for less help - just because a problem is frontend and you are a backend developer, it doesn't mean you immediately have to reach out to someone else and wait until the issue is solved. It makes for more efficient work and more fun engineering.

As a first step, I learned about coding best practices. Boy oh boy did I have *no* idea. Things like commit best practices? Like, you're telling me I can't just `git commit -m "Luigi's YAHOOOO"` and call it a day? Also, I learned about [comment best practices](https://stackoverflow.blog/2021/12/23/best-practices-for-writing-code-comments/), which again I would have never really given a second thought otherwise. Then I also learned about when and _when not to_ throw errors, as they can obscure debugging. Finally, I explored Software Design best practices, a concept rarely spoken about. It is the idea that you should plan your code and project beforehand. Duh, right? But who here has done that ever, let's be honest.

Then, I was introduced to another wild concept to me: linting. For the uninitiated like me, linting your code means adding some config to throw an error if it is not properly formatted. You may know Prettier, right? Well, we use the same plugin but ensuring the code run fails if it is not ✨pretty✨. But no, really, this is very useful, especially when creating a PR, as it means the code review will be way easier for whatever poor soul you decided to assign.

Something like:

```javascript
import pluginJs from "@eslint/js";
import tseslint from "typescript-eslint";
import pluginReact from "eslint-plugin-react";
import pluginPrettier from "eslint-plugin-prettier";
import prettierConfig from "eslint-config-prettier";

/** @type {import('eslint').Linter.Config[]} */
export default [
  { files: ["*/.ts"] },
  {
    languageOptions: {
      parserOptions: {
        warnOnUnsupportedTypeScriptVersion: false,
      },
    },
  },
  pluginJs.configs.recommended,
  ...tseslint.configs.recommended,
  pluginReact.configs.flat.recommended,
  prettierConfig,
  {
    plugins: {
      prettier: pluginPrettier,
    },
    rules: {
      "prettier/prettier": "error",
      ...tseslint.configs.recommended.rules,
      "no-console": "off",
      "react/react-in-jsx-scope": "off",
    },
  },
  {
    settings: {
      react: {
        version: "detect",
      },
    },
  },
];
```

does the job. Note we used [ESLint](https://eslint.org/) with Typescript typing.

That's when the project truly started. I built a simple backend using [`@hapi/hapi`](https://hapi.dev/) and `@hapi/boom`. I used hapi to run the server locally, and then axios to create the instance. Then I just used GET requests to fetch the data. 

Running a simple local server is quite straightforward. You create an axios instance

```typescript
import axios from "axios";
import axiosRetry, { exponentialDelay } from "axios-retry";

const axiosInstance = axios.create({
  baseURL: ⁠ https://your-url⁠,
  timeout: 5000,
});

axiosRetry(axiosInstance, {
  retries: 3,
  retryDelay: exponentialDelay,
  retryCondition: (error) => {
    return (
      axiosRetry.isNetworkOrIdempotentRequestError(error) ||
      (axios.isAxiosError(error) && error.code === "ECONNABORTED")
    );
  },
});

export default axiosInstance;
```
then add a plugin to create the url extensions. For example, you may have https://example.com, and you want to add `/contact-us`. Then you can just set a path `"/contact-us"` and you will now be able to fetch data from this endpoint.

In my project, I was simulating a server by exposing a `/` endpoint, which would return a list of categories, courses and reviews for a course selling website.

```typescript
import { Server } from "@hapi/hapi";
import { fetchData } from "../request";
import Joi from "joi";
import { courseSchema, reviewSchema } from "./schemas";

export const allResponseSchema = Joi.object({
  courses: Joi.array().items(courseSchema).required(),
  reviews: Joi.array().items(reviewSchema).required(),
});

const allPlugin = {
  name: "allPlugin",
  version: "1.0.0",
  register: async (server: Server) => {
    server.route({
      method: "GET",
      path: "/",
      handler: async (request, h) => {
        const data = await fetchData();
        return h.response({
          data,
        });
      },
    });
  },
};

export default allPlugin;
```

for the whole data, and

```typescript
categoryPlugin.ts

import { Server } from "@hapi/hapi";
import { fetchData } from "../request";
import Joi from "joi";
import { Category, Course } from "../types";
import { categorySchema, courseSchema } from "./schemas";
import * as Boom from "@hapi/boom";

export const responseCategorySchema = Joi.object({
  categories: Joi.array().items(categorySchema).required(),
  courses: Joi.array().items(courseSchema).required(),
});

const categoryPlugin = {
  name: "categoryPlugin",
  version: "1.0.0",
  register: async (server: Server) => {
    server.route({
      method: "GET",
      path: "/category",
      options: {
        validate: {
          query: Joi.object({
            id: Joi.string().required(),
          }),
        },
        response: {
          schema: responseCategorySchema,
        },
      },
      handler: async (request, h) => {
        const { id } = request.query as Record<string, string>;

        const data = await fetchData();

        const categoryExists = data.categories.find(
          (cat: Category) => cat.id === id
        );
        if (!categoryExists) {
          return Boom.notFound(⁠ Category with id "${id}" not found ⁠);
        }

        const filteredCourses = data.courses.filter(
          (course: Course) => course.category_id === id
        );
        return h.response({
          categories: [categoryExists],
          courses: filteredCourses,
        });
      },
    });
  },
};

export default categoryPlugin;
```

to query by category (returns courses for that category).

> Super cool note, I used [Joi](https://joi.dev/) to validate the data; this means passing a scheme, which is the expected set of data types that we should receive, using typescript to make sure the data is typed. If the response is invalid, the server will throw an error and the data will not render on the UI.
{: .prompt-tip}

An example of a schema would be:

```typescript
import Joi from "joi";

export const categorySchema = Joi.object({
  id: Joi.string().required(),
  name: Joi.string().required(),
  color: Joi.string().required(),
});
```


Furthermore, as I went along, I grew familiar with unit testing. This one was mindblowing. We use [jest](https://jestjs.io/). The idea is writing tests, or functions where you run one of the components you created, you can mock the tested function - you make it expect a `mockValue`, which you give from the data and you are sure the component should output, instead of running the actual function (or server). The test runs, either passing or failing. If it fails, it is a pretty good indication that your product is not working as intended, and so you get to solve issues before the problems arise. High quality code B)

A server test: 

```typescript

import { Server } from "@hapi/hapi";
import "../src/axios";
import { init } from "../src/server";
import { fetchData } from "../src/request";

export const mockData = {
  categories: [
    {
      id: "programming",
      name: "Programming",
      color: "1f77b4",
    },
  ],
  courses: [
    {
      id: "python-intro",
      title: "Introduction to Python",
      description:
        "Learn the fundamentals of Python, one of the most versatile programming languages.",
      author_name: "John Doe",
      author_avatar_url: "https://picsum.photos/seed/john-doe/100",
      author_profession: "Software Engineer",
      author_biography:
        "John is a software engineer with over 10 years of experience in Python and data science.",
      category_id: "programming",
      image_url: "https://picsum.photos/seed/python-intro/300/200",
      last_updated: "2024-09-08T15:30:00Z",
    },
  ],
  reviews: [
    {
      id: "review-1",
      course_id: "python-intro",
      name: "Alice Johnson",
      rating: 5,
    },
  ],
};

jest.mock("../src/request", () => ({
  fetchData: jest.fn(),
}));

describe("Hapi Server Tests with Mocked Axios", () => {
  let server: Server;

  beforeAll(async () => {
    server = await init();
  });

  afterAll(async () => {
    await server.stop();
  });

  beforeEach(() => {
    (fetchData as jest.MockedFunction<typeof fetchData>).mockResolvedValue(
      mockData
    );
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  it("should return filtered data by category", async () => {
    const category = "programming";

    const response = await server.inject({
      method: "GET",
      url: ⁠ /category?id=${category} ⁠,
    });

    const result = JSON.parse(response.payload);
    expect(response.statusCode).toBe(200);
    expect(result.categories).toHaveLength(1);
    expect(result.categories[0].id).toBe(category);
    result.courses.forEach((course: { category_id: string }) => {
      expect(course.category_id).toBe(category);
    });
  });

  it("should return filtered data by course", async () => {
    const course = "python-intro";

    const response = await server.inject({
      method: "GET",
      url: ⁠ /course?id=${course} ⁠,
    });

    const result = JSON.parse(response.payload);

    expect(response.statusCode).toBe(200);
    expect(result.courses).toHaveLength(1);
    expect(result.courses[0].id).toBe(course);
  });

  it("should return filtered data by review", async () => {
    const review = "review-1";

    const response = await server.inject({
      method: "GET",
      url: ⁠ /review?id=${review} ⁠,
    });

    const result = JSON.parse(response.payload);

    expect(response.statusCode).toBe(200);
    expect(result).toHaveProperty("review");
    expect(result.review.course_id).toBe("python-intro");
  });
});
```

> Warning: most of the time, and especially for async fetching functions, you don't want to be running the actual function, as this may create time delays in the real product by adding requests to the server. These are called integration tests, and while also important, should not be abused.
{: .prompt-warning}

Hence, unit tests, as the name indicates, should only test units; mainly the logic or rendering of a component. For example, whether a filtering function filters correctly (given an input and an expected, known output), or rendering a `CoursePreview` and checking, using `fireEvent` from `jsdom library`, whether it correctly routes to the `CoursePage` upon being clicked. This library is extremely useful for frontend tests.

Finally for the first part of this recap, I learned about CI/CD - automating the build, testing and deployment of software. We use GitHub Actions for this. Basically, you can set it up to run multiple things when you push to a branch, like lint and tests. If the build fails, you get a notification and, depending on the project's setup, it will block PR merges. This ensures your code is kept healthy and only fully and correctly working products are pushed to main.

In your root directory, create something like `.github/.workflows/ci.yml`:

```yml
name: CI Pipeline

on:
  push:
    branches:
      - "*"
  pull_request:
    branches:
      - "*"

jobs:
  lint-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 23.2.0

      - name: Set up pnpm
        uses: pnpm/action-setup@v3
        with:
          version: 9

      - name: Install dependencies
        uses: actions/cache@v3
        with:
          path: ~/.pnpm-store
          key: ${{ runner.os }}-pnpm-${{ hashFiles('**/pnpm-lock.yaml') }}
          restore-keys: |
            ${{ runner.os }}-pnpm-

      - name: Install dependencies in CourseNinja
        working-directory: CourseNinja
        run: pnpm install

      - name: Install dependencies in Server
        working-directory: Server
        run: pnpm install

      - name: Run linter
        run: pnpm run lint:fix && pnpm run lint

      - name: Run tests
        run: pnpm run test
```

Then, as long as you have defined the required scripts in `package.json`, like:

```json
{
"scripts": {
    "start": "ts-node src/server.ts",
    "build": "tsc",
    "lint": "eslint '*/.{js,jsx,ts,tsx}' --cache .",
    "lint:fix": "eslint '*/.{js,jsx,ts,tsx}' --cache . --fix",
    "test": "jest"
  },
}
```

this will do the job!

And that does it for the backend. I know, pretty intense stuff. I'll be back soon with more on the frontend of the project. Spoiler alert: state management go brrr.