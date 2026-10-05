# Moodbite.ai (Version 2.0)

> **Emotion-Aware Indian Culinary Recommendation Engine & Interactive Cooking Assistant**  
> Engineering Capstone Project by **Saksham Tiwari**

[![Next.js](https://img.shields.io/badge/Next.js-15.5.27-black?logo=next.js)](https://nextjs.org/)
[![React](https://img.shields.io/badge/React-19.1.0-blue?logo=react)](https://react.dev/)
[![Google Gemini](https://img.shields.io/badge/Google-Gemini%203.5%20Flash-4285F4?logo=google)](https://aistudio.google.com/)
[![Hugging Face](https://img.shields.io/badge/RoBERTa-GoEmotions-yellow?logo=huggingface)](https://huggingface.co/SamLowe/roberta-base-go_emotions)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

🌐 **Live Production Deployment**: [**https://moodbiteai.vercel.app/**](https://moodbiteai.vercel.app/)

---

## 📖 Project Overview

**Moodbite.ai (Version 2.0)** is an intelligent Indian culinary recommendation platform that pairs affective sentiment analysis with kitchen pantry availability. The application interprets user emotions from natural language text or voice expressions, combines them with available kitchen ingredients (entered manually or scanned via multimodal computer vision), and generates **4 to 5 culturally authentic Indian meal recommendations** filtered through a deterministic dietary keyword heuristic (Vegetarian vs. Non-Vegetarian).

Once a user selects a recipe, Moodbite.ai transitions into an interactive sous-chef—providing precise measurements, step-by-step cooking checklists, professional culinary tips, non-medical culinary style rationales (focusing solely on flavor, warmth, and texture), authentic Pexels dish photography, and hands-free Web Speech API narration.

---

## ✨ Version 2.0 Key Enhancements

1. **Conversational Intent Guard & Chit-Chat Handler**:
   - Intelligent detection of casual greetings (*"hey"*, *"hello"*, *"namaste"*) and unfamiliar/gibberish non-food inputs.
   - Prevents fabricating dishes inappropriately when no food or emotional intent is present.
   - Offers warm conversational guidance and 1-click interactive prompt pills to help users get started effortlessly.

2. **Modern Minimalist Dark UI & Centered Hero Layout**:
   - High-craft dark glassmorphism design (`bg-gradient-to-br from-gray-950 via-slate-950 to-black`) inspired by modern AI interfaces.
   - Centered hero with elegant serif styling (*"How are you feeling?"*) transitioning seamlessly into a conversational feed upon interaction.
   - Profile avatar popover menu with account state, Pantry Manager shortcut, and Supabase Authentication links.

3. **In-Chat Dietary Selector with Zero-Clipping Popover**:
   - Integrated dietary selector pill directly inside the input toolbar (`[🥬 Veg ▾]`, `[🍗 Non-Veg ▾]`, `[🍲 Any ▾]`).
   - Positioned with high z-index and overflow-visible boundaries to prevent UI clipping across all screen sizes.

4. **Dish Feedback Mechanism (Like / Dislike)**:
   - Interactive 👍 and 👎 feedback controls embedded on every recommended dish card and recipe detail view.
   - Dual-layer persistence: cached locally in `localStorage` for zero-latency recall and synced asynchronously to Supabase (`user_dish_feedback`) for continuous recommendation personalization.

5. **Elimination of Arbitrary AI Self-Ratings & Model Tags**:
   - Clean, transparent user interface without arbitrary match percentages or intrusive model provider labels.
   - Grounded in verifiable signals: exact pantry ingredient match counts, dietary compliance checkmarks, and explicit preparation fallback notices.

6. **Comprehensive Mobile & Desktop Responsiveness**:
   - Touch-friendly tap targets (minimum 44px), fluid typography, and zero horizontal overflow across devices.

---

## 🏛️ System Architecture

```
                                  [ User Interface ]
                       (Voice / Text / Camera / Pantry Selector)
                                          │
                                          ▼
                               [ Input Validation Layer ]
                             (lib/validation.js with Zod)
                                          │
                                          ▼
                         [ Isolated Route Rate Limiter ]
                   (lib/rateLimit.js - Keyed by IP:RouteKey)
                                          │
                   ┌──────────────────────┴──────────────────────┐
                   ▼                                             ▼
       [ Emotion Inference Head ]                    [ Multimodal Vision ]
        RoBERTa-base (GoEmotions)                   Gemini 3.5 Flash Lite
         (Hugging Face API Router)                  (Ingredient Extraction)
         (1.5s AbortController cap)                   (Capped at 25 items)
                   │                                             │
                   └──────────────────────┬──────────────────────┘
                                          │
                       (Emotion label + User context)
                                          │
                                          ▼
                             [ Conversational Intent Guard ]
                      ├── Greeting Detection ("hey", "hello")
                      ├── Bot Help / Guidance Queries
                      └── Unfamiliar Non-Culinary Input Fallback
                                          │
                               (Intent: Food / Mood Query)
                                          ▼
                             [ Primary Reasoning Engine ]
                            Google Gemini 3.5 Flash Lite
                              (Ultra-fast ~800ms tier)
                                          │
                               (Fallback on failure)
                                          ▼
                            [ Secondary Fallback Engine ]
                               Google Gemini 3.5 Flash
                                          │
                               (Fallback on failure)
                                          ▼
                            [ Tertiary Fallback Engine ]
                              Anthropic Claude 3 Haiku
                                (OpenRouter Provider)
                                          │
                                          ▼
                        [ Deterministic Post-Processing ]
                       (lib/dietaryCheck.js - Non-LLM)
                       ├── Meat-Keyword Heuristic Elimination (Veg)
                       ├── Vegetarian Exception Allowlist (soya keema, etc.)
                       ├── Ingredient Occurrence Verification
                       └── Zod Schema Conformance Check
                                          │
                                          ▼
                             [ Visual & Audio Enrichment ]
                             ├── Pexels API (Dish Photography with CDN Cache)
                             └── Web Speech API (Voice Narration)
```

---

## 🛡️ Reliability & Security Highlights

### 1. Key Security and Access Architecture
All backend AI and Vision API keys (`GEMINI_API_KEY`, `HUGGING_FACE_API_TOKEN`, `OPENROUTER_API_KEY`, `PEXELS_API_KEY`) are strictly loaded and executed within serverless routes (`pages/api/*`), never exposed to the client. The `NEXT_PUBLIC_SUPABASE_ANON_KEY` is public by design for client-side authentication, and all database tables are protected using Supabase Row Level Security (RLS) policies (`auth.uid() = user_id`).

### 2. Request Input Validation
All API endpoints validate incoming parameters via [Zod](https://zod.dev):
- **`/api/suggestFood`**: Text length bounded (1–500 chars), ingredient array capped at 25 items (max 50 chars each), strict enum for dietary preferences (`'veg' | 'non-veg' | 'all'`).
- **`/api/identifyIngredients`**: Content-Length verified; Content-Type verification (`image/jpeg`, `image/png`, `image/webp`, `image/gif`), streaming byte-counter capping uploads to 5 MB (`413 Payload Too Large`).
- **`/api/getRecipeDetails`**: Dish name bounded (2–100 chars), prompt injection delimiters (`<dish_name>`, `<available_ingredients>`).
- **`/api/generateFoodImage`**: Query length limits and URI decoding guards.

### 3. Deterministic Model Hierarchy & Sub-Second Latency
Rather than executing redundant parallel model calls, Moodbite.ai implements a cost-efficient **hierarchical fallback order with a strict 12-second total deadline budget**:
1. **Primary**: Google Gemini 3.5 Flash Lite (ultra-fast, ~700–1000ms response time)
2. **Secondary**: Google Gemini 3.5 Flash (~2000ms response time)
3. **Tertiary Fallback**: Anthropic Claude 3 Haiku via OpenRouter
4. **Guaranteed Safe Fallback**: Deterministic local comfort meal structure (`isFallback: true`)

### 4. Isolated Per-Route Rate Limiting
Protects against API quota exhaustion with per-route sliding window buckets (`lib/rateLimit.js` keyed by `${ip}:${routeKey}`):
- `/api/suggestFood`: 25 requests/min.
- `/api/getRecipeDetails`: 25 requests/min.
- `/api/identifyIngredients`: 20 requests/min.
- `/api/generateFoodImage`: 40 requests/min.
- Standard HTTP `429 Too Many Requests` responses with `Retry-After` headers and clean client-side feedback.

---

## 📋 Environment Variables

Copy `.env.example` to `.env.local` and configure your API keys:

```bash
cp .env.example .env.local
```

| Variable | Required | Description | Source |
|---|---|---|---|
| `GEMINI_API_KEY` | **Required** | Primary Gemini 3.5 Flash Lite and vision API key | [Google AI Studio](https://aistudio.google.com/) |
| `HUGGING_FACE_API_TOKEN` | Optional | RoBERTa GoEmotions inference token | [Hugging Face](https://huggingface.co/) |
| `OPENROUTER_API_KEY` | Optional | Tertiary Claude Haiku fallback provider | [OpenRouter](https://openrouter.ai/) |
| `PEXELS_API_KEY` | Optional | High-resolution food photography curation | [Pexels API](https://www.pexels.com/api/) |
| `NEXT_PUBLIC_SUPABASE_URL` | Optional | Supabase Project URL for Authentication & DB | [Supabase](https://supabase.com) |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Optional | Supabase Anon Public Key for Client Auth | [Supabase](https://supabase.com) |
| `RATE_LIMIT_WINDOW_MS` | Optional | Rate limit sliding window (default: 60000ms) | Internal config |
| `RATE_LIMIT_MAX_REQUESTS` | Optional | Max requests per IP window (default: 20) | Internal config |

---

## 🚀 Getting Started

### Prerequisites
- Node.js `18.x` or later (tested on Node `v20.x` / `v24.x`)
- npm or yarn

### Installation
```bash
# 1. Clone repository
git clone https://github.com/Saksham-st12/Moodbite.ai.git
cd Moodbite.ai

# 2. Install dependencies
npm install

# 3. Configure environment
cp .env.example .env.local
# Edit .env.local and add your API keys

# 4. Start local development server
npm run dev
```
Open [http://localhost:3000](http://localhost:3000) in your web browser.

---

## 🧪 Testing Suites

### Automated Smoke Testing
```bash
npm test
```
Executes the full automated smoke test suite (**36 assertions**) against the live server verifying:
- Zod 4 request validation handling (e.g. clean HTTP 400 on empty input)
- AI Model health (asserts active `gemini-3.5-flash-lite`, not fallback)
- Recipe details and culinary comfort extraction (no medical claims)
- Dietary compliance and vegetarian exception handling (`soya keema`, `meat-free`, etc.)
- Substring collision immunity (`oil` in `boil`, `pea` in `peanut`)
- Mixed-diet mode (`'all'`) preserving non-veg dish categorization
- Delimiter and HTML sanitization defenses

### Fast Offline Unit Testing
```bash
npm run test:unit
```
Runs **36 fast offline unit tests** for `lib/dietaryCheck.js` and `lib/emotion.js` without requiring network access, API tokens, or a running server.

### Affective Classifier Evaluation
```bash
npm run eval:emotion
```
Evaluates emotion inference accuracy with Wilson score confidence intervals across standard emotional benchmarks.

---

## 📄 License
This project is open-source under the [MIT License](LICENSE).
