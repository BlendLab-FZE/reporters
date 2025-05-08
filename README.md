# 📰 Reporters Platform – System Vision & User Journeys

**Version:** 1.0  
**Date:** May 2025  
**Project Scope:** Platform for Independent Journalists and Media Organizations  
**Language:** English

---

## 🎯 I. System Goals

The platform empowers independent journalists and media organizations by enabling:

- Publishing articles in multiple languages  
- Creating professional journalist profiles  
- Managing subscriptions and donations  
- Connecting journalists to media organizations  
- Providing direct interaction with articles (votes, reports, comments – coming soon)  
- Verifying articles using Artificial Intelligence (AI)  
- Flexible role-based administration and moderation  
- Supporting future features like digital wallets, videos, advertisements, etc.

---

## 🔧 II. Required Features

1. **User Accounts & Identity**
   - Sign up/login via email or social login
   - Email verification
   - Flexible role-based access system

2. **Journalist Profiles**
   - Full personal profile
   - Trust rating
   - Verification badges

3. **Article Publishing**
   - Multi-language support
   - Versioning & revisions

4. **Categories & Tags**
   - For enhanced browsing and filtering

5. **Multimedia Support**
   - Images, video, and audio attachments

6. **Engagement Features**
   - Upvotes/downvotes
   - Reporting inappropriate content
   - Comments (future)

7. **Subscriptions & Donations**
   - Connect readers with journalists financially

8. **AI Verification**
   - Store results of AI model review

9. **Audit Logging**
   - Track all admin/moderator edits

10. **Media Organizations**
    - Ability to associate journalists with institutions

---

## ✍️ III. Journalist Journey

### 1. Initial Registration
- Sign up with email/password or social login

### 2. Journalist Application
- Fill in personal information and resume
- Upload references (e.g., past work or awards)

### 3. Admin Review
- Admin verifies identity and references
- Upon approval, a journalist profile is created

### 4. Set Up Profile
- Add photo, bio, preferred language and region
- Include additional references or links

### 5. Link Media Organization (optional)
- Select existing media organization or request new one
- Set role/title and joining date

### 6. Create a New Article
- Click “New Article”
- Set publication status (e.g., draft)
- Choose content type (text/image)

### 7. Upload Media
- Upload a main image or photo gallery
- (Video uploads will be supported in future versions)

### 8. Write Original Content
- Add title, summary, and full text in the original language
- Flag `is_original = true` for the base version

### 9. Add Translations (optional)
- Select a target language
- Provide translated content manually or via AI
- Flag AI translations appropriately (`ai_translated = true`)

### 10. AI-Based Review
- Automated check for prohibited/misleading content
- If passed, assign `ai_verified` badge
- If flagged, sent for human moderator review

### 11. Publish Article
- Click “Publish” after passing validation
- Status is set to `published` and timestamp is recorded

### 12. Audience Interaction
- Readers can vote or report content
- Journalist receives notifications for interactions

### 13. Create Subscription Plans
- Add monthly packages (price, currency, description)
- Activate plan to make it visible to users

### 14. Receive Donations
- Readers can send one-time donations
- Payments are automatically recorded and shown in dashboard

### 15. Manage Subscriptions
- View subscriber count and total revenue
- Update pricing or deactivate plan anytime

### 16. Request Human Verification
- Upon achieving notable milestones, apply for human verification
- Moderator assigns `human_verified` or `both_verified` badges

### 17. Edit or Archive Articles
- Drafts can be edited
- Published articles can be archived (`archived = true`)

### 18. View Personal Statistics
- See number of followers, trust ratings, total donations, etc.

---

## 👥 IV. End-User Journey

### 1. Account Creation
- Sign up via email or social provider
- Confirm email and activate account

### 2. Profile Setup
- Set display name and avatar
- Choose preferred regions and languages for content

### 3. Explore Homepage
- Browse recent public articles filtered by language or region
- Use search bar and filters (category, tag, verification badge)

### 4. Read Articles
- Open full article with translated version matching UI language
- View images and gallery

### 5. Engage with Content
- Click 👍 “Upvote” or 👎 “Downvote”
- Report content with reason and optional note

### 6. Follow Journalists
- Visit journalist’s profile and click “Follow”
- Toggle notifications for new articles

### 7. One-Time Donations
- Select an amount and optional encouragement message
- Complete payment via Stripe or similar
- Receive receipt and update “My Contributions” dashboard

### 8. Monthly Subscriptions
- View available plans
- Select and complete recurring payment
- Gain access to “Subscribers Only” content

### 9. Manage Account & Contributions
- Dashboard displays active subscriptions, payment history, and donations
- Cancel subscription or update payment method

### 10. Customize Experience
- Change preferred languages and regions
- Manage followed journalists and notification settings

### 11. Comments & Discussions *(Coming Soon)*
- Add comments to articles
- Reply to other users

### 12. Account Deletion
- Request permanent data deletion or temporary deactivation
- Confirm via email before proceeding

---

