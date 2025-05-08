# 📚 Journalist Platform – Database Documentation

**Version 1.0 — May 2025**

---

## 0 ▪︎ Extensions & Enum Types

```sql
-- EXTENSIONS
CREATE EXTENSION IF NOT EXISTS "pgcrypto";      -- uuid generator
CREATE EXTENSION IF NOT EXISTS "pg_trgm";       -- trigram/FTS
CREATE EXTENSION IF NOT EXISTS "pgvector";      -- optional AI embeddings

-- ENUMS
CREATE TYPE badge_type        AS ENUM ('none','ai_verified','human_verified','both');
CREATE TYPE article_status    AS ENUM ('draft','ai_pending','human_review','published','archived');
CREATE TYPE visibility_level  AS ENUM ('public','followers','subscribers','private');
CREATE TYPE vote_type         AS ENUM ('up','down');
CREATE TYPE report_status     AS ENUM ('pending','reviewed','dismissed');
CREATE TYPE media_type        AS ENUM ('image','video','audio');
CREATE TYPE billing_cycle     AS ENUM ('monthly','yearly');
CREATE TYPE payment_provider  AS ENUM ('stripe','paypal','tap','paymob','inapp');
CREATE TYPE payment_status    AS ENUM ('pending','succeeded','failed','refunded');
CREATE TYPE audit_op          AS ENUM ('insert','update','delete');
```

---

## 1 ▪︎ Reference Tables

### 1.1 `languages`

| Column        | Type      | Constraints / Default | Note              |
| ------------- | --------- | --------------------- | ----------------- |
| `code`        | `CHAR(2)` | **PK**                | ISO-639-1         |
| `code_alpha3` | `CHAR(3)` | `UNIQUE`              | ISO-639-2/3       |
| `name_en`     | `TEXT`    | `NOT NULL`            | English name      |
| `native_name` | `TEXT`    |                       | Native script     |
| `direction`   | `CHAR(3)` | `DEFAULT 'LTR'`       | `LTR`,`RTL`,`TTB` |

---

### 1.2 `regions`

| Column       | Type      | Constraints / Default               | Note                 |
| ------------ | --------- | ----------------------------------- | -------------------- |
| `id`         | `UUID`    | **PK**, `DEFAULT gen_random_uuid()` |                      |
| `iso_code`   | `CHAR(3)` | `UNIQUE`                            | ISO-3166-2           |
| `name_en`    | `TEXT`    | `NOT NULL`                          |                      |
| `name_local` | `TEXT`    |                                     |                      |
| `parent_id`  | `UUID`    | `FK → regions(id)`                  | hierarchical regions |

---

### 1.3 `currencies`

| Column        | Type       | Constraints / Default | Note                     |
| ------------- | ---------- | --------------------- | ------------------------ |
| `code`        | `CHAR(3)`  | **PK**                | ISO-4217                 |
| `name_en`     | `TEXT`     | `NOT NULL`            |                          |
| `minor_units` | `SMALLINT` | `NOT NULL DEFAULT 2`  | number of decimal places |

---

## 2 ▪︎ Identity & Security

### 2.1 `users`

| Column          | Type          | Constraints / Default               |
| --------------- | ------------- | ----------------------------------- |
| `id`            | `UUID`        | **PK**, `DEFAULT gen_random_uuid()` |
| `email`         | `TEXT`        | `UNIQUE NOT NULL`                   |
| `password_hash` | `TEXT`        | `NOT NULL`                          |
| `is_verified`   | `BOOLEAN`     | `DEFAULT FALSE`                     |
| `created_at`    | `TIMESTAMPTZ` | `DEFAULT now()`                     |
| `updated_at`    | `TIMESTAMPTZ` | `DEFAULT now()`                     |

---

### 2.2 `roles`

| Column        | Type   | Constraints                         |
| ------------- | ------ | ----------------------------------- |
| `id`          | `UUID` | **PK**, `DEFAULT gen_random_uuid()` |
| `name`        | `TEXT` | `UNIQUE NOT NULL`                   |
| `description` | `TEXT` |                                     |

---

### 2.3 `permissions`

