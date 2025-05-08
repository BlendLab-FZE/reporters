

```sql

/* ============================================================
   0. EXTENSIONS
   ============================================================ */
create extension if not exists "pgcrypto";   -- uuid generator
create extension if not exists "pg_trgm";    -- trigram / FTS
create extension if not exists "pgvector";   -- AI embeddings

/* ============================================================
   1. ENUM TYPES
   ============================================================ */
create type badge_type        as enum ('none','ai_verified','human_verified','both');
create type article_status    as enum ('draft','ai_pending','human_review','published','archived');
create type visibility_level  as enum ('public','followers','subscribers','private');
create type vote_type         as enum ('up','down');
create type report_status     as enum ('pending','reviewed','dismissed');
create type media_type        as enum ('image','video','audio');
create type billing_cycle     as enum ('monthly','yearly');
create type payment_provider  as enum ('stripe','paypal','tap','paymob','inapp');
create type payment_status    as enum ('pending','succeeded','failed','refunded');
create type audit_op          as enum ('insert','update','delete');

/* ============================================================
   2. REFERENCE TABLES
   ============================================================ */
create table languages (
  code            char(2) primary key,
  code_alpha3     char(3) unique,
  name_en         text not null,
  native_name     text,
  direction       char(3) default 'LTR'   -- LTR / RTL / TTB
);

create table regions (
  id              uuid primary key default gen_random_uuid(),
  iso_code        char(3) unique,
  name_en         text not null,
  name_local      text,
  parent_id       uuid references regions(id)
);

create table currencies (
  code            char(3) primary key,
  name_en         text not null,
  minor_units     smallint not null default 2
);

/* ============================================================
   3. ADMIN ROLES & PERMISSIONS
   ============================================================ */
create table roles (
  id              uuid primary key default gen_random_uuid(),
  name            text unique not null,
  description     text
);

create table permissions (
  id              uuid primary key default gen_random_uuid(),
  key             text unique not null,
  description     text
);

create table role_permissions (
  role_id         uuid references roles(id) on delete cascade,
  permission_id   uuid references permissions(id) on delete cascade,
  primary key (role_id, permission_id)
);

create table admin_users (
  id              uuid primary key default gen_random_uuid(),
  user_id         uuid unique not null references auth.users(id) on delete cascade,
  role_id         uuid references roles(id),
  created_at      timestamptz default now()
);

/* ============================================================
   4. PLATFORM USERS (READERS / SUPPORTERS)
   ============================================================ */
create table platform_users (
  id                 uuid primary key default gen_random_uuid(),
  user_id            uuid references auth.users(id),
  display_name       text,
  profile_picture    text,
  preferred_regions  char(6)[],
  created_at         timestamptz default now()
);

/* ============================================================
   5. JOURNALISTS & ORGANISATIONS
   ============================================================ */
create table journalists (
  id                 uuid primary key default gen_random_uuid(),
  user_id            uuid unique not null references auth.users(id),
  display_name       text not null,
  bio                text,
  avatar_url         text,
  badge_type         badge_type default 'none',
  trust_score        numeric(3,2) default 0.00,
  preferred_region   uuid references regions(id),
  preferred_language char(2) references languages(code),
  created_at         timestamptz default now(),
  updated_at         timestamptz default now()
);

create table organizations (
  id            uuid primary key default gen_random_uuid(),
  org_type      text check (org_type in ('media','ngo','gov','company')),
  name          text not null,
  slug          text unique not null,
  logo_url      text,
  description   text,
  region_id     uuid references regions(id),
  created_at    timestamptz default now(),
  updated_at    timestamptz default now()
);

create table journalist_organizations (
  journalist_id    uuid references journalists(id) on delete cascade,
  organization_id  uuid references organizations(id) on delete cascade,
  role_in_org      text,
  joined_at        timestamptz default now(),
  primary key (journalist_id, organization_id)
);

create table journalist_followers (
  follower_user_id uuid references auth.users(id) on delete cascade,
  journalist_id    uuid references journalists(id) on delete cascade,
  followed_at      timestamptz default now(),
  primary key (follower_user_id, journalist_id)
);

/* ============================================================
   6. CONTENT (ARTICLES, VERSIONS, TRANSLATIONS, MEDIA)
   ============================================================ */
create table journalist_articles (
  id              uuid primary key default gen_random_uuid(),
  journalist_id   uuid references journalists(id) on delete set null,
  slug            text unique not null,
  main_language   char(2) references languages(code),
  status          article_status default 'draft',
  visibility      visibility_level default 'public',
  created_at      timestamptz default now(),
  updated_at      timestamptz default now(),
  published_at    timestamptz,
  deleted_at      timestamptz
);

create table article_versions (
  id              uuid primary key default gen_random_uuid(),
  article_id      uuid references journalist_articles(id) on delete cascade,
  version_number  integer not null,
  editor_user_id  uuid references auth.users(id),
  commit_message  text,
  delta           jsonb,
  created_at      timestamptz default now(),
  unique(article_id, version_number)
);

create table article_translations (
  id              uuid primary key default gen_random_uuid(),
  article_id      uuid references journalist_articles(id) on delete cascade,
  language_code   char(2) references languages(code),
  title           text not null,
  summary         text,
  content         text not null,
  seo_title       text,
  seo_description text,
  created_at      timestamptz default now(),
  updated_at      timestamptz default now(),
  unique(article_id, language_code)
);

create table tags (
  id   uuid primary key default gen_random_uuid(),
  name text not null,
  slug text unique not null
);

create table article_tags (
  article_id uuid references journalist_articles(id) on delete cascade,
  tag_id     uuid references tags(id) on delete cascade,
  primary key(article_id, tag_id)
);

create table categories (
  id        serial primary key,
  parent_id int references categories(id),
  name_en   text not null,
  name_ar   text
);

create table article_categories (
  article_id  uuid references journalist_articles(id) on delete cascade,
  category_id int references categories(id) on delete cascade,
  primary key(article_id, category_id)
);

create table media (
  id          uuid primary key default gen_random_uuid(),
  article_id  uuid references journalist_articles(id) on delete cascade,
  media_type  media_type not null,
  url         text not null,
  caption     text,
  "order"     int,
  uploaded_at timestamptz default now()
);

/* ============================================================
   7. FEEDBACK & MODERATION
   ============================================================ */
create table article_votes (
  user_id    uuid references auth.users(id) on delete cascade,
  article_id uuid references journalist_articles(id) on delete cascade,
  vote_type  vote_type not null,
  voted_at   timestamptz default now(),
  primary key(user_id, article_id)
);

create table article_reports (
  id               uuid primary key default gen_random_uuid(),
  reporter_user_id uuid references auth.users(id),
  article_id       uuid references journalist_articles(id) on delete cascade,
  reason           text,
  comment          text,
  status           report_status default 'pending',
  created_at       timestamptz default now(),
  reviewed_at      timestamptz,
  moderator_user_id uuid references admin_users(id)
);

create table article_badges (
  article_id uuid references journalist_articles(id) on delete cascade,
  badge_type badge_type,
  granted_by uuid references admin_users(id),
  granted_at timestamptz default now(),
  primary key(article_id, badge_type)
);

/* ============================================================
   8. MONETISATION
   ============================================================ */
create table subscription_tiers (
  id            uuid primary key default gen_random_uuid(),
  journalist_id uuid references journalists(id) on delete cascade,
  name          text not null,
  description   text,
  price_cents   int  not null,
  currency      char(3) references currencies(code),
  billing_cycle billing_cycle not null,
  active        boolean default true,
  created_at    timestamptz default now()
);

create table journalist_subscriptions (
  id                 uuid primary key default gen_random_uuid(),
  subscriber_user_id uuid references auth.users(id) on delete cascade,
  journalist_id      uuid references journalists(id) on delete cascade,
  tier_id            uuid references subscription_tiers(id),
  start_date         date not null,
  end_date           date,
  status             text check (status in ('active','canceled','expired')),
  stripe_subscription_id text,
  unique(subscriber_user_id, journalist_id)
);

create table payments (
  id                   uuid primary key default gen_random_uuid(),
  payer_user_id        uuid references auth.users(id),
  subscription_id      uuid references journalist_subscriptions(id),
  amount_cents         int not null,
  currency             char(3) references currencies(code),
  provider             payment_provider default 'stripe',
  provider_payment_id  text,
  status               payment_status default 'pending',
  metadata             jsonb,
  created_at           timestamptz default now()
);

create table donations (
  id                   uuid primary key default gen_random_uuid(),
  donor_user_id        uuid references auth.users(id),
  journalist_id        uuid references journalists(id) on delete cascade,
  amount_cents         int not null,
  currency             char(3) references currencies(code),
  provider             payment_provider default 'stripe',
  provider_payment_id  text,
  status               payment_status default 'pending',
  created_at           timestamptz default now()
);

/* ============================================================
   9. AI VERIFICATION & AUDIT
   ============================================================ */
create table ai_verification_logs (
  id          uuid primary key default gen_random_uuid(),
  article_id  uuid references journalist_articles(id) on delete cascade,
  model_name  text,
  confidence  numeric(4,3),
  verdict     text check (verdict in ('passed','flagged')),
  result_json jsonb,
  created_at  timestamptz default now()
);

create table audit_log_base (
  id            uuid primary key default gen_random_uuid(),
  table_name    text not null,
  record_id     uuid,
  operation     audit_op not null,
  changed_data  jsonb,
  performed_by  uuid references auth.users(id),
  performed_at  timestamptz not null
) partition by range(performed_at);

create table audit_log_2025_05 partition of audit_log_base
for values from ('2025-05-01') to ('2025-06-01');

/* ============================================================
   10. INDEXES
   ============================================================ */
create index idx_articles_status           on journalist_articles(status);
create index idx_articles_pubdate_desc     on journalist_articles(published_at desc);
create index idx_translations_lang         on article_translations(language_code);
create index idx_followers_count           on journalist_followers(journalist_id);
create index idx_payments_provider_date    on payments(provider, created_at desc);
create index idx_votes_article             on article_votes(article_id, vote_type);
create index idx_reports_status            on article_reports(status);

/* ============================================================
   11. TRIGGERS (updated_at)
   ============================================================ */
create or replace function touch_updated_at()
returns trigger language plpgsql as $$
begin
  new.updated_at := now();
  return new;
end $$;

create trigger trg_touch_journalists
before update on journalists
for each row execute function touch_updated_at();

create trigger trg_touch_articles
before update on journalist_articles
for each row execute function touch_updated_at();

create trigger trg_touch_translations
before update on article_translations
for each row execute function touch_updated_at();

/* ============================================================
   12. SEED DATA (minimal)
   ============================================================ */
-- 1) Insert a demo language & currency
insert into languages (code, code_alpha3, name_en, native_name) values
  ('en','eng','English','English')
on conflict do nothing;

insert into currencies (code,name_en,minor_units) values ('USD','US Dollar',2)
on conflict do nothing;

-- 2) Seed admin roles
insert into roles (name,description) values
  ('superadmin','Full system access'),
  ('editor','Content & badge review'),
  ('moderator','User & report moderation'),
  ('finance','Financial oversight')
on conflict do nothing;

-- 3) Create one demo Supabase user (replace email / pwd!)
insert into auth.users (id, email, encrypted_password, email_confirmed_at)
values
  (gen_random_uuid(),'admin@example.com','dummy',now())
on conflict do nothing;

-- 4) Map that auth user to superadmin
insert into admin_users (user_id, role_id)
select u.id, r.id
from auth.users u, roles r
where u.email='admin@example.com'
  and r.name='superadmin'
on conflict do nothing;

```
