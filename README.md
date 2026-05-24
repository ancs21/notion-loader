# notion-loader

[Astro Content Loader](https://docs.astro.build/en/reference/content-loader-reference/) for Notion as a CMS, with Astro 6 support.

Fork of [`astro-notion/notion-astro-loader`](https://github.com/astro-notion/notion-astro-loader) (MIT, v1.1.2) with patches for Astro 6's stricter Loader schema types and Zod v4.

## Install

```sh
npm install notion-loader
# or: bun add notion-loader
```

Peers: `astro >= 6.0.0`, `zod >= 4.0.0` (both ship with Astro 6).
The Notion SDK (`@notionhq/client`) is bundled as a regular dependency — no need to install it separately.

## Usage

`src/content.config.ts`:

```ts
import { defineCollection, z } from "astro:content";
import { notionLoader, notionPageSchema } from "notion-loader";
import * as transformedPropertySchema from "notion-loader/schemas/transformed-properties";

const posts = defineCollection({
  loader: notionLoader({
    auth: import.meta.env.NOTION_TOKEN,
    database_id: import.meta.env.NOTION_POSTS_DB_ID,
    imageSavePath: "assets/images/notion",
    filter: {
      property: "Status",
      select: { equals: "Published" },
    },
  }),
  schema: notionPageSchema({
    properties: z.object({
      Title:   transformedPropertySchema.title,
      Slug:    transformedPropertySchema.rich_text,
      Excerpt: transformedPropertySchema.rich_text.optional(),
      Tags:    transformedPropertySchema.multi_select.optional(),
    }),
  }),
});

export const collections = { posts };
```

Rendering an entry's body in an `.astro` file:

```astro
---
import { render } from "astro:content";
const { Content, headings } = await render(entry);
---
<Content />
```

## Exports

| Specifier | What it gives you |
|---|---|
| `notion-loader` | `notionLoader`, `notionPageSchema`, `richTextToPlainText`, `fileToUrl`, `fileToImageAsset`, `dateToDateObjects` |
| `notion-loader/schemas/transformed-properties` | Zod schemas that decode Notion properties to primitives (`string`, `string[]`, `Date`, …) |
| `notion-loader/schemas/raw-properties` | Zod schemas that keep Notion's raw property objects |
| `notion-loader/schemas/page` | The `notionPageSchema` factory |

## Differences from upstream `notion-astro-loader@1.1.2`

| Area | Change |
|---|---|
| `loader.ts` | `schema: (async () => …) as never` cast for Astro 6's tightened `Loader` schema type |
| `rehype/rehype-images.ts` | Sets `file.data.astro.localImagePaths` alongside `imagePaths` (Astro 6 renamed the field) |
| `render.ts` | `rehype-katex` removed by default — `notion-rehype-k` emits `<b>` without a `properties` object on bold text, which crashes the plugin. Pass it via `rehypePlugins` if you need math rendering. |

## Development

```sh
bun install
bun run build      # tsc -p tsconfig.build.json → dist/
```

## License

MIT. Originally Copyright (c) 2024 astro-notion. Modifications Copyright (c) 2025 ancs21.