| Column        | Type   | Constraints                         |
| ------------- | ------ | ----------------------------------- |
| `id`          | `UUID` | **PK**, `DEFAULT gen_random_uuid()` |
| `key`         | `TEXT` | `UNIQUE NOT NULL`                   |
| `description` | `TEXT` |                                     |

---

### 2.4 `role_permissions`

| Column          | Type   | Constraints                    |
| --------------- | ------ | ------------------------------ |
| `role_id`       | `UUID` | **PK**, `FK → roles(id)`       |
| `permission_id` | `UUID` | **PK**, `FK → permissions(id)` |

---

### 2.5 `admin_users`

| Column       | Type          | Constraints / Default               |
| ------------ | ------------- | ----------------------------------- |
| `id`         | `UUID`        | **PK**, `DEFAULT gen_random_uuid()` |
| `user_id`    | `UUID`        | `UNIQUE NOT NULL FK → users(id)`    |
| `role_id`    | `UUID`        | `FK → roles(id)`                    |
| `created_at` | `TIMESTAMPTZ` | `DEFAULT now()`                     |

---

### 2.6 `platform_users`  *(readers/supporters)*

| Column              | Type          | Constraints / Default               |
| ------------------- | ------------- | ----------------------------------- |
| `id`                | `UUID`        | **PK**, `DEFAULT gen_random_uuid()` |
| `user_id`           | `UUID`        | `FK → users(id)`                    |
| `display_name`      | `TEXT`        |                                     |
| `profile_picture`   | `TEXT`        |                                     |
| `preferred_regions` | `CHAR(6)[]`   |                                     |
| `created_at`        | `TIMESTAMPTZ` | `DEFAULT now()`                     |

---

## 3 ▪︎ Journalists & Organizations

### 3.1 `journalists`

| Column               | Type           | Constraints / Default               |
| -------------------- | -------------- | ----------------------------------- |
| `id`                 | `UUID`         | **PK**, `DEFAULT gen_random_uuid()` |
| `user_id`            | `UUID`         | `UNIQUE NOT NULL FK → users(id)`    |
| `display_name`       | `TEXT`         | `NOT NULL`                          |
| `bio`                | `TEXT`         |                                     |
| `avatar_url`         | `TEXT`         |                                     |
| `badge_type`         | `badge_type`   | `DEFAULT 'none'`                    |
| `trust_score`        | `NUMERIC(3,2)` | `DEFAULT 0.00`                      |
| `preferred_region`   | `UUID`         | `FK → regions(id)`                  |
| `preferred_language` | `CHAR(2)`      | `FK → languages(code)`              |
| `created_at`         | `TIMESTAMPTZ`  | `DEFAULT now()`                     |
| `updated_at`         | `TIMESTAMPTZ`  | `DEFAULT now()`                     |

---

### 3.2 `organizations`

| Column        | Type          | Constraints / Default                                 |
| ------------- | ------------- | ----------------------------------------------------- |
| `id`          | `UUID`        | **PK**, `DEFAULT gen_random_uuid()`                   |
| `org_type`    | `TEXT`        | `CHECK (org_type IN ('media','ngo','gov','company'))` |
| `name`        | `TEXT`        | `NOT NULL`                                            |
| `slug`        | `TEXT`        | `UNIQUE NOT NULL`                                     |
| `logo_url`    | `TEXT`        |                                                       |
| `description` | `TEXT`        |                                                       |
| `region_id`   | `UUID`        | `FK → regions(id)`                                    |
| `created_at`  | `TIMESTAMPTZ` | `DEFAULT now()`                                       |
| `updated_at`  | `TIMESTAMPTZ` | `DEFAULT now()`                                       |

---

### 3.3 `journalist_organizations`

| Column            | Type          | Constraints                      |
| ----------------- | ------------- | -------------------------------- |
| `journalist_id`   | `UUID`        | **PK**, `FK → journalists(id)`   |
| `organization_id` | `UUID`        | **PK**, `FK → organizations(id)` |
| `role_in_org`     | `TEXT`        |                                  |
| `joined_at`       | `TIMESTAMPTZ` | `DEFAULT now()`                  |

---

### 3.4 `journalist_followers`

