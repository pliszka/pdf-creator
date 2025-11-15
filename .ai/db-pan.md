# MySQL Database Schema - PDF Books Generator

This document outlines the complete database schema for the project, designed for MySQL. The structure is organized by Bounded Contexts to align with the application architecture.

## 1. List of Tables
The schema is divided into logical sections corresponding to the application's Bounded Contexts.

---
### Identity & Access Management Bounded Context

#### `users`
Stores information about all registered users.
- `id` (uuid, Primary Key) - Unique identifier for the user.
- `email` (varchar(255), Not Null, Unique) - User's email address, used for login.
- `password` (varchar(255), Not Null) - Hashed password for the user.
- `created_at` (timestamptz, Not Null, Default: `now()`) - Timestamp of user creation.
- `updated_at` (timestamptz, Not Null, Default: `now()`) - Timestamp of last update.
- `deleted_at` (timestamptz, Null) - Timestamp for soft-deleting the user.

#### `roles`
Defines the available user roles in the system.
- `id` (uuid, Primary Key) - Unique identifier for the role.
- `name` (varchar(50), Not Null, Unique) - The name of the role (e.g., 'Author', 'Buyer', 'Superadmin').

#### `user_roles`
A pivot table linking users to their assigned roles.
- `user_id` (uuid, Foreign Key to `users.id`, Primary Key) - The ID of the user.
- `role_id` (uuid, Foreign Key to `roles.id`, Primary Key) - The ID of the role.

---
### Content Bounded Context

#### `books`
Stores the core information and draft content for each book.
- `id` (uuid, Primary Key) - Unique identifier for the book.
- `author_id` (uuid, Not Null, Foreign Key to `users.id`) - The user who owns and authors the book.
- `title` (varchar(255), Not Null) - The working title of the book.
- `content` (jsonb, Null) - The modular content of the book, structured as a JSON array of blocks. Each block must have a unique UUID `id`.
- `status` (enum(`'draft'`, `'published'`), Not Null, Default: `'draft'`) - The current publication status of the book.
- `version` (integer, Not Null, Default: 1) - The current draft version number.
- `price` (numeric(10, 2), Null) - The price of the book.
- `currency` (char(3), Null) - The ISO 4217 currency code (e.g., 'PLN', 'EUR').
- `created_at` (timestamptz, Not Null, Default: `now()`) - Timestamp of book creation.
- `updated_at` (timestamptz, Not Null, Default: `now()`) - Timestamp of last update.
- `deleted_at` (timestamptz, Null) - Timestamp for soft-deleting the book.

#### `book_publications`
Stores immutable snapshots of a book's content each time it is published.
- `id` (uuid, Primary Key) - Unique identifier for the publication record.
- `book_id` (uuid, Not Null, Foreign Key to `books.id`) - The book that was published.
- `version` (integer, Not Null) - The version number that was published.
- `content_snapshot` (jsonb, Not Null) - A complete, immutable copy of the book's `content` at the time of publication.
- `metadata_snapshot` (jsonb, Not Null) - A snapshot of key book metadata (e.g., title, price) at the time of publication.
- `created_at` (timestamptz, Not Null, Default: `now()`) - Timestamp of when the publication occurred.
- *Constraint*: A unique constraint on `(book_id, version)` ensures a version can only be published once.

---
### Reviews Bounded Context

#### `review_codes`
Stores the unique codes generated for reviewers.
- `id` (uuid, Primary Key) - Unique identifier for the review session.
- `book_id` (uuid, Not Null, Foreign Key to `books.id`) - The book this code grants access to.
- `code` (varchar(255), Not Null, Unique) - The unique, shareable access code.
- `expires_at` (timestamptz, Not Null) - The timestamp when this code becomes invalid (7 days from creation).
- `created_at` (timestamptz, Not Null, Default: `now()`) - Timestamp of code creation.

