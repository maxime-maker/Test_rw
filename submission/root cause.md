# Root Causes

Please copy-paste the final answer that you obtained from the AI for each question. The chat interface has a copy button that you can use to copy each message in Markdown format. Please do NOT include images or screenshots.

## Problem 1

**Problem**: The tags are broken up into individual characters on the post view page.

**Question**: What is the underlying issue that causes this problem to occur and from which component (file) of this project does this issue originate?

**Answer**: *### Problem: The tags are broken up into individual characters on the post view page.

#### Underlying Issue:
The underlying issue is that the tags are not correctly parsed and stored as an array of strings. Instead, they might be stored as a single string, causing the frontend to display each character of the string as a separate tag.

#### Component/File:
The issue originates from the `ArticleEditComponent` in the frontend and the `ArticleService` in the backend.

#### Frontend Component (`article-edit.component.ts`):

In the `ArticleEditComponent`, ensure that the tags are correctly parsed from the form input before being sent to the backend.

```typescript
// article-edit.component.ts
submit() {
  this.data$.subscribe(data => {
    const articleData = { ...data, tagList: data.tagList.split(',').map(tag => tag.trim()) };
    this.store.dispatch(articleEditActions.publishArticle({ article: articleData }));
  });
}
```

#### Backend Service (`article.service.ts`):

In the `ArticleService`, ensure that the tags are correctly processed and stored as an array of strings.

```typescript
// article.service.ts
async createArticle(dto: CreateArticleDto): Promise<Article> {
  const tags = dto.tagList.split(',').map(tag => tag.trim());
  const article = this.em.create(Article, { ...dto, tagList: tags });
  await this.em.persistAndFlush(article);

  for (const tag of tags) {
    let existingTag = await this.em.findOne(Tag, { name: tag });
    if (!existingTag) {
      existingTag = this.em.create(Tag, { name: tag });
      await this.em.persistAndFlush(existingTag);
    }
  }

  return article;
}
```

### Summary:
The issue of tags being broken up into individual characters on the post view page is caused by incorrect parsing and storage of tags. The problem originates from the `ArticleEditComponent` in the frontend and the `ArticleService` in the backend. Ensuring that tags are correctly parsed and stored as an array of strings in both the frontend and backend will resolve this issue.*


## Problem 2

**Problem**: New tags  are not shown on the home page under "Popular Tags", even after a page refresh.

**Question**: What is the underlying issue that causes this problem to occur and from which component (file) of this project does this issue originate?

**Answer**: *### Problem: New tags are not shown on the home page under "Popular Tags", even after a page refresh.

#### Underlying Issue:
The underlying issue is that the new tags are not being correctly added to the database or the popular tags endpoint is not including the new tags. This could be due to the backend service that handles tag creation not properly updating the tags table, or the popular tags endpoint not fetching the latest tags from the database.

#### Component/File:
The issue likely originates from the `ArticleService` in the backend, which handles the creation of articles and tags, and the `TagController` or similar controller handling the popular tags endpoint.

#### Backend Service (`article.service.ts`):

Ensure that the new tags are correctly added to the database when an article is created.

```typescript
// article.service.ts
async createArticle(dto: CreateArticleDto): Promise<Article> {
  const tags = dto.tagList.split(',').map(tag => tag.trim());
  const article = this.em.create(Article, { ...dto, tagList: tags });
  await this.em.persistAndFlush(article);

  for (const tag of tags) {
    let existingTag = await this.em.findOne(Tag, { name: tag });
    if (!existingTag) {
      existingTag = this.em.create(Tag, { name: tag });
      await this.em.persistAndFlush(existingTag);
    }
  }

  return article;
}
```

#### Backend Controller (`tag.controller.ts`):

Ensure that the popular tags endpoint returns all tags, including the newly added ones.

```typescript
// tag.controller.ts
import { Controller, Get } from '@nestjs/common';
import { EntityManager } from '@mikro-orm/core';
import { Tag } from './entities/tag.entity';

@Controller('tags')
export class TagController {
  constructor(private readonly em: EntityManager) {}

  @Get()
  async findAllTags(): Promise<string[]> {
    const tags = await this.em.find(Tag, {});
    return tags.map(tag => tag.name);
  }
}
```

### Summary:
The issue of new tags not showing on the home page under "Popular Tags" is caused by the backend not correctly adding new tags to the database or the popular tags endpoint not fetching the latest tags. The problem originates from the `ArticleService` in the backend for tag creation and the `TagController` for fetching popular tags. Ensuring that new tags are correctly added to the database and that the popular tags endpoint fetches the latest tags will resolve this issue.*