| Column             | Type          | Constraints / Default          |
| ------------------ | ------------- | ------------------------------ |
| `follower_user_id` | `UUID`        | **PK**, `FK → users(id)`       |
| `journalist_id`    | `UUID`        | **PK**, `FK → journalists(id)` |
| `followed_at`      | `TIMESTAMPTZ` | `DEFAULT now()`                |

---

## 4 ▪︎ Content (Articles & Media)

### 4.1 `journalist_articles`

| Column          | Type               | Constraints / Default               |
| --------------- | ------------------ | ----------------------------------- |
| `id`            | `UUID`             | **PK**, `DEFAULT gen_random_uuid()` |
| `journalist_id` | `UUID`             | `FK → journalists(id)`              |
| `slug`          | `TEXT`             | `UNIQUE NOT NULL`                   |
| `main_language` | `CHAR(2)`          | `FK → languages(code)`              |
| `status`        | `article_status`   | `DEFAULT 'draft'`                   |
| `visibility`    | `visibility_level` | `DEFAULT 'public'`                  |
| `created_at`    | `TIMESTAMPTZ`      | `DEFAULT now()`                     |
| `updated_at`    | `TIMESTAMPTZ`      | `DEFAULT now()`                     |
| `published_at`  | `TIMESTAMPTZ`      |                                     |
| `deleted_at`    | `TIMESTAMPTZ`      |                                     |

---

### 4.2 `article_versions`

| Column           | Type                            | Constraints / Default               |
| ---------------- | ------------------------------- | ----------------------------------- |
| `id`             | `UUID`                          | **PK**, `DEFAULT gen_random_uuid()` |
| `article_id`     | `UUID`                          | `FK → journalist_articles(id)`      |
| `version_number` | `INTEGER`                       | `NOT NULL`                          |
| `editor_user_id` | `UUID`                          | `FK → users(id)`                    |
| `commit_message` | `TEXT`                          |                                     |
| `delta`          | `JSONB`                         | (diff or full snapshot)             |
| `created_at`     | `TIMESTAMPTZ`                   | `DEFAULT now()`                     |
| `UNIQUE`         | (`article_id`,`version_number`) |                                     |

---

### 4.3 `article_translations`

| Column            | Type                           | Constraints / Default               |
| ----------------- | ------------------------------ | ----------------------------------- |
| `id`              | `UUID`                         | **PK**, `DEFAULT gen_random_uuid()` |
| `article_id`      | `UUID`                         | `FK → journalist_articles(id)`      |
| `language_code`   | `CHAR(2)`                      | `FK → languages(code)`              |
| `title`           | `TEXT`                         | `NOT NULL`                          |
| `summary`         | `TEXT`                         |                                     |
| `content`         | `TEXT`                         | `NOT NULL`                          |
| `seo_title`       | `TEXT`                         |                                     |
| `seo_description` | `TEXT`                         |                                     |
| `created_at`      | `TIMESTAMPTZ`                  | `DEFAULT now()`                     |
| `updated_at`      | `TIMESTAMPTZ`                  | `DEFAULT now()`                     |
| `UNIQUE`          | (`article_id`,`language_code`) |                                     |

---

### 4.4 `tags`

| Column | Type   | Constraints / Default               |
| ------ | ------ | ----------------------------------- |
| `id`   | `UUID` | **PK**, `DEFAULT gen_random_uuid()` |
| `name` | `TEXT` | `NOT NULL`                          |
| `slug` | `TEXT` | `UNIQUE NOT NULL`                   |

---

### 4.5 `article_tags`

| Column       | Type   | Constraints                            |
| ------------ | ------ | -------------------------------------- |
| `article_id` | `UUID` | **PK**, `FK → journalist_articles(id)` |
| `tag_id`     | `UUID` | **PK**, `FK → tags(id)`                |

---

### 4.6 `categories`

| Column      | Type     | Constraints / Default |
| ----------- | -------- | --------------------- |
| `id`        | `SERIAL` | **PK**                |
| `parent_id` | `INT`    | `FK → categories(id)` |
| `name_en`   | `TEXT`   | `NOT NULL`            |
| `name_ar`   | `TEXT`   |                       |

---

### 4.7 `article_categories`

| Column        | Type   | Constraints                            |
| ------------- | ------ | -------------------------------------- |
| `article_id`  | `UUID` | **PK**, `FK → journalist_articles(id)` |
| `category_id` | `INT`  | **PK**, `FK → categories(id)`          |

