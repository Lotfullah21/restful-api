# REST API Naming Conventions

Naming conventions are the rules and standards used to name your API endpoints (URLs). Following consistent conventions makes your API predictable and easy to use for anyone consuming it — including your future self. A developer should be able to look at a URL and immediately understand what resource it refers to and what action it performs.

## 1. Use Nouns for Resources, Not Verbs

A REST API exposes resources (things), not actions. The HTTP method (GET, POST, PUT, DELETE) already describes the action, so the URL should only describe the resource.

**Wrong (verb in URL):**

```
GET /getCourses
POST /createCourse
DELETE /deleteCourse/1
```

**Correct (noun in URL):**

```
GET /courses
POST /courses
DELETE /courses/1
```

Why `/courses/1` and not `/course/1`?

- `/courses` represents the collection resource.
- `/courses/1` represents one item inside that same collection.
- Keeping the collection name consistent (`/courses` for both list and single-item routes) makes the API predictable.

The rule is simple: the URL is a noun (what), and the HTTP method is the verb (how).

## 2. Use Plural Nouns for Collections

When an endpoint returns or manages a list of items, the resource name should be plural.

```
GET /courses       -> returns a list of all courses
GET /courses/1     -> returns a single course with ID 1
POST /courses      -> creates a new course
DELETE /courses/1  -> deletes course with ID 1
```

Using plural consistently avoids confusion. You do not need to switch between `/course` (singular) and `/courses` (plural) — always use plural for the collection endpoint.

## 3. Use Lowercase Letters

URLs should always be lowercase. Uppercase letters in URLs can cause confusion because some servers treat them as different paths.

**Wrong:**

```
GET /Courses
GET /CourseDetails/1
```

**Correct:**

```
GET /courses
GET /course-details/1
```

## 4. Use Hyphens to Separate Words, Not Underscores or camelCase

When a resource name has multiple words, use hyphens (`-`) to separate them. Hyphens are more readable in URLs and are preferred by search engines and humans alike.

**Wrong:**

```
GET /course_details
GET /courseDetails
GET /CourseDetails
```

**Correct:**

```
GET /course-details
```

Note: Underscores (`_`) can be hidden by some link underlines in browsers, making them hard to read. Hyphens are always visible.

## 5. Use Nested URLs for Relationships

When a resource belongs to another resource, express that relationship by nesting the URL. This makes the hierarchy between resources clear.

**Example:** A course has many lessons. To get all lessons of a specific course:

```
GET /courses/1/lessons          -> all lessons of course 1
GET /courses/1/lessons/5        -> lesson 5 of course 1
POST /courses/1/lessons         -> create a new lesson under course 1
DELETE /courses/1/lessons/5     -> delete lesson 5 from course 1
```

Avoid going more than 2-3 levels deep in nesting. If nesting gets too deep, it becomes hard to read and maintain.

## 6. Do Not Use File Extensions in URLs

URLs should not include file extensions like `.json` or `.xml`. Instead, the client specifies the desired format using the `Accept` header.

**Wrong:**

```
GET /courses.json
GET /courses.xml
```

**Correct:**

```
GET /courses
```

With the header: `Accept: application/json`

## 7. Use Query Strings for Filtering, Sorting, and Pagination

These are not separate resources — they are ways to query the same resource. They belong in the query string (after the `?`), not in the URL path.

```
GET /courses?category=programming         -> filter by category
GET /courses?sort=created_at              -> sort by creation date
GET /courses?page=2&limit=10              -> pagination: page 2, 10 items per page
GET /courses?category=programming&page=1  -> combine filters
```

## 8. Use Consistent API Versioning in the URL

When you need to release a new version of your API that is not backwards-compatible (i.e., it changes existing behavior), include the version number in the URL. This lets old clients continue using v1 while new clients use v2.

```
GET /api/v1/courses     -> version 1 of the courses endpoint
GET /api/v2/courses     -> version 2 with new or changed behavior
```

Place the version prefix early in the path, right after `/api/`.

## Summary Table

| Rule                       | Wrong                               | Correct                         |
| -------------------------- | ----------------------------------- | ------------------------------- |
| Use nouns, not verbs       | `/getCourses`                       | `/courses`                      |
| Use plural for collections | `/course`                           | `/courses`                      |
| Use lowercase              | `/Courses`                          | `/courses`                      |
| Use hyphens between words  | `/course_details`, `/courseDetails` | `/course-details`               |
| Nest related resources     | `/lessons?courseId=1`               | `/courses/1/lessons`            |
| No file extensions         | `/courses.json`                     | `/courses`                      |
| Filters in query string    | `/courses/programming`              | `/courses?category=programming` |
| Version the API            | `/courses-new`                      | `/api/v2/courses`               |