#### `review_comments`
Stores comments left by reviewers, linked to a specific review session and a commentable entity.
- `id` (uuid, Primary Key) - Unique identifier for the comment.
- `review_code_id` (uuid, Not Null, Foreign Key to `review_codes.id`) - The review session this comment belongs to.
- `commentable_id` (uuid, Not Null) - The ID of the entity being commented on (e.g., a book's ID or a content block's ID).
- `commentable_type` (varchar(255), Not Null) - The type of entity being commented on (e.g., `'book'`, `'book_content_block'`).
- `comment` (text, Not Null) - The content of the comment.
- `status` (enum(`'new'`, `'in_progress'`, `'done'`, `'rejected'`), Not Null, Default: `'new'`) - The author-managed status of the feedback.
- `created_at` (timestamptz, Not Null, Default: `now()`) - Timestamp of comment creation.
- `updated_at` (timestamptz, Not Null, Default: `now()`) - Timestamp of last update.
- `deleted_at` (timestamptz, Null) - Timestamp for soft-deleting the comment.

---
### Sales & Library Bounded Context

#### `orders`
Stores information about (mocked) purchase transactions.
- `id` (uuid, Primary Key) - Unique identifier for the order.
- `user_id` (uuid, Not Null, Foreign Key to `users.id`) - The user who placed the order.
- `book_id` (uuid, Not Null, Foreign Key to `books.id`) - The book being purchased.
- `amount` (numeric(10, 2), Not Null) - The amount paid.
- `currency` (char(3), Not Null) - The currency of the payment.
- `status` (enum(`'pending'`, `'paid'`, `'failed'`), Not Null, Default: `'pending'`) - The status of the transaction.
- `transaction_id` (varchar(255), Null) - An ID from the (mocked) payment provider.
- `created_at` (timestamptz, Not Null, Default: `now()`) - Timestamp of order creation.
- `updated_at` (timestamptz, Not Null, Default: `now()`) - Timestamp of last update.

#### `library_items`
A pivot table representing a user's collection of purchased books.
- `user_id` (uuid, Foreign Key to `users.id`, Primary Key) - The ID of the user (buyer).
- `book_id` (uuid, Foreign Key to `books.id`, Primary Key) - The ID of the purchased book.
- `created_at` (timestamptz, Not Null, Default: `now()`) - Timestamp when the item was added to the library.

#### `public_opinions`
Stores public-facing ratings and reviews from buyers.
- `id` (uuid, Primary Key) - Unique identifier for the opinion.
- `book_id` (uuid, Not Null, Foreign Key to `books.id`) - The book being reviewed.
- `user_id` (uuid, Not Null, Foreign Key to `users.id`) - The user who wrote the review.
- `rating` (smallint, Not Null) - The star rating from 1 to 5.
- `comment` (text, Null) - The text content of the review.
- `created_at` (timestamptz, Not Null, Default: `now()`) - Timestamp of opinion creation.
- `updated_at` (timestamptz, Not Null, Default: `now()`) - Timestamp of last update.
- `deleted_at` (timestamptz, Null) - Timestamp for soft-deleting the opinion.
- *Constraint*: A `CHECK` constraint on `rating` to ensure it is between 1 and 5.
- *Constraint*: A unique constraint on `(user_id, book_id)` to prevent a user from reviewing the same book multiple times.

---
### Support Bounded Context

#### `support_tickets`
Stores user-submitted support requests.
- `id` (uuid, Primary Key) - Unique identifier for the ticket, to be shown to the user.
- `user_id` (uuid, Null, Foreign Key to `users.id`) - The user who submitted the ticket (nullable for unauthenticated users).
- `subject` (varchar(255), Not Null) - The subject of the support ticket.
- `message` (text, Not Null) - The detailed message from the user.
- `status` (enum(`'new'`, `'open'`, `'closed'`), Not Null, Default: `'new'`) - The current status of the ticket.
- `created_at` (timestamptz, Not Null, Default: `now()`) - Timestamp of ticket creation.
- `updated_at` (timestamptz, Not Null, Default: `now()`) - Timestamp of last update.