---

### 4.8 `media`

| Column        | Type          | Constraints / Default               |
| ------------- | ------------- | ----------------------------------- |
| `id`          | `UUID`        | **PK**, `DEFAULT gen_random_uuid()` |
| `article_id`  | `UUID`        | `FK → journalist_articles(id)`      |
| `media_type`  | `media_type`  | `NOT NULL`                          |
| `url`         | `TEXT`        | `NOT NULL`                          |
| `caption`     | `TEXT`        |                                     |
| `"order"`     | `INT`         |                                     |
| `uploaded_at` | `TIMESTAMPTZ` | `DEFAULT now()`                     |

---

## 5 ▪︎ Interaction

### 5.1 `article_votes`

| Column       | Type          | Constraints / Default                  |
| ------------ | ------------- | -------------------------------------- |
| `user_id`    | `UUID`        | **PK**, `FK → users(id)`               |
| `article_id` | `UUID`        | **PK**, `FK → journalist_articles(id)` |
| `vote_type`  | `vote_type`   | `NOT NULL`                             |
| `voted_at`   | `TIMESTAMPTZ` | `DEFAULT now()`                        |

---

### 5.2 `article_reports`

| Column              | Type            | Constraints / Default               |
| ------------------- | --------------- | ----------------------------------- |
| `id`                | `UUID`          | **PK**, `DEFAULT gen_random_uuid()` |
| `reporter_user_id`  | `UUID`          | `FK → users(id)`                    |
| `article_id`        | `UUID`          | `FK → journalist_articles(id)`      |
| `reason`            | `TEXT`          |                                     |
| `comment`           | `TEXT`          |                                     |
| `status`            | `report_status` | `DEFAULT 'pending'`                 |
| `created_at`        | `TIMESTAMPTZ`   | `DEFAULT now()`                     |
| `reviewed_at`       | `TIMESTAMPTZ`   |                                     |
| `moderator_user_id` | `UUID`          | `FK → admin_users(id)`              |

---

### 5.3 `article_badges`

| Column       | Type          | Constraints / Default                  |
| ------------ | ------------- | -------------------------------------- |
| `article_id` | `UUID`        | **PK**, `FK → journalist_articles(id)` |
| `badge_type` | `badge_type`  | **PK**                                 |
| `granted_by` | `UUID`        | `FK → admin_users(id)`                 |
| `granted_at` | `TIMESTAMPTZ` | `DEFAULT now()`                        |

---

## 6 ▪︎ Monetisation

### 6.1 `subscription_tiers`

| Column          | Type            | Constraints / Default               |
| --------------- | --------------- | ----------------------------------- |
| `id`            | `UUID`          | **PK**, `DEFAULT gen_random_uuid()` |
| `journalist_id` | `UUID`          | `FK → journalists(id)`              |
| `name`          | `TEXT`          | `NOT NULL`                          |
| `description`   | `TEXT`          |                                     |
| `price_cents`   | `INT`           | `NOT NULL`                          |
| `currency`      | `CHAR(3)`       | `FK → currencies(code)`             |
| `billing_cycle` | `billing_cycle` | `NOT NULL`                          |
| `active`        | `BOOLEAN`       | `DEFAULT TRUE`                      |
| `created_at`    | `TIMESTAMPTZ`   | `DEFAULT now()`                     |

---

### 6.2 `journalist_subscriptions`

| Column                   | Type                                   | Constraints / Default                               |
| ------------------------ | -------------------------------------- | --------------------------------------------------- |
| `id`                     | `UUID`                                 | **PK**, `DEFAULT gen_random_uuid()`                 |
| `subscriber_user_id`     | `UUID`                                 | `FK → users(id)`                                    |
| `journalist_id`          | `UUID`                                 | `FK → journalists(id)`                              |
| `tier_id`                | `UUID`                                 | `FK → subscription_tiers(id)`                       |
| `start_date`             | `DATE`                                 | `NOT NULL`                                          |
| `end_date`               | `DATE`                                 |                                                     |
| `status`                 | `TEXT`                                 | `CHECK (status IN ('active','canceled','expired'))` |
| `stripe_subscription_id` | `TEXT`                                 |                                                     |
| `UNIQUE`                 | (`subscriber_user_id`,`journalist_id`) |                                                     |

