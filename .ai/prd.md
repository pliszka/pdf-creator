# Product Requirements Document (PRD) - PDF Books Generator
## 1. Product Overview
The PDF Books Generator is a web-based platform designed to provide a seamless, all-in-one solution for digital content creators. It enables authors to write, edit, and structure their content using a modular editor, manage a peer-review process with invited experts, generate professional-grade PDF files, and sell their finished ebooks. The Minimum Viable Product (MVP) focuses on delivering these core functionalities with a pre-defined template and a mocked payment system to validate the core user journey and gather feedback from early adopters. The primary target audience consists of ebook authors, coaches, and marketers who require an efficient tool to bring their digital products to market.

## 2. User Problem
Content creators currently rely on a fragmented ecosystem of tools to produce and sell digital books. They use one application for writing (e.g., Google Docs, Scrivener), another for layout and PDF conversion (e.g., InDesign, Canva), a third for gathering feedback (e.g., email, shared drives), and yet another for sales and distribution (e.g., Gumroad, Shopify). This process is inefficient, technically challenging, and costly. The PDF Books Generator aims to solve this by integrating the entire lifecycle of an ebook—from creation to sale—into a single, user-friendly platform.

## 3. Functional Requirements
- 3.1. User Management:
    - User registration via email and password (no email confirmation required for MVP).
    - User login and session-based authentication.
    - Three user roles: Author, Reviewer, Buyer.
    - One administrative role: Superadmin.

- 3.2. Content Creation:
    - A modular, drag-and-drop editor for structuring book content.
    - Support for two block types: Text and Image.
    - Text block supports H1, H2, H3, and Paragraph (P) elements.
    - Image block supports uploading images up to 5MB, with automatic optimization to <1MB.
    - Real-time validation for content limits (5,000 chars/block, 100 blocks/doc, 50,000 chars/doc).
    - Auto-saving of drafts (`last_modified_at` timestamp).

- 3.3. PDF Generation:
    - Synchronous PDF generation using a `pdfmake` microservice.
    - Generation is based on a structured JSON representation of the content.
    - A single, non-configurable template is used for all generated PDFs.
    - Support for an optional watermark containing the buyer's email address.

- 3.4. Review Process:
    - Authors can generate unique, 7-day valid access codes for reviewers.
    - Reviewers access a read-only version of the book via a special link and code.
    - Reviewers can add comments to specific content blocks.
    - Authors can view all comments and manage their status (`new`, `in_progress`, `done`, `rejected`).

- 3.5. Sales and Distribution:
    - A mocked payment gateway to simulate the purchase flow.
    - Authors set a price and currency (PLN/EUR) for their books.
    - Buyers get perpetual access to purchased books and their future updates in their user panel.
    - Buyers can leave public-facing reviews (star rating + text) on the book's sales page.

- 3.6. Support and Administration:
    - An integrated support form for users to submit issues.
    - Each support ticket is assigned a unique UUID for tracking.
    - A Superadmin panel for user management, book oversight, and review moderation.

## 4. Product Boundaries
- 4.1. In Scope for MVP:
    - User registration and login.
    - Modular content editor with text and image blocks.
    - Generation of PDFs using a single template.
    - Review process with access codes and in-line comments.
    - Mocked payment gateway and sales flow.
    - Basic user panels for authors, buyers, and reviewers.
    - Basic analytics for authors (sales count, total revenue).
    - Manual SEO settings (title, description) for book pages.

- 4.2. Out of Scope for MVP:
    - Integration with real payment gateways (Stripe, etc.).
    - Automated payout system for authors.
    - Discount codes or advanced pricing models.
    - User-configurable PDF templates or styles.
    - Advanced account management (e.g., changing email).
    - Multi-author collaboration on a single book.
    - Email notifications for reviews or sales.
    - Custom domain support for book pages.

## 5. User Stories
### Authentication
- ID: US-001
- Title: User Registration
- Description: As a new user, I want to create an account using my email and a password so that I can access the platform's features.
- Acceptance Criteria:
    - 1. The registration form must accept an email and a password.
    - 2. Upon successful submission, a new user account is created in the system.
    - 3. The user is automatically logged in and redirected to their dashboard.
    - 4. An error message is displayed if the email is already in use.

- ID: US-002
- Title: User Login
- Description: As a registered user, I want to log in to my account so that I can access my books and settings.
- Acceptance Criteria:
    - 1. The login form must accept an email and a password.
    - 2. Upon successful authentication, the user is redirected to their dashboard.
    - 3. An error message is displayed for invalid credentials.

### Author Experience
- ID: US-003
- Title: Create a New Book
- Description: As an author, I want to start a new book project so that I can begin adding content.
- Acceptance Criteria:
    - 1. There is a "Create New Book" button in my dashboard.
    - 2. Clicking it prompts me for a title.
    - 3. A new, empty book project is created and I am taken to the editor.

- ID: US-004
- Title: Add Content Blocks
- Description: As an author, I want to add text and image blocks to my book to build its content.
- Acceptance Criteria:
    - 1. A "+" button is available in the editor to add new blocks.
    - 2. Clicking it opens a modal where I can select "Text" or "Image".
    - 3. Selecting a type adds a new, empty block of that type to the editor.