---
## 2. Relationships
- **`users` to `roles`**: Many-to-Many, through `user_roles`.
- **`users` to `books`**: One-to-Many (an author has many books).
- **`users` to `orders`**: One-to-Many (a user can place many orders).
- **`users` to `public_opinions`**: One-to-Many (a user can write many opinions).
- **`users` to `library_items`**: One-to-Many (a user has many items in their library).
- **`users` to `support_tickets`**: One-to-Many (a user can submit many tickets).
- **`books` to `book_publications`**: One-to-Many (a book can have many published versions).
- **`books` to `review_codes`**: One-to-Many (a book can have many review codes).
- **`books` to `orders`**: One-to-Many (a book can be in many orders).
- **`books` to `public_opinions`**: One-to-Many (a book can have many opinions).
- **`books` to `library_items`**: One-to-Many (a book can be in many libraries).
- **`review_codes` to `review_comments`**: One-to-Many (a single review session can have multiple comments).

## 3. Indexes
- **`users`**: Index on `email`.
- **`books`**: Index on `author_id` and `status`.
- **`book_publications`**: Index on `book_id`.
- **`review_codes`**: Index on `book_id`.
- **`review_comments`**: Composite index on `(commentable_id, commentable_type)`. Index on `review_code_id`.
- **`orders`**: Index on `user_id` and `book_id`.
- **`library_items`**: The composite primary key on `(user_id, book_id)` serves as an efficient index.
- **`public_opinions`**: Index on `book_id`. The unique constraint on `(user_id, book_id)` also creates an index.
- **`support_tickets`**: Index on `user_id` and `status`.

## 4. MySQL Policies (Row-Level Security)
To enforce data access rules at the database level, the following RLS policies should be implemented. This requires `ROW LEVEL SECURITY` to be enabled on the respective tables.

- **Policy on `books`:**
    - **Name:** `authors_can_access_own_books`
    - **Applies to:** `SELECT`, `UPDATE`, `DELETE`
    - **Condition:** `author_id = current_setting('app.current_user_id')::uuid`
    - **Description:** Allows an authenticated user to see and manage only the books where they are the author.

- **Policy on `review_comments`:**
    - **Name:** `authors_can_see_comments_on_their_books`
    - **Applies to:** `SELECT`, `UPDATE`
    - **Condition:** The comment belongs to a `review_code` that is linked to a book owned by the current user.
      ```sql
      EXISTS (
          SELECT 1 FROM review_codes rc
          JOIN books b ON rc.book_id = b.id
          WHERE rc.id = review_comments.review_code_id
          AND b.author_id = current_setting('app.current_user_id')::uuid
      )
      ```

- **Policy on `library_items`:**
    - **Name:** `users_can_see_own_library`
    - **Applies to:** `SELECT`
    - **Condition:** `user_id = current_setting('app.current_user_id')::uuid`
    - **Description:** Ensures a user can only see their own library items.

## 5. Additional Notes
- **UUIDs as Primary Keys:** All `id` columns are of type `uuid`. The application layer will be responsible for generating these UUIDs before insertion.
- **JSON:** The schema uses `json` for content storage due to its superior indexing and query performance in MySQL compared to the `json` type.
- **Soft Deletes:** The `deleted_at` timestamp allows for soft-deleting records. Application-level logic (e.g., Laravel's Global Scopes) should be used to automatically exclude soft-deleted records from regular queries.
- **Session Variable for RLS:** The RLS policies rely on a session-level variable (`app.current_user_id`) being set by the application backend upon user authentication. This is a common and secure pattern for passing user context to the database.
- **Enums:** Custom `enum` types should be created in MySQL for `book_status`, `review_comment_status`, `order_status`, and `ticket_status` to enforce value constraints.