---

### 6.3 `payments`

| Column                | Type               | Constraints / Default               |
| --------------------- | ------------------ | ----------------------------------- |
| `id`                  | `UUID`             | **PK**, `DEFAULT gen_random_uuid()` |
| `payer_user_id`       | `UUID`             | `FK → users(id)`                    |
| `subscription_id`     | `UUID`             | `FK → journalist_subscriptions(id)` |
| `amount_cents`        | `INT`              | `NOT NULL`                          |
| `currency`            | `CHAR(3)`          | `FK → currencies(code)`             |
| `provider`            | `payment_provider` | `DEFAULT 'stripe'`                  |
| `provider_payment_id` | `TEXT`             |                                     |
| `status`              | `payment_status`   | `DEFAULT 'pending'`                 |
| `metadata`            | `JSONB`            | non-PII                             |
| `created_at`          | `TIMESTAMPTZ`      | `DEFAULT now()`                     |

---

### 6.4 `donations`

| Column                | Type               | Constraints / Default               |
| --------------------- | ------------------ | ----------------------------------- |
| `id`                  | `UUID`             | **PK**, `DEFAULT gen_random_uuid()` |
| `donor_user_id`       | `UUID`             | `FK → users(id)`                    |
| `journalist_id`       | `UUID`             | `FK → journalists(id)`              |
| `amount_cents`        | `INT`              | `NOT NULL`                          |
| `currency`            | `CHAR(3)`          | `FK → currencies(code)`             |
| `provider`            | `payment_provider` | `DEFAULT 'stripe'`                  |
| `provider_payment_id` | `TEXT`             |                                     |
| `status`              | `payment_status`   | `DEFAULT 'pending'`                 |
| `created_at`          | `TIMESTAMPTZ`      | `DEFAULT now()`                     |

---

## 7 ▪︎ AI Verification & Audit

### 7.1 `ai_verification_logs`

| Column        | Type           | Constraints / Default                     |
| ------------- | -------------- | ----------------------------------------- |
| `id`          | `UUID`         | **PK**, `DEFAULT gen_random_uuid()`       |
| `article_id`  | `UUID`         | `FK → journalist_articles(id)`            |
| `model_name`  | `TEXT`         |                                           |
| `confidence`  | `NUMERIC(4,3)` | 0-1.000                                   |
| `verdict`     | `TEXT`         | `CHECK (verdict IN ('passed','flagged'))` |
| `result_json` | `JSONB`        | raw AI output                             |
| `created_at`  | `TIMESTAMPTZ`  | `DEFAULT now()`                           |

---

### 7.2 `audit_log_base`  *(partitioned monthly)*

| Column         | Type                  |
| -------------- | --------------------- |
| `id`           | `UUID PK`             |
| `table_name`   | `TEXT`                |
| `record_id`    | `UUID`                |
| `operation`    | `audit_op`            |
| `changed_data` | `JSONB`               |
| `performed_by` | `UUID FK → users(id)` |
| `performed_at` | `TIMESTAMPTZ`         |

> Each month a child table like `audit_log_2025_05` is created.

---

## 8 ▪︎ Indexes & Optimization (key examples)

```sql
CREATE INDEX idx_article_status        ON journalist_articles(status);
CREATE INDEX idx_article_pubdate_desc  ON journalist_articles(published_at DESC);
CREATE INDEX idx_translation_lang      ON article_translations(language_code);
CREATE INDEX idx_follow_count          ON journalist_followers(journalist_id);
CREATE INDEX idx_payments_provider     ON payments(provider, created_at DESC);
CREATE INDEX idx_votes_article         ON article_votes(article_id, vote_type);
CREATE INDEX idx_reports_status        ON article_reports(status);
```

---

## 9 ▪︎ House-keeping Triggers

```sql
CREATE OR REPLACE FUNCTION touch_updated_at()
RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN NEW.updated_at = now(); RETURN NEW; END $$;

CREATE TRIGGER trg_touch_users
  BEFORE UPDATE ON users
  FOR EACH ROW EXECUTE FUNCTION touch_updated_at();

-- similar triggers can be added to journalist_articles, article_translations, etc.
```