- ID: US-005
- Title: Edit and Format Text
- Description: As an author, I want to edit text within a text block and format it as a heading (H1, H2, H3) or a paragraph.
- Acceptance Criteria:
    - 1. I can type and modify text within text blocks.
    - 2. Each text element within a block can be styled as H1, H2, H3, or P.
    - 3. The editor provides real-time feedback if a block exceeds the character limit.

- ID: US-006
- Title: Upload an Image
- Description: As an author, I want to upload an image to an image block.
- Acceptance Criteria:
    - 1. I can select an image file from my computer to upload into an image block.
    - 2. The image must not exceed 5MB.
    - 3. After upload, a preview of the image is visible in the editor.

- ID: US-007
- Title: Manage Content Blocks
- Description: As an author, I want to reorder and delete blocks to structure my document.
- Acceptance Criteria:
    - 1. I can drag and drop blocks to change their order in the document.
    - 2. Each block has a "delete" icon.
    - 3. Clicking "delete" shows a confirmation popup.
    - 4. Confirming deletion removes the block. A temporary "Undo" option appears.

- ID: US-008
- Title: Generate a PDF
- Description: As an author, I want to generate a PDF of my book to preview the final output.
- Acceptance Criteria:
    - 1. There is a "Generate PDF" button in the editor.
    - 2. Clicking it sends the book's content to the PDF service.
    - 3. The generated PDF is downloaded to my computer or opened in a new tab.
    - 4. A user-friendly error is shown if generation fails.

- ID: US-009
- Title: Invite Reviewers
- Description: As an author, I want to generate access links for reviewers so they can provide feedback.
- Acceptance Criteria:
    - 1. In the book's settings, there is a "Reviews" section.
    - 2. I can click a button to "Generate Review Code".
    - 3. The system generates a unique code and provides a full invitation link to copy.
    - 4. The code is active for 7 days.

- ID: US-010
- Title: Manage Reviews
- Description: As an author, I want to view feedback from reviewers and manage its status.
- Acceptance Criteria:
    - 1. I can see all comments from all reviewers associated with my book.
    - 2. For each comment, I can change its status via a dropdown menu (`new`, `in_progress`, `done`, `rejected`).
    - 3. The status change is saved immediately.

- ID: US-011
- Title: Publish and Price a Book
- Description: As an author, I want to set a price for my book and publish it for sale.
- Acceptance Criteria:
    - 1. In the book's settings, I can set a price and select a currency (PLN or EUR).
    - 2. I can click a "Publish" button.
    - 3. Publishing makes the book's sales page publicly accessible.
    - 4. I can unpublish a book, which makes its sales page return a 404 error.

### Reviewer Experience
- ID: US-012
- Title: Access a Book for Review
- Description: As a reviewer, I want to use a link and a code to access a book for review.
- Acceptance Criteria:
    - 1. I can open the provided review link.
    - 2. The page prompts me to enter my access code.
    - 3. Upon entering a valid, active code, I can view the book's content.
    - 4. An error message is shown if the code is invalid or expired.

- ID: US-013
- Title: Provide Feedback
- Description: As a reviewer, I want to add comments to specific parts of the book.
- Acceptance Criteria:
    - 1. I can add a comment to any content block.
    - 2. My submitted comments are visible to me and the author.
    - 3. I cannot see comments left by other reviewers.

### Buyer Experience
- ID: US-014
- Title: Purchase a Book
- Description: As a buyer, I want to purchase a book using the mocked payment system.
- Acceptance Criteria:
    - 1. The public sales page displays the book's title, description, author, and price.
    - 2. Clicking "Buy Now" simulates a payment process.
    - 3. After a successful "payment," the book is added to my account's library.

- ID: US-015
- Title: Access Purchased Book
- Description: As a buyer, I want to access my purchased books at any time from my dashboard.
- Acceptance Criteria:
    - 1. My dashboard contains a "My Library" section.
    - 2. It lists all books I have purchased.
    - 3. Clicking a book allows me to download the latest version of its PDF.

- ID: US-016
- Title: Leave a Public Opinion
- Description: As a buyer, I want to leave a public rating and review for a book I have purchased.
- Acceptance Criteria:
    - 1. In my library, I have an option to "Leave a Review" for a purchased book.
    - 2. I can select a star rating (1-5) and write a text comment.
    - 3. My submitted opinion is publicly visible on the book's sales page.

### System & Support
- ID: US-017
- Title: Submit a Support Ticket
- Description: As a logged-in user, I want to submit a support request through a form.
- Acceptance Criteria:
    - 1. A "Support" or "Contact" link is available in my dashboard.
    - 2. The form allows me to describe my issue.
    - 3. Upon submission, a support ticket is created in the system.
    - 4. I am shown a unique UUID for my ticket for future reference.

## 6. Success Metrics
- 6.1. Primary Business Goal:
    - Achieve 1,000 ebook sales per month (long-term target).

- 6.2. Author Adoption KPIs:
    - Onboard 2 active authors within the first month.
    - Onboard 10 active authors within the first quarter.
    - Onboard 100 active authors within the first year.
    - An "active author" is defined as a user who has published at least one book within the last 3 months.

- 6.3. Engagement Metrics:
    - Number of PDFs generated per week.
    - Number of reviews submitted per week.
    - Ratio of published books to created book projects.
```