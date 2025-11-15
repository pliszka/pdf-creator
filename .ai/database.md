# PDF Books Generator - MySQL Database Schema
*   **Wersja:** 1.0
*   **Data:** 2025-11-15 12:20:37
*   **Autor:** @pliszkaPL

```sql
-- #####################################################################
-- # 1. Uwierzytelnianie i Role (Identity & Access Management Bounded Context)
-- #####################################################################

CREATE TABLE `users` (
    `id` BINARY(16) NOT NULL PRIMARY KEY,
    `email` VARCHAR(255) NOT NULL UNIQUE,
    `password` VARCHAR(255) NOT NULL,
    `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    `deleted_at` TIMESTAMP NULL DEFAULT NULL
);

CREATE TABLE `roles` (
    `id` BINARY(16) NOT NULL PRIMARY KEY,
    `name` VARCHAR(50) NOT NULL UNIQUE COMMENT 'e.g., Author, Buyer, Superadmin'
);

CREATE TABLE `user_roles` (
    `user_id` BINARY(16) NOT NULL,
    `role_id` BINARY(16) NOT NULL,
    PRIMARY KEY (`user_id`, `role_id`),
    FOREIGN KEY (`user_id`) REFERENCES `users`(`id`) ON DELETE CASCADE,
    FOREIGN KEY (`role_id`) REFERENCES `roles`(`id`) ON DELETE CASCADE
);


-- #####################################################################
-- # 2. Zarządzanie Treścią (Content Bounded Context)
-- #####################################################################

CREATE TABLE `books` (
    `id` BINARY(16) NOT NULL PRIMARY KEY,
    `author_id` BINARY(16) NOT NULL,
    `title` VARCHAR(255) NOT NULL,
    `content` JSON,
    `status` ENUM('draft', 'published') NOT NULL DEFAULT 'draft',
    `version` INT UNSIGNED NOT NULL DEFAULT 1,
    `price` DECIMAL(10, 2) NULL DEFAULT NULL,
    `currency` CHAR(3) NULL DEFAULT NULL,
    `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    `deleted_at` TIMESTAMP NULL DEFAULT NULL,
    FOREIGN KEY (`author_id`) REFERENCES `users`(`id`) ON DELETE CASCADE
);

CREATE TABLE `book_publications` (
    `id` BINARY(16) NOT NULL PRIMARY KEY,
    `book_id` BINARY(16) NOT NULL,
    `version` INT UNSIGNED NOT NULL,
    `content_snapshot` JSON NOT NULL,
    `metadata_snapshot` JSON NOT NULL COMMENT 'Snapshot of title, price, etc. at time of publication',
    `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY `book_version_unique` (`book_id`, `version`),
    FOREIGN KEY (`book_id`) REFERENCES `books`(`id`) ON DELETE CASCADE
);


-- #####################################################################
-- # 3. Recenzje (Reviews Bounded Context)
-- #####################################################################

CREATE TABLE `review_codes` (
    `id` BINARY(16) NOT NULL PRIMARY KEY,
    `book_id` BINARY(16) NOT NULL,
    `code` VARCHAR(255) NOT NULL UNIQUE,
    `expires_at` TIMESTAMP NOT NULL,
    `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (`book_id`) REFERENCES `books`(`id`) ON DELETE CASCADE
);

CREATE TABLE `review_comments` (
    `id` BINARY(16) NOT NULL PRIMARY KEY,
    `review_code_id` BINARY(16) NOT NULL,
    `commentable_id` BINARY(16) NOT NULL COMMENT 'ID of the book or book_item (block)',
    `commentable_type` VARCHAR(255) NOT NULL COMMENT 'e.g., App\\Models\\Book, App\\Models\\BookContentBlock',
    `comment` TEXT NOT NULL,
    `status` ENUM('new', 'in_progress', 'done', 'rejected') NOT NULL DEFAULT 'new',
    `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    `deleted_at` TIMESTAMP NULL DEFAULT NULL,
    FOREIGN KEY (`review_code_id`) REFERENCES `review_codes`(`id`) ON DELETE CASCADE,
    INDEX `commentable_index` (`commentable_id`, `commentable_type`)
);


-- #####################################################################
-- # 4. Sprzedaż i Opinie Publiczne (Sales Bounded Context)
-- #####################################################################

CREATE TABLE `orders` (
    `id` BINARY(16) NOT NULL PRIMARY KEY,
    `user_id` BINARY(16) NOT NULL,
    `book_id` BINARY(16) NOT NULL,
    `amount` DECIMAL(10, 2) NOT NULL,
    `currency` CHAR(3) NOT NULL,
    `status` ENUM('pending', 'paid', 'failed') NOT NULL DEFAULT 'pending',
    `transaction_id` VARCHAR(255) NULL COMMENT 'ID from mocked payment gateway',
    `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (`user_id`) REFERENCES `users`(`id`),
    FOREIGN KEY (`book_id`) REFERENCES `books`(`id`)
);

CREATE TABLE `library_items` (
    `user_id` BINARY(16) NOT NULL,
    `book_id` BINARY(16) NOT NULL,
    `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`user_id`, `book_id`),
    FOREIGN KEY (`user_id`) REFERENCES `users`(`id`) ON DELETE CASCADE,
    FOREIGN KEY (`book_id`) REFERENCES `books`(`id`) ON DELETE CASCADE
);

CREATE TABLE `public_opinions` (
    `id` BINARY(16) NOT NULL PRIMARY KEY,
    `book_id` BINARY(16) NOT NULL,
    `user_id` BINARY(16) NOT NULL,
    `rating` TINYINT UNSIGNED NOT NULL,
    `comment` TEXT NULL,
    `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    `deleted_at` TIMESTAMP NULL DEFAULT NULL,
    UNIQUE KEY `user_book_opinion_unique` (`user_id`, `book_id`),
    FOREIGN KEY (`book_id`) REFERENCES `books`(`id`) ON DELETE CASCADE,
    FOREIGN KEY (`user_id`) REFERENCES `users`(`id`) ON DELETE CASCADE
);


-- #####################################################################
-- # 5. Wsparcie (Support Bounded Context)
-- #####################################################################

CREATE TABLE `support_tickets` (
    `id` BINARY(16) NOT NULL PRIMARY KEY,
    `user_id` BINARY(16) NULL,
    `subject` VARCHAR(255) NOT NULL,
    `message` TEXT NOT NULL,
    `status` ENUM('new', 'open', 'closed') NOT NULL DEFAULT 'new',
    `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (`user_id`) REFERENCES `users`(`id`) ON DELETE SET NULL
);
